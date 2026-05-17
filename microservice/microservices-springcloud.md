# Microservices với Spring Cloud

> Spring Boot 3.5 · Spring Cloud 2025.0.0  
> Service Discovery · Circuit Breaker · API Gateway

---

## 1. Bức tranh tổng thể

Microservices chia ứng dụng lớn thành nhiều service nhỏ, mỗi service chạy độc lập và giao tiếp qua HTTP. Ba vấn đề cốt lõi cần giải quyết:

| Vấn đề | Công cụ | Vai trò |
|--------|---------|---------|
| Service tìm thấy nhau | Eureka | Bảng danh bạ — ai đang ở đâu |
| Service bị chết | Circuit Breaker (Resilience4j) | Fail fast — không chờ timeout |
| Client gọi vào đâu | API Gateway | Một cửa ngõ duy nhất |

---

## 2. Service Discovery — Eureka

### 2.1 Vấn đề

Khi scale một service lên nhiều instance, mỗi instance chạy trên một IP:port khác nhau. Hardcode địa chỉ gây ra:

- Scale thêm instance → phải sửa code, redeploy
- Một instance chết → vẫn gọi vào đó, nhận lỗi
- Deploy môi trường khác → địa chỉ khác hoàn toàn

### 2.2 Cách Eureka hoạt động

```
Service khởi động → đăng ký vào Eureka (tên + địa chỉ)
                         ↓
              Mỗi 30s gửi heartbeat "tôi vẫn sống"
                         ↓
Service muốn gọi → hỏi Eureka → nhận danh sách → chọn instance
                         ↓
Instance chết → Eureka xóa khỏi danh sách
```

### 2.3 Eureka Server — pom.xml

```xml
<properties>
    <spring-cloud.version>2025.0.0</spring-cloud.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
    </dependency>
</dependencies>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring-cloud.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

### 2.4 Eureka Server — Main Class

```java
@SpringBootApplication
@EnableEurekaServer  // bật Eureka Server
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

### 2.5 Eureka Server — application.yml

```yaml
server:
  port: 8761  # port mặc định của Eureka

spring:
  application:
    name: eureka-server

eureka:
  client:
    register-with-eureka: false  # không tự đăng ký vào chính nó
    fetch-registry: false        # không fetch danh sách từ chính nó
```

> **Tại sao cần `false`?**  
> Spring Cloud mặc định coi mọi app đều là Eureka client, kể cả Eureka Server.  
> Nếu không tắt, nó tự đăng ký vào chính nó và báo lỗi liên tục.

### 2.6 Eureka Client (Inventory/Order Service) — pom.xml

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <!-- eureka-client thay vì eureka-server -->
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>

<!-- Thêm dependencyManagement block giống Eureka Server -->
```

### 2.7 Eureka Client — application.yml

```yaml
server:
  port: 8081

spring:
  application:
    name: inventory-service  # tên Eureka dùng để nhận diện

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/  # địa chỉ Eureka Server
```

### 2.8 Gọi service bằng tên — @LoadBalanced

Thay vì hardcode IP, dùng tên service. `@LoadBalanced` cho phép `RestTemplate` resolve tên service qua Eureka:

```java
@Configuration
public class AppConfig {

    @Bean
    @LoadBalanced  // hỏi Eureka để resolve tên service ra IP thật
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

```java
// Gọi bằng tên service — không cần biết IP hay port
restTemplate.getForObject(
    "http://inventory-service/inventory",  // Eureka tự resolve
    String.class
);
```

### 2.9 Load Balancing

Khi có nhiều instance cùng tên, `@LoadBalanced` tự phân phối request theo Round Robin:

```bash
# Chạy thêm instance trên port khác (Windows PowerShell)
mvn spring-boot:run "-Dspring-boot.run.arguments=--server.port=8082"

# Kết quả: Eureka thấy 2 instance inventory-service
# Request luân phiên: 8081 → 8082 → 8081 → 8082...
```

Để quan sát rõ, thêm port vào response:

```java
@RestController
public class InventoryController {

    @Value("${server.port}")  // đọc port hiện tại của instance
    private String port;

    @GetMapping("/inventory")
    public String checkInventory() {
        return "Inventory Service đang chạy trên port " + port;
    }
}
```

---

## 3. Circuit Breaker — Resilience4j

### 3.1 Vấn đề

Khi Inventory Service chết, Order Service không có cơ chế dự phòng — chờ timeout 30 giây rồi trả lỗi 500. Nếu có 1000 user đang đặt hàng lúc đó, tất cả đều bị ảnh hưởng.

### 3.2 Ba trạng thái

| Trạng thái | Hành vi | Chuyển sang |
|------------|---------|-------------|
| **CLOSED** | Gọi bình thường, theo dõi tỉ lệ lỗi | → OPEN nếu lỗi vượt ngưỡng |
| **OPEN** | Không gọi, vào fallback ngay lập tức | → HALF_OPEN sau N giây |
| **HALF_OPEN** | Cho 1–2 request thử lại | → CLOSED nếu thành công, OPEN nếu vẫn lỗi |

### 3.3 pom.xml — thêm vào Order Service

```xml
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-spring-boot3</artifactId>
</dependency>

<!-- BẮT BUỘC: Resilience4j dùng AOP để wrap method, thiếu cái này annotation không hoạt động -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

### 3.4 application.yml — config Circuit Breaker

```yaml
resilience4j:
  circuitbreaker:
    instances:
      inventoryService:                               # tên instance, tự đặt
        sliding-window-size: 5                        # theo dõi 5 request gần nhất
        failure-rate-threshold: 50                    # nếu 50% lỗi → OPEN
        wait-duration-in-open-state: 10s              # sau 10s → HALF_OPEN
        permitted-number-of-calls-in-half-open-state: 2  # thử 2 request
```

### 3.5 Sử dụng @CircuitBreaker

```java
@RestController
public class OrderController {

    @Autowired
    private RestTemplate restTemplate;

    @GetMapping("/order")
    @CircuitBreaker(name = "inventoryService", fallbackMethod = "fallbackOrder")
    public String createOrder() {
        String response = restTemplate.getForObject(
            "http://inventory-service/inventory",
            String.class
        );
        return "Order thành công. Kho: " + response;
    }

    // fallback: cùng return type + thêm param Exception ở cuối
    public String fallbackOrder(Exception e) {
        return "Inventory đang bận, xử lý sau. Lỗi: " + e.getMessage();
    }
}
```

> **Quy tắc fallback method:**
> - Cùng return type với method chính
> - Thêm parameter `Exception` ở cuối
> - Không cần annotation thêm
> - Phải nằm trong cùng class

### 3.6 Kết quả thực tế

Khi Inventory tắt, gọi 10 lần liên tiếp:

| Lần | Thời gian | Trạng thái |
|-----|-----------|------------|
| 1–2 | 4000–9000ms | CLOSED — thật sự gọi vào Inventory, chờ timeout |
| 3 | ~1500ms | Đang chuyển — Circuit sắp OPEN |
| 4–10 | ~300–400ms | OPEN — vào fallback ngay, không gọi nữa |

Từ **9671ms** xuống còn **~350ms** — đây là ý nghĩa của fail fast.

### 3.7 Các annotation khác của Resilience4j

| Annotation | Dùng khi | Ý nghĩa |
|------------|----------|---------|
| `@CircuitBreaker` | Service có thể chết | Ngắt mạch, fail fast |
| `@Retry` | Lỗi tạm thời (network flicker) | Thử lại N lần trước khi báo lỗi |
| `@RateLimiter` | Giới hạn số request/giây | Chống DDoS, bảo vệ service |
| `@Bulkhead` | Giới hạn concurrent calls | Cách ly thread pool |
| `@TimeLimiter` | Timeout quá dài | Giới hạn thời gian chờ |

---

## 4. Flow tổng thể khi instance chết

```
1. Inventory instance 2 chết
          ↓
2. K8s phát hiện → tạo instance mới thay thế
          ↓
3. Instance mới khởi động → tự đăng ký vào Eureka
          ↓
4. Eureka cập nhật danh sách
          ↓
5. Trong thời gian chờ → Circuit Breaker OPEN
   → Order Service nhận fallback response ngay lập tức
          ↓
6. Instance mới sẵn sàng → Circuit HALF_OPEN → thử lại
          ↓
7. Thành công → Circuit CLOSED → hoạt động bình thường
```

---

## 5. Phân vai các công cụ

| Công cụ | Giải quyết | Không giải quyết |
|---------|------------|------------------|
| **Eureka** | Service tìm thấy nhau, load balancing | Tạo/restart instance |
| **K8s** | Tạo instance, restart khi chết, autoscale | Routing request |
| **Circuit Breaker** | Fail fast khi service chết | Sửa service bị lỗi |
| **Redis** | Cache, giảm tải DB | Discovery, routing |
| **Kafka** | Xử lý async, giảm coupling | Synchronous request |
| **API Gateway** | Một cửa ngõ, auth, rate limit | Business logic |

> **Bộ nhớ nhanh:**
> - **K8s** = đảm bảo đúng số instance **tồn tại**
> - **Eureka** = đảm bảo các service **tìm thấy nhau**
> - **Circuit Breaker** = đảm bảo hệ thống **fail fast** khi có sự cố
> - Ba thứ này phối hợp với nhau, không thay thế nhau

---

## 6. API Gateway — Spring Cloud Gateway

### 6.1 Vấn đề

Không có Gateway, client phải biết địa chỉ từng service:

```
Client → http://localhost:8081/inventory  (Inventory Service)
Client → http://localhost:8082/order      (Order Service)
Client → http://localhost:8083/payment    (Payment Service)
```

Hậu quả:
- Lộ internal architecture — client biết IP/port từng service
- Auth, logging, rate limiting phải làm lại ở từng service
- CORS, SSL phải config nhiều chỗ

### 6.2 API Gateway giải quyết

Một cửa ngõ duy nhất — client chỉ biết 1 địa chỉ:

```
Client → http://localhost:8080/inventory  →  Inventory Service
Client → http://localhost:8080/order      →  Order Service
Client → http://localhost:8080/payment    →  Payment Service
```

### 6.3 pom.xml

```xml
<properties>
    <spring-cloud.version>2025.0.0</spring-cloud.version>
</properties>

<dependencies>
    <!-- Gateway — KHÔNG thêm spring-boot-starter-web, sẽ conflict -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-gateway</artifactId>
    </dependency>

    <!-- Để Gateway hỏi Eureka lấy địa chỉ service -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
</dependencies>

<!-- Thêm dependencyManagement block giống các service khác -->
```

> **Tại sao không dùng spring-boot-starter-web?**  
> Gateway chạy trên Spring WebFlux (reactive/non-blocking). Nếu thêm spring-boot-starter-web (servlet-based) sẽ conflict và không chạy được.

### 6.4 application.yml

```yaml
server:
  port: 8080

spring:
  application:
    name: api-gateway

  cloud:
    gateway:
      discovery:
        locator:
          enabled: true                # tự động tạo route theo tên service trong Eureka
          lower-case-service-id: true  # dùng chữ thường: inventory-service thay vì INVENTORY-SERVICE

      routes:
        - id: inventory-route
          uri: lb://inventory-service  # lb = load balanced, hỏi Eureka lấy địa chỉ
          predicates:
            - Path=/inventory/**       # request có path /inventory/... → route vào đây

        - id: order-route
          uri: lb://order-service
          predicates:
            - Path=/order/**

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

> **`lb://` là gì?**  
> Báo Gateway dùng load balancer để resolve tên service qua Eureka, không gọi thẳng IP.  
> Nếu có 2 instance inventory-service, Gateway tự phân phối request giữa 2 cái.

### 6.5 Thứ tự khởi động

```
1. Eureka Server  (8761)  — phải chạy trước
2. Inventory Service (8083, 8084)
3. Order Service  (8082)
4. API Gateway    (8080)  — chạy sau cùng
```

### 6.6 Global Filter — logging mọi request

Mọi request đi qua Gateway đều bị bắt lại, không cần sửa từng service:

```java
@Component
public class LoggingFilter implements GlobalFilter {

    private static final Logger log = LoggerFactory.getLogger(LoggingFilter.class);

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String path = exchange.getRequest().getPath().toString();
        String method = exchange.getRequest().getMethod().toString();

        log.info("Request: {} {}", method, path);

        return chain.filter(exchange).then(Mono.fromRunnable(() -> {
            int status = exchange.getResponse().getStatusCode().value();
            log.info("Response: {} {} → {}", method, path, status);
        }));
    }
}
```

### 6.7 Các filter hay dùng trong application.yml

```yaml
routes:
  - id: order-route
    uri: lb://order-service
    predicates:
      - Path=/order/**
    filters:
      - StripPrefix=1               # bỏ prefix /order trước khi gửi vào service
      - AddRequestHeader=X-Gateway, true  # thêm header vào request
      - name: RequestRateLimiter
        args:
          redis-rate-limiter.replenishRate: 10   # 10 request/giây
          redis-rate-limiter.burstCapacity: 20   # tối đa 20 request cùng lúc
```

### 6.8 Bảng Predicate hay dùng

| Predicate | Ý nghĩa | Ví dụ |
|-----------|---------|-------|
| `Path` | Route theo path | `Path=/order/**` |
| `Method` | Route theo HTTP method | `Method=GET,POST` |
| `Header` | Route theo header | `Header=X-Version, v2` |
| `Query` | Route theo query param | `Query=type, premium` |
| `After` | Chỉ route sau thời điểm | `After=2025-01-01T00:00:00Z` |

---

## 7. Kiến trúc hoàn chỉnh

```
Client (mobile/web)
        ↓
  API Gateway :8080          ← một cửa ngõ, logging, rate limit, auth
        ↓
  Eureka Server :8761        ← biết ai đang ở đâu
     ↙        ↘
Inventory    Order Service
Service      :8082
:8083, :8084  ↓
(load         @CircuitBreaker
balanced)     → fallback khi Inventory chết
```

Mỗi tầng giải quyết đúng 1 vấn đề, phối hợp với nhau tạo thành hệ thống resilient.
# Microservices Architecture Design Patterns

## [Decomposition Patterns](#1-order-service-for-the-order-subdomains)
- [Decompose by Business Capability](#decompose-by-business-capability--why-how-and-what-with-a-java-example)
- [Decompose by Subdomain](#how-subdomains-communicate)
- [Decompose by Transaction](#decompose-by-transaction--why-how-and-what-with-a-java-example)
- [Strangler Pattern](#strangler-pattern--why-how-and-what-with-a-java-example)
- [Bulkhead Pattern](#bulkhead-pattern--why-how-and-what-with-a-java-example)
- [Sidecar Pattern](#sidecar-pattern--why-how-and-what-with-a-java-example)


## Integration Patterns 
- [API Gateway Pattern](#api-gateway-pattern--why-how-and-what-with-java-example)
- Aggregator Pattern
- Proxy Pattern
- Gateway Routing
- Chained Microservice Pattern
- Branch Pattern
- Client-Side UI Composition Pattern

## Database Patterns
- Database Per Service Pattern
- Shared Database per Service Pattern
- CQRS Pattern (Command Query Responsibility Segregation)
- Saga Pattern

## Observability Patterns
- Log Aggregation Pattern
- Performance Metrics Pattern
- Distributed Tracing Pattern
- Health Check Pattern

## Cross-Cutting Concerns Patterns
- External Configuration Pattern
- Service Discovery Pattern
- Circuit Breaker Pattern
- Blue-Green Deployment Pattern
- Canary Deployment Pattern


### <a id=""> **Decompose by Business Capability** – Why, How, and What (with a Java Example)</a>

**<a id="decompose-by-business-capability--why-how-and-what-with-a-java-example">Why Decompose by Business Capability?**</a>

The *Decompose by Business Capability* pattern is used to break down large monolithic applications into smaller, self-contained microservices based on core business functionalities or capabilities. This is important because it aligns microservices with the business, making systems more agile, scalable, and maintainable. Each microservice is responsible for a specific business capability and can evolve independently without impacting others.

By decomposing by business capability:
- **Scalability**: Each service can scale independently based on the business demand for that particular capability.
- **Autonomy**: Teams can work independently on different capabilities, reducing dependencies.
- **Resilience**: Failures in one service (e.g., payment service) don’t necessarily bring down others (e.g., inventory management).

---

**How to Decompose by Business Capability?**

To decompose by business capability, follow these steps:

1. **Identify core business capabilities**: Define the main functionalities of your application. For example, in an e-commerce platform, capabilities could include *User Management*, *Order Management*, *Payment*, *Inventory*, etc.
   
2. **Model each capability as a microservice**: Each capability should be implemented as an independent microservice. This allows each service to handle its own data and logic.

3. **Define communication protocols**: Microservices should communicate with each other using well-defined APIs (usually REST or messaging).

4. **Design for autonomy**: Each microservice should have its own data store (database per service pattern) and should not rely on the internal implementation details of other services.

---

**What is Decompose by Business Capability?**

This approach focuses on dividing an application into smaller, business-aligned microservices. Each service is designed to serve a specific business functionality and can be owned by a small team. For example, in a banking system, the *Account Management* service, *Loan Service*, and *Transaction Service* would each be independent microservices.

**Example Use Case:**
In an online shopping system, decomposing the monolithic application into these business capabilities:
- *User Service*: Manages user registration, profiles, and authentication.
- *Product Service*: Manages product catalog, details, and inventory.
- *Order Service*: Handles the creation, management, and tracking of orders.
- *Payment Service*: Handles payment processing and transactions.

---

**Example in Java**

Let’s say we have a monolithic e-commerce system. We’ll decompose it by creating a microservice for *Order Management*.

1. **Monolithic Controller Example**:
   In a monolithic application, the code would likely be in one place:
   ```java
   @RestController
   public class ECommerceController {

       @Autowired
       private UserService userService;
       
       @Autowired
       private OrderService orderService;
       
       @Autowired
       private PaymentService paymentService;

       @PostMapping("/placeOrder")
       public ResponseEntity<String> placeOrder(@RequestBody OrderRequest request) {
           // Check user details
           userService.validateUser(request.getUserId());
           
           // Place order
           orderService.createOrder(request);
           
           // Process payment
           paymentService.processPayment(request.getPaymentDetails());

           return new ResponseEntity<>("Order placed successfully!", HttpStatus.OK);
       }
   }
   ```

2. **Decomposed by Business Capability (Microservices)**:

   Now, we'll separate the *Order Service* from the rest. Here’s how a dedicated `OrderService` microservice might look.

   **Order Service Java Example**:
   ```java
   @RestController
   @RequestMapping("/orders")
   public class OrderController {

       @Autowired
       private OrderRepository orderRepository;

       @PostMapping
       public ResponseEntity<String> createOrder(@RequestBody OrderRequest request) {
           Order order = new Order();
           order.setUserId(request.getUserId());
           order.setProductId(request.getProductId());
           order.setQuantity(request.getQuantity());
           order.setOrderDate(new Date());

           // Save the order in the database
           orderRepository.save(order);

           return new ResponseEntity<>("Order created successfully", HttpStatus.CREATED);
       }

       @GetMapping("/{id}")
       public ResponseEntity<Order> getOrderById(@PathVariable Long id) {
           Optional<Order> order = orderRepository.findById(id);
           return order.map(value -> new ResponseEntity<>(value, HttpStatus.OK))
                       .orElseGet(() -> new ResponseEntity<>(HttpStatus.NOT_FOUND));
       }
   }
   ```

   **Payment Service Java Example**:
   ```java
   @RestController
   @RequestMapping("/payments")
   public class PaymentController {

       @Autowired
       private PaymentService paymentService;

       @PostMapping
       public ResponseEntity<String> processPayment(@RequestBody PaymentRequest request) {
           // Logic to process payment
           paymentService.processPayment(request);

           return new ResponseEntity<>("Payment processed successfully", HttpStatus.OK);
       }
   }
   ```

   In this approach:
   - The `OrderController` handles the order-specific logic.
   - The `PaymentController` is a separate service dedicated to processing payments.

   Each microservice has its own logic, and these services communicate via APIs. The system is now decomposed into manageable, scalable units that can be worked on independently by different teams.

---

**Conclusion**
Decomposing by business capability breaks monoliths into microservices based on business functions. This results in services that are easy to develop, test, and scale. Each service is aligned with the business, creating a system that’s flexible and adaptive to change.




### <a id="how-subdomains-communicate">**Decompose by Subdomain** – Why, How, and What (with a Java Example)</a>

---

### **Why Decompose by Subdomain?**

In complex systems, breaking down an application based on subdomains allows each service to be modeled around a specific area of the business. This decomposition aligns with **Domain-Driven Design (DDD)** principles, which advocate organizing services around business subdomains. A subdomain is a distinct business area with its own logic, rules, and operations. This approach creates clear boundaries between services and makes the system easier to understand and scale.

Decomposing by subdomain is useful for:
- **Better understanding of the system**: It reflects the structure of the business, making it easier for both developers and stakeholders to relate to.
- **Modularity and isolation**: Each subdomain operates independently, reducing complexity and coupling between services.
- **Adaptability**: Teams can focus on specific subdomains, allowing faster development and deployment cycles.

---

### **How to Decompose by Subdomain?**

To apply this pattern, you follow these key steps:

1. **Identify subdomains**: Analyze the business and identify its core domains and subdomains. In Domain-Driven Design, there are three types of subdomains:
   - **Core Subdomains**: The most critical part of the business (e.g., product pricing in an e-commerce platform).
   - **Supporting Subdomains**: Domains that help with the functioning of core subdomains (e.g., shipping).
   - **Generic Subdomains**: Common business areas that can be reused or outsourced (e.g., user authentication).

2. **Define bounded contexts**: Each subdomain is treated as a **bounded context**. A bounded context is the boundary within which a particular model applies. Inside this boundary, all entities, services, and data are specific to the subdomain.

3. **Map subdomains to microservices**: Each subdomain becomes a separate microservice with its own data, behavior, and API. This ensures that subdomains do not share unnecessary dependencies, and their boundaries are clear.

4. **Communication between subdomains**: Subdomains communicate via APIs, and the data is only shared through well-defined contracts, reducing tight coupling.

---

### **What is Decompose by Subdomain?**

When we decompose by subdomain, we create microservices that are aligned with different business areas, or subdomains, of an organization. Each microservice is responsible for a single subdomain and operates within a bounded context. This keeps the business logic for each domain self-contained and scalable.

For example, in an e-commerce platform, subdomains might include:
- **Order Management**: Handles order placement, status, and tracking.
- **Inventory Management**: Manages stock levels, product availability, and warehousing.
- **Payment Processing**: Handles transactions and billing.
- **Shipping**: Manages shipment tracking and delivery processes.

By organizing around subdomains, the system reflects the business's structure, making it easier to scale and evolve.

---

### **Example in Java**

Let’s say we have a monolithic e-commerce application, and we want to decompose it into subdomains. Each subdomain will have its own microservice. In this case, we’ll break it down into the following subdomains:
- **Order Management**: Manages orders and customer purchases.
- **Payment**: Handles payments for orders.
- **Inventory**: Manages product stock.

#### **1. Order Service for the Order Subdomain**

```java
@RestController
@RequestMapping("/orders")
public class OrderController {

    @Autowired
    private OrderService orderService;

    @PostMapping
    public ResponseEntity<String> createOrder(@RequestBody OrderRequest request) {
        orderService.createOrder(request);
        return new ResponseEntity<>("Order created successfully", HttpStatus.CREATED);
    }

    @GetMapping("/{id}")
    public ResponseEntity<Order> getOrderById(@PathVariable Long id) {
        Order order = orderService.getOrderById(id);
        return new ResponseEntity<>(order, HttpStatus.OK);
    }
}
```

#### **2. Payment Service for the Payment Subdomain**

```java
@RestController
@RequestMapping("/payments")
public class PaymentController {

    @Autowired
    private PaymentService paymentService;

    @PostMapping
    public ResponseEntity<String> processPayment(@RequestBody PaymentRequest request) {
        paymentService.processPayment(request);
        return new ResponseEntity<>("Payment processed successfully", HttpStatus.OK);
    }
}
```

#### **3. Inventory Service for the Inventory Subdomain**

```java
@RestController
@RequestMapping("/inventory")
public class InventoryController {

    @Autowired
    private InventoryService inventoryService;

    @PostMapping
    public ResponseEntity<String> updateStock(@RequestBody StockUpdateRequest request) {
        inventoryService.updateStock(request);
        return new ResponseEntity<>("Stock updated successfully", HttpStatus.OK);
    }

    @GetMapping("/{productId}")
    public ResponseEntity<Inventory> getInventory(@PathVariable Long productId) {
        Inventory inventory = inventoryService.getInventoryByProductId(productId);
        return new ResponseEntity<>(inventory, HttpStatus.OK);
    }
}
```

#### **How Subdomains Communicate**

When an order is placed, the **Order Service** might need to communicate with the **Payment Service** to process the payment, and with the **Inventory Service** to update the stock.

- The **Order Service** places an order and initiates a payment.
- The **Payment Service** processes the payment and confirms it.
- The **Inventory Service** updates stock after an order is placed.

Here’s how the communication might work between these subdomains:

```java
// OrderService calls the PaymentService for payment processing
public class OrderService {

    @Autowired
    private PaymentClient paymentClient;

    public void createOrder(OrderRequest request) {
        // Create the order logic...
        
        // Call Payment Service
        paymentClient.processPayment(request.getPaymentDetails());
        
        // Update order status after payment
    }
}
```

In this example, the `PaymentClient` is a REST client used by the `OrderService` to make an API call to the Payment microservice.

---

### **Conclusion**

Decomposing by subdomain ensures that your application’s services are logically organized around business areas, making it easier to scale, maintain, and extend over time. Each subdomain’s boundaries are clear, and the microservices within these boundaries are responsible for specific, well-defined business logic. This approach enables teams to focus on a single business area, making both development and system evolution more efficient.



### <a id="">**Decompose by Transaction** – Why, How, and What (with a Java Example)</a>

---

### <a id="#decompose-by-transaction--why-how-and-what-with-a-java-example">**Why Decompose by Transaction?**</a>

The *Decompose by Transaction* pattern is used to break down a monolithic application into smaller microservices by focusing on transactions or business operations that need to be isolated and handled independently. Transactions often involve multiple steps or processes, and if we decompose the system by transaction boundaries, each service can handle one or more of these processes. This decomposition provides:
- **Better isolation**: Each service is responsible for completing a specific transaction.
- **Reduced complexity**: The logic for each transaction is encapsulated within its own service, making the system easier to maintain.
- **Improved scalability**: Services can scale independently based on the transaction's demand.

By decomposing by transaction, we ensure that critical business processes (transactions) are managed efficiently, while also enhancing the modularity and fault tolerance of the system.

---

### **How to Decompose by Transaction?**

To apply this pattern, you need to follow these steps:

1. **Identify transactional boundaries**: Look at the different transactions in your application, which often involve multiple steps. For example, in an e-commerce system, a transaction could involve *placing an order*, *processing payment*, *updating inventory*, and *shipping the product*.
   
2. **Group operations into services**: Each transaction consists of multiple steps. Group these steps into distinct services based on their functionality. For example, the payment process might be handled by a `Payment Service`, while updating stock might be handled by an `Inventory Service`.

3. **Design services with transactional integrity**: Each service needs to ensure that the transaction it handles is either completed successfully or rolled back in case of failure. Microservices can ensure transactional integrity using patterns like **Sagas** (a series of compensating transactions to handle rollbacks) instead of traditional two-phase commit transactions.

4. **Use messaging or APIs for communication**: Transactions often require coordination between multiple services. This communication can be done asynchronously (via messaging queues like Kafka or RabbitMQ) or synchronously (via REST APIs).

---

### **What is Decompose by Transaction?**

When decomposing by transaction, you break an application down into microservices where each service handles a specific business transaction or a set of related operations. This allows the transaction logic to be isolated into smaller, manageable services. Each service can focus on a single responsibility (for example, payments, inventory updates, or shipment).

For example, in an online banking system:
- **Account Service**: Handles account creation, balance management, and transaction history.
- **Payment Service**: Manages money transfers, withdrawals, and deposits.
- **Fraud Detection Service**: Monitors suspicious activities and validates transactions.

Each service manages a specific type of transaction, ensuring modularity and a focus on independent business processes.

---

### **Example in Java**

Let’s take the example of an online e-commerce system. In this system, we want to decompose by transaction. The key transactions might be:
- **Order Placement**: Involves creating an order.
- **Payment Processing**: Involves charging the customer.
- **Inventory Update**: Reduces the stock of ordered items.

Each of these transactions can be handled by a separate service.

#### **1. Order Service**

The **Order Service** is responsible for creating an order and coordinating with other services (like payment and inventory).

```java
@RestController
@RequestMapping("/orders")
public class OrderController {

    @Autowired
    private OrderService orderService;

    @PostMapping
    public ResponseEntity<String> placeOrder(@RequestBody OrderRequest orderRequest) {
        // Place order logic
        orderService.createOrder(orderRequest);
        return new ResponseEntity<>("Order placed successfully", HttpStatus.CREATED);
    }
}
```

The `OrderService` class can internally call the `PaymentService` and `InventoryService`.

#### **2. Payment Service**

The **Payment Service** is responsible for processing payments.

```java
@RestController
@RequestMapping("/payments")
public class PaymentController {

    @Autowired
    private PaymentService paymentService;

    @PostMapping
    public ResponseEntity<String> processPayment(@RequestBody PaymentRequest paymentRequest) {
        paymentService.processPayment(paymentRequest);
        return new ResponseEntity<>("Payment processed successfully", HttpStatus.OK);
    }
}
```

The `PaymentService` might interact with external payment gateways or process payments directly.

#### **3. Inventory Service**

The **Inventory Service** handles stock updates.

```java
@RestController
@RequestMapping("/inventory")
public class InventoryController {

    @Autowired
    private InventoryService inventoryService;

    @PostMapping("/update")
    public ResponseEntity<String> updateInventory(@RequestBody InventoryUpdateRequest request) {
        inventoryService.updateStock(request);
        return new ResponseEntity<>("Inventory updated", HttpStatus.OK);
    }
}
```

In this approach, the services are isolated and handle individual parts of the transaction. Each transaction is handled in a different microservice:
- **Order Service** creates the order.
- **Payment Service** processes the payment.
- **Inventory Service** updates the stock.

#### **Handling Transaction Failures (Saga Pattern)**

In a distributed system, ensuring transactional consistency can be difficult. Instead of traditional database transactions, microservices use **Sagas** to handle transaction rollbacks.

In our example, if payment processing fails, we might want to cancel the order and roll back any updates made to inventory.

```java
@Service
public class OrderService {

    @Autowired
    private PaymentClient paymentClient;

    @Autowired
    private InventoryClient inventoryClient;

    public void createOrder(OrderRequest request) {
        // 1. Create Order in database (not committed yet)
        
        try {
            // 2. Process Payment
            paymentClient.processPayment(request.getPaymentDetails());

            // 3. Update Inventory
            inventoryClient.updateStock(request.getProductId(), request.getQuantity());

            // 4. Commit Order in database (if everything else succeeds)
        } catch (Exception e) {
            // Rollback actions (if payment or inventory fails)
            cancelOrder(request.getOrderId());
            throw new RuntimeException("Transaction failed: " + e.getMessage());
        }
    }
}
```

In this case:
- The **Saga** pattern ensures that if payment or inventory update fails, the system compensates for it by canceling the order, maintaining data consistency.

---

### **Conclusion**

The *Decompose by Transaction* pattern breaks down a system based on transactional boundaries, isolating complex business processes into separate services. This allows for more maintainable, scalable, and resilient systems. By using patterns like **Sagas** for managing distributed transactions, we can ensure consistency across microservices without relying on traditional database transactions. Each microservice can independently manage a part of the business process, making the entire system more flexible and responsive to changes.


### <a id="#strangler-pattern--why-how-and-what-with-a-java-example">**Strangler Pattern** – Why, How, and What (with a Java Example)</a>

---

### **Why Use the Strangler Pattern?**

The *Strangler Pattern* is used for **incrementally refactoring a monolithic application** into microservices. It's particularly useful when dealing with legacy systems that are hard to modify or rewrite in one go. The idea is to incrementally replace parts of the monolithic system with new microservices, avoiding the need for a large, risky, and expensive full-system rewrite.

The pattern gets its name from a type of vine that grows around a tree, gradually taking over the host tree's function until the tree is completely replaced. In the context of software, this means you "strangle" the legacy code by building new microservices around it, eventually replacing the entire monolith over time.

Key benefits of using the Strangler Pattern:
- **Reduced risk**: You avoid a "big bang" release, and the system remains functional throughout the transition.
- **Incremental migration**: You can move one part of the system at a time, making the migration manageable.
- **Maintainability**: It allows gradual improvement of the codebase, isolating the legacy system while building the new one.

---

### **How to Implement the Strangler Pattern?**

Here’s a step-by-step approach to applying the Strangler Pattern:

1. **Identify functionality to be replaced**: Break the monolith into smaller parts, identifying areas that can be refactored into standalone microservices. Prioritize high-impact areas or those with frequent changes.

2. **Create a new microservice**: Build a new microservice that replicates the functionality of the identified monolithic component.

3. **Route requests to the new service**: Modify the system so that requests for the replaced functionality are routed to the new microservice. This can be done using:
   - **API Gateway**: Redirect certain API calls to the new service while keeping others directed to the legacy system.
   - **Proxy**: Route traffic to the new service based on conditions (e.g., feature flags).

4. **Decommission the old code**: Once the new microservice is fully functional and all traffic has been routed to it, remove the corresponding functionality from the monolith.

5. **Repeat the process**: Continue identifying and replacing parts of the monolith until the entire system is refactored into microservices.

---

### **What is the Strangler Pattern?**

The Strangler Pattern is an architectural pattern for gradually replacing a monolithic system with microservices. It allows teams to avoid the complexity and risk associated with rewriting an entire system from scratch by migrating piece by piece.

#### Example Use Case:
Imagine you have an old monolithic e-commerce system that handles orders, payments, and product management. You want to migrate it to a microservices architecture. Instead of rewriting the entire system, you could:
- First, identify that **order processing** is a good candidate to refactor.
- Develop a new **Order Microservice**.
- Redirect all order-related API calls to the new service while the rest of the system continues using the monolith.
- Once the order service is stable, decommission the old order functionality in the monolith and move on to refactor another part (e.g., payments).

---

### **Java Example of Strangler Pattern**

Let’s walk through a practical example of using the Strangler Pattern to refactor the **Order Management** component of a legacy monolithic e-commerce system into a microservice.

#### **1. The Legacy Monolithic Order Controller**

Initially, the monolith has an order controller that handles everything related to order creation and management.

```java
@RestController
@RequestMapping("/orders")
public class LegacyOrderController {

    @PostMapping
    public ResponseEntity<String> placeOrder(@RequestBody OrderRequest request) {
        // Business logic for placing order (in monolith)
        // This could be handling database operations, calling payment, etc.
        return new ResponseEntity<>("Order placed successfully", HttpStatus.OK);
    }

    @GetMapping("/{id}")
    public ResponseEntity<Order> getOrder(@PathVariable Long id) {
        // Logic to fetch order details
        Order order = findOrderById(id);
        return new ResponseEntity<>(order, HttpStatus.OK);
    }

    // Other legacy order-related code
}
```

This legacy code handles both order creation and order retrieval within the monolithic system.

#### **2. Create the New Order Microservice**

Next, we create a new microservice to handle order creation.

```java
@RestController
@RequestMapping("/orders")
public class OrderMicroserviceController {

    @Autowired
    private OrderService orderService;

    @PostMapping
    public ResponseEntity<String> placeOrder(@RequestBody OrderRequest request) {
        // New microservice logic for placing an order
        orderService.createOrder(request);
        return new ResponseEntity<>("Order placed successfully by microservice", HttpStatus.CREATED);
    }

    @GetMapping("/{id}")
    public ResponseEntity<Order> getOrder(@PathVariable Long id) {
        // New microservice logic for retrieving an order
        Order order = orderService.getOrderById(id);
        return new ResponseEntity<>(order, HttpStatus.OK);
    }
}
```

This microservice is a replica of the order management functionality but is now decoupled from the monolithic system.

#### **3. Set up API Gateway or Proxy for Routing**

Now we need to route requests for order creation to the new microservice while other requests still go to the monolith. This can be done using an **API Gateway** or a proxy.

**Example using Spring Cloud Gateway** (an API Gateway):

```java
@Bean
public RouteLocator routeLocator(RouteLocatorBuilder builder) {
    return builder.routes()
        .route("order_service", r -> r.path("/orders/**")
            .uri("lb://order-microservice")) // Route order requests to the microservice
        .route("legacy_order", r -> r.path("/legacy-orders/**")
            .uri("lb://legacy-monolith"))  // Route other requests to the legacy system
        .build();
}
```

- All requests related to orders (`/orders/**`) are now redirected to the new microservice.
- Other requests that have not yet been migrated will continue to use the legacy system.

#### **4. Remove Old Functionality from the Monolith**

After successfully routing all traffic to the new service and verifying its stability, we can remove the corresponding logic from the monolithic `LegacyOrderController`.

```java
// The legacy order logic can now be removed or disabled:
public class LegacyOrderController {
    // Remove or comment out the order-related methods

    // Decommissioning the old order logic
}
```

---

### **Conclusion**

The *Strangler Pattern* allows for incremental migration from a monolithic architecture to a microservices-based system without a full-system rewrite. It reduces the risks and costs associated with migrating large, complex applications by gradually replacing legacy components with new microservices.

This method lets you maintain system stability while gradually transitioning to a modern, scalable architecture, ensuring continuous business operations during the migration. By routing requests through an API Gateway or proxy, you can ensure that only refactored parts of the system are directed to the new microservices, making the transition seamless.



### **Bulkhead Pattern** – Why, How, and What (with a Java Example)

---

### <a id="#bulkhead-pattern--why-how-and-what-with-a-java-example"> **Why Use the Bulkhead Pattern?**</a>

The *Bulkhead Pattern* is a resilience design pattern in microservices architecture, inspired by the compartments in a ship (bulkheads) that prevent water from flooding the entire ship in case of a breach. In software systems, this pattern is used to **isolate different parts of an application** (or services) so that a failure in one part does not cause the entire system to fail.

This isolation helps improve:
- **Fault tolerance**: If one part of the system fails, the failure is contained, preventing cascading failures.
- **Resource management**: Each part of the system is allocated its own resources (threads, memory, etc.), preventing resource starvation in other parts.
- **Availability**: It ensures that critical services remain available even if non-critical services experience issues.

For example, in a large e-commerce application, if the payment service is down, you wouldn’t want the entire website to become unavailable. By using the Bulkhead Pattern, you can isolate the payment service to prevent it from affecting other components like product browsing or order placement.

---

### **How to Implement the Bulkhead Pattern?**

To apply the Bulkhead Pattern, you divide your system into isolated parts or "bulkheads" that operate independently. Each bulkhead gets its own pool of resources, such as:
- **Thread pools**: Allocate separate thread pools to different components or services.
- **Connection pools**: Limit the number of connections each service can consume.
- **Circuit breakers**: Apply circuit breakers to stop requests to failing components.

If one service becomes overloaded or unresponsive, it won’t affect the availability or performance of other services.

#### **Steps to Implement the Bulkhead Pattern:**

1. **Identify critical services**: Determine which parts of your system are more critical and require isolation from less critical services.
   
2. **Create resource isolation**: Assign separate resources (thread pools, connection pools, etc.) for each service or set of services.

3. **Use timeouts and fallbacks**: Set timeouts for each service to avoid waiting indefinitely for a response, and provide fallback mechanisms when a service fails.

4. **Monitor resource usage**: Continuously monitor how resources are used and adjust the allocation as needed.

---

### **What is the Bulkhead Pattern?**

The Bulkhead Pattern is a structural design pattern that isolates different services or components of a system to protect them from failures in other components. Each service or component is treated as a bulkhead and has its own resources allocated, such as thread pools and memory. By doing this, you ensure that failures or resource exhaustion in one service do not affect the entire system.

#### Example Use Case:
In an e-commerce system:
- **Payment Service** and **Inventory Service** can each have their own thread pool. If the Payment Service becomes slow due to external dependencies (e.g., a third-party payment gateway), it won’t affect the Inventory Service or Order Management Service.

---

### **Java Example of Bulkhead Pattern**

Let’s look at how you can implement the Bulkhead Pattern using Java and a resilience library like **Resilience4j**. Resilience4j provides a `Bulkhead` module that limits the number of concurrent calls that a service can handle, preventing resource exhaustion.

#### **1. Maven Dependency for Resilience4j**

First, add the Resilience4j dependencies to your `pom.xml`.

```xml
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-bulkhead</artifactId>
    <version>1.7.0</version>
</dependency>
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-timelimiter</artifactId>
    <version>1.7.0</version>
</dependency>
```

#### **2. Configuring Bulkhead for a Service**

Now, let’s apply the Bulkhead Pattern to a **Payment Service** that calls an external payment provider.

```java
import io.github.resilience4j.bulkhead.Bulkhead;
import io.github.resilience4j.bulkhead.BulkheadConfig;
import io.github.resilience4j.bulkhead.BulkheadRegistry;
import io.github.resilience4j.timelimiter.TimeLimiter;
import io.github.resilience4j.timelimiter.TimeLimiterConfig;

import java.time.Duration;
import java.util.concurrent.*;

@Service
public class PaymentService {

    // Configure the Bulkhead
    private final Bulkhead bulkhead;
    private final TimeLimiter timeLimiter;

    public PaymentService() {
        BulkheadConfig bulkheadConfig = BulkheadConfig.custom()
                .maxConcurrentCalls(5) // Max 5 concurrent calls to this service
                .maxWaitDuration(Duration.ofSeconds(1)) // Wait time for available threads
                .build();

        // Initialize the Bulkhead
        bulkhead = Bulkhead.of("paymentBulkhead", bulkheadConfig);

        // Optional: Add a TimeLimiter for timeout
        TimeLimiterConfig timeLimiterConfig = TimeLimiterConfig.custom()
                .timeoutDuration(Duration.ofSeconds(3)) // Timeout for external call
                .build();
        timeLimiter = TimeLimiter.of(timeLimiterConfig);
    }

    public String processPayment(PaymentRequest request) throws Exception {
        // Use the Bulkhead to limit the number of concurrent requests
        Callable<String> bulkheadPaymentProcess = Bulkhead.decorateCallable(bulkhead, () -> {
            // Simulating a payment process (external call)
            return externalPaymentGatewayCall(request);
        });

        // Add TimeLimiter to prevent waiting indefinitely
        Future<String> future = timeLimiter.executeFuture(() -> CompletableFuture.supplyAsync(() -> {
            try {
                return bulkheadPaymentProcess.call();
            } catch (Exception e) {
                throw new RuntimeException("Payment processing failed");
            }
        }));

        // Return the result or handle timeout/failure
        return future.get();
    }

    private String externalPaymentGatewayCall(PaymentRequest request) throws InterruptedException {
        // Simulate external call latency
        Thread.sleep(2000);
        return "Payment successful for " + request.getOrderId();
    }
}
```

#### **3. Explanation of the Code:**

- **Bulkhead configuration**: Limits the number of concurrent calls to the `processPayment` method to 5. If more than 5 requests come in, other requests will have to wait for up to 1 second.
- **TimeLimiter**: If the payment processing takes more than 3 seconds (perhaps due to slow third-party systems), the call will time out, and a fallback mechanism can be invoked.
- **External call simulation**: The `externalPaymentGatewayCall` simulates a call to an external payment provider, which could have latency.

With this setup, if the Payment Service becomes overloaded, other parts of the system are not affected since the bulkhead limits concurrent calls to the service.

---

### **4. Testing the Bulkhead in Action**

Here’s how you might test the Bulkhead Pattern using this service:

```java
public class BulkheadTest {

    public static void main(String[] args) throws Exception {
        PaymentService paymentService = new PaymentService();

        // Simulating multiple payment requests
        for (int i = 0; i < 10; i++) {
            int finalI = i;
            CompletableFuture.runAsync(() -> {
                try {
                    PaymentRequest request = new PaymentRequest("Order" + finalI);
                    System.out.println(paymentService.processPayment(request));
                } catch (Exception e) {
                    System.out.println("Request " + finalI + " failed: " + e.getMessage());
                }
            });
        }
    }
}
```

Output:
- You will see that up to 5 payment requests are processed concurrently.
- Requests beyond that will either wait (if they can) or fail due to the `Bulkhead` limits.
- If the external call takes too long, the **TimeLimiter** will time out the request and potentially invoke a fallback mechanism.

---

### **Conclusion**

The *Bulkhead Pattern* ensures that different parts of a system are isolated, preventing failures in one service from affecting the entire system. This pattern helps improve fault tolerance, resource allocation, and overall system reliability. Using libraries like **Resilience4j**, you can easily implement this pattern in Java by limiting the number of concurrent requests and adding timeouts to protect your services from resource exhaustion.

By isolating services, you ensure that high-priority or critical services can continue to function even if other services are experiencing issues, thus maintaining system stability and availability.


### <a id="sidecar-pattern--why-how-and-what-with-a-java-example">**Sidecar Pattern** – Why, How, and What (with a Java Example)</a>

---

### **Why Use the Sidecar Pattern?**

The *Sidecar Pattern* is commonly used in **microservices architecture** to handle cross-cutting concerns such as **logging, monitoring, configuration, or networking**. It involves running an auxiliary service (the *sidecar*) alongside the main service to augment or enhance the functionality of the main service without modifying its code. The *sidecar* lives in the same environment as the main application, often in the same container or virtual machine.

This pattern is typically used for:
- **Separation of concerns**: It allows you to offload responsibilities like logging, security, or configuration management from the main service to the sidecar service.
- **Reusability**: The sidecar service can be reused across multiple services in the system without duplicating code.
- **Operational consistency**: Applying standard behavior (e.g., security, logging) across microservices using the same sidecar.

For example, you can use a sidecar to handle security policies, rate limiting, or observability (like metrics and logs) without modifying the core logic of the primary service.

---

### **How to Implement the Sidecar Pattern?**

In the Sidecar Pattern, the main service and its sidecar are deployed together, sharing the same environment (e.g., a pod in Kubernetes). The sidecar is not dependent on the internal code of the main service but rather runs as a separate process and augments the behavior of the main service. 

#### **Steps to Implement the Sidecar Pattern:**

1. **Identify cross-cutting concerns**: Determine which concerns (like logging, monitoring, or security) can be moved out of the main service and handled externally by the sidecar.
   
2. **Deploy the sidecar**: The sidecar service is deployed alongside the main service, usually in the same container (for Docker) or the same pod (for Kubernetes).
   
3. **Enable communication between the main service and sidecar**: This communication can occur through HTTP, gRPC, or shared volumes. The sidecar can either intercept traffic or act as a proxy.

4. **Offload tasks to the sidecar**: The main service delegates responsibilities like logging or configuration to the sidecar.

---

### **What is the Sidecar Pattern?**

The Sidecar Pattern involves running a helper service (sidecar) alongside the main service to handle concerns that are **not central to the main business logic**. The sidecar is often used for:
- **Logging and monitoring**: For instance, sending logs and metrics to an external system.
- **Configuration management**: Applying dynamic configuration changes without restarting the main service.
- **Networking**: Handling service discovery, load balancing, or network security.

#### Example Use Case:
Imagine a microservice that handles orders in an e-commerce application. You can deploy a sidecar that handles:
- **Centralized logging**: Collect logs from the main service and send them to a central log aggregation system.
- **Metrics collection**: Record metrics like response times and error rates.
- **Service discovery**: Make the service discoverable by other services in the architecture.

---

### **Java Example of the Sidecar Pattern**

Let’s see how to implement a sidecar for logging in a Java-based microservice using **Docker**. In this example, we will:
1. Create a **Main Service** that handles user requests.
2. Create a **Logging Sidecar** that collects and forwards logs from the main service to a centralized logging system.

#### **1. Main Service (Java Spring Boot)**

Let’s start by building a basic Spring Boot application that acts as the main service.

```java
@RestController
@RequestMapping("/api")
public class MainServiceController {

    private static final Logger logger = LoggerFactory.getLogger(MainServiceController.class);

    @GetMapping("/orders")
    public ResponseEntity<String> getOrderDetails() {
        logger.info("Fetching order details");
        // Simulate fetching order details
        return new ResponseEntity<>("Order details fetched", HttpStatus.OK);
    }

    @PostMapping("/orders")
    public ResponseEntity<String> createOrder(@RequestBody String order) {
        logger.info("Creating new order: " + order);
        // Simulate order creation
        return new ResponseEntity<>("Order created", HttpStatus.CREATED);
    }
}
```

This service logs the operations for getting and creating orders. However, rather than handling the logging itself, we will delegate the task to a sidecar service.

#### **2. Sidecar Logging Service (Fluentd)**

To implement the Sidecar Pattern, we will use **Fluentd** as a logging sidecar. Fluentd will collect logs from the main service and forward them to a centralized logging system like **Elasticsearch** or **AWS CloudWatch**.

**Fluentd Configuration (fluent.conf):**

```plaintext
<source>
  @type tail
  path /var/log/app/logs.log
  pos_file /var/log/app/logs.log.pos
  tag main-service.log
  format none
</source>

<match main-service.log>
  @type stdout
  # In a real scenario, this could forward to Elasticsearch or CloudWatch
</match>
```

This configuration:
- **Tails** the logs from the main service's log file.
- Outputs the logs to **stdout** (in a real use case, this could send logs to a central logging system).

#### **3. Docker Setup**

Now, we will set up Docker to run both the main service and the sidecar together.

**Dockerfile for Main Service:**

```dockerfile
# Use a base image for Java Spring Boot
FROM openjdk:11-jre-slim
WORKDIR /app
COPY target/main-service.jar /app/main-service.jar
ENTRYPOINT ["java", "-jar", "/app/main-service.jar"]
```

**Dockerfile for Fluentd (Sidecar):**

```dockerfile
FROM fluent/fluentd:latest
COPY fluent.conf /fluentd/etc/fluent.conf
ENTRYPOINT ["fluentd", "-c", "/fluentd/etc/fluent.conf"]
```

#### **4. Docker Compose Configuration**

We’ll use **Docker Compose** to deploy both the main service and the sidecar together.

```yaml
version: '3'
services:
  main-service:
    build:
      context: .
      dockerfile: Dockerfile
    volumes:
      - ./logs:/var/log/app  # Share log directory with Fluentd
    ports:
      - "8080:8080"

  logging-sidecar:
    build:
      context: ./fluentd
      dockerfile: Dockerfile
    volumes:
      - ./logs:/var/log/app  # Share log directory with main service
    depends_on:
      - main-service
```

This `docker-compose.yml` file:
- Runs the **main service** and the **sidecar** together.
- Mounts a shared volume (`./logs`) to allow the main service to write logs that the sidecar can read and forward.

#### **5. Running the Setup**

1. **Build and run** the services using Docker Compose:

   ```bash
   docker-compose up
   ```

2. The main service logs its activities (like creating and fetching orders) to `/var/log/app/logs.log`, and the sidecar (Fluentd) reads these logs and forwards them.

---

### **Conclusion**

The *Sidecar Pattern* is a powerful architectural pattern that allows you to extend the functionality of your services by attaching auxiliary components without modifying the core logic. This pattern enables a clean separation of concerns, where cross-cutting functionalities like logging, monitoring, and security are handled by sidecars.

By using Docker or Kubernetes, you can deploy a sidecar alongside your main services, ensuring consistency, reuse, and flexibility. This pattern is widely used in microservices environments for tasks like logging, service discovery, monitoring, and configuration management, helping teams build scalable, resilient, and maintainable systems.


### <a id="api-gateway-pattern--why-how-and-what-with-java-example"> **API Gateway Pattern** – Why, How, and What (with Java Example)</a>

---

### **Why Use the API Gateway Pattern?**

The *API Gateway Pattern* is used in **microservices architecture** to manage the communication between client applications and multiple microservices. It acts as a **single entry point** for all client requests, abstracting the complexity of multiple services and providing functionalities like:
- **Routing**: Forwarding client requests to appropriate microservices.
- **Security**: Centralizing authentication, authorization, and rate-limiting.
- **Aggregation**: Combining responses from different services into a single response.
- **Cross-cutting concerns**: Handling logging, monitoring, and caching centrally.

It helps prevent the following issues:
- **Client coupling**: Without an API Gateway, clients would need to directly interact with each microservice, which leads to tight coupling and makes client-side code more complex.
- **Multiple round trips**: Clients might need to make multiple calls to gather data, leading to inefficient communication and performance bottlenecks.

### **How Does the API Gateway Pattern Work?**

The API Gateway serves as a **facade** for the microservices. Clients only need to interact with the gateway, which:
1. **Authenticates and authorizes** requests.
2. **Routes requests** to the appropriate service.
3. **Aggregates responses** from different services (if necessary).
4. **Translates protocols or formats** if different microservices use different communication mechanisms.

The API Gateway may also:
- Perform **caching** to reduce load on backend services.
- Enforce **rate limiting** to prevent abuse.
- Handle **failover** and **load balancing** between microservices.

### **What is the API Gateway Pattern?**

The API Gateway is a **proxy server** that sits between the clients and microservices. It ensures that all requests pass through it, simplifying the client interaction with the system and centralizing concerns that apply to multiple microservices. 

Key responsibilities include:
- **Request routing**: Directing traffic to the correct service based on URL paths, headers, or query parameters.
- **Response transformation**: Modifying the response from one or more services to present a unified result to the client.
- **Security enforcement**: Verifying and enforcing security policies like OAuth2, JWT tokens, or API keys.
- **Service discovery**: Dynamically routing to services based on their current location (e.g., in a cloud environment).

---

### **Example Use Case for API Gateway**

In a typical **e-commerce system**, a client might need to fetch information from multiple microservices such as:
1. **User Service** – For user profile data.
2. **Order Service** – To get the user's order history.
3. **Product Service** – To fetch product details.

The API Gateway consolidates these requests into a single call from the client:

```yaml
Client --> [API Gateway] --> [User Service]
                         --> [Order Service]
                         --> [Product Service]
```

Instead of making three separate calls from the client, the API Gateway routes them to the appropriate services and returns a single, aggregated response.

---

### **Java Example of API Gateway Using Spring Cloud Gateway**

In this example, we will build a **Spring Boot API Gateway** using **Spring Cloud Gateway**. It will route requests to different services based on the URL path.

#### **1. Set Up Spring Cloud Gateway**

In the `pom.xml` file, add the dependencies for Spring Cloud Gateway and Netflix Eureka (for service discovery):

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

#### **2. API Gateway Configuration**

The API Gateway routes traffic based on URL paths. Below is the configuration that directs `/user/**` to the **User Service** and `/order/**` to the **Order Service**.

```java
@SpringBootApplication
@EnableEurekaClient
public class ApiGatewayApplication {

    public static void main(String[] args) {
        SpringApplication.run(ApiGatewayApplication.class, args);
    }

    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
                .route("user_service", r -> r.path("/user/**")
                        .uri("lb://USER-SERVICE"))  // 'lb' stands for load balancer, it will route to Eureka-discovered User Service
                .route("order_service", r -> r.path("/order/**")
                        .uri("lb://ORDER-SERVICE"))  // Eureka-discovered Order Service
                .build();
    }
}
```

- **`/user/**`**: Routes all requests with this path to the **User Service**.
- **`/order/**`**: Routes all requests with this path to the **Order Service**.
- **`lb://`**: This indicates that service discovery is enabled and the services are registered with **Eureka**.

#### **3. Application Properties (application.yml)**

Here’s an example of the application configuration file:

```yaml
server:
  port: 8080

spring:
  application:
    name: api-gateway

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/

cloud:
  gateway:
    routes:
      - id: user_service
        uri: lb://USER-SERVICE
        predicates:
          - Path=/user/**
      - id: order_service
        uri: lb://ORDER-SERVICE
        predicates:
          - Path=/order/**
```

- The `api-gateway` will run on port 8080.
- The services (e.g., `USER-SERVICE` and `ORDER-SERVICE`) will be discovered using **Eureka**.

#### **4. Eureka Service Registry**

Make sure that both the **User Service** and **Order Service** are registered with **Eureka**. The API Gateway will use Eureka to discover the services and route traffic accordingly.

**User Service (Eureka Client Configuration):**
```yaml
spring:
  application:
    name: user-service

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

**Order Service (Eureka Client Configuration):**
```yaml
spring:
  application:
    name: order-service

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

#### **5. Running the Application**

1. Start the **Eureka Server**.
2. Start the **User Service** and **Order Service** (which are Eureka clients).
3. Start the **API Gateway**.

Once all the services are up, the API Gateway will automatically discover the services via Eureka and route the requests based on the defined paths.

- A request to `http://localhost:8080/user/123` will be routed to the **User Service**.
- A request to `http://localhost:8080/order/456` will be routed to the **Order Service**.

---

### **Advantages of API Gateway**

1. **Centralized control**: The API Gateway centralizes common functionalities like logging, authentication, and rate-limiting.
2. **Simplified client**: Clients don’t need to worry about interacting with multiple services. They just call the gateway.
3. **Service abstraction**: Clients are unaware of how services are organized or what changes behind the scenes.
4. **Aggregated responses**: The API Gateway can aggregate responses from multiple services into a single response, reducing the need for multiple round trips.

---

### **Conclusion**

The *API Gateway Pattern* is a key architectural pattern in microservices that simplifies client interaction, enhances security, and handles cross-cutting concerns in a centralized manner. With tools like **Spring Cloud Gateway**, building a scalable and efficient API Gateway in Java becomes straightforward, allowing developers to focus on the core business logic of their microservices.
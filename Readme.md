# Microservices Architecture Design Patterns

## [Decomposition Patterns](#1-order-service-for-the-order-subdomains)
- [Decompose by Business Capability](#decompose-by-business-capability--why-how-and-what-with-a-java-example)
- [Decompose by Subdomain](#how-subdomains-communicate)
- [Decompose by Transaction](#decompose-by-transaction--why-how-and-what-with-a-java-example)
- Strangler Pattern
- Bulkhead Pattern
- Sidecar Pattern
- Integration Patterns

## API Gateway Pattern
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
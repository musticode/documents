# Microservice Anti Patterns

1. Distributed Monolith
2. Too Many Services (Over-Decomposition)
3. Shared Database Anti-Pattern
4. Microservices Spaghetti
5. Lack of Observability
6. Hardcoded Configuration
7. Poor API Design
8. Inadequate Service Ownership
9. Over-Reliance on Synchronous Communication
10. Ignoring Backward Compatibility


### Distributed monolith 
- occurs when microservice are deployed independently but remain tightly coupled in their operation. This happens when a service boundaries are poorly defined, and services rely excessively on synchronous communication or shared state. Although services are distributed, the arch behaves like a monolithic application
**Causes**
- Over-reliance on synchronous REST API for communications
- Lack of clear seperation of concerns during service design
- Using a shared database schema across services
**Symtoms**
- Tight coupling : a change in one service forces changes in dependent services
- Corrdinated deployments : multiple services need simultaneous redeployment for feature updates
- Single points of failure : if a service fails, it cascades across the system
**Examples**
- An e-commerce application where the order, payment and inventory services depend on each other for every transaction
- A system where all services read from and write to the same database tables

## Links

- https://blog.stackademic.com/the-dark-side-of-microservices-top-10-anti-patterns-exposed-47aa867ef1b2



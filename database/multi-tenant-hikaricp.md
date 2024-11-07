# [Spring Boot] Multi-tenant with HikariCP

- Diff :
  - Single tenant : seperate application, seperate database
  - Multi tenant : same application, seperate database
- What is Multi-tenancy? Multi-tenancy is an architecture where a single instance of software application serves service to multiple clients. Unlike the traditional architecture of separate applications and databases, Multi-tenancy allows the same application to connect to separate databases. The architecutre provides an economic benefits, scalability, and increased productivity by sharing the costs of software development and maintenance. However, it also presents challenges such as data isolation, performance issues, and maintenance complexity. Ensuring stable connections between applications and databases is crucial for service operation.


### How to manage connections?
- Spring Boot saves connection and related information as DataSource object. By registering the DataSource object as a bean, it passes the connection information as parameters. Then, Spring Boot obtains database connection from DataSource.


[](https://medium.com/@myggona/spring-boot-multi-tenant-with-hikaricp-c1c5072cbe0e)
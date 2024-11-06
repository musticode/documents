## Configurations

Spring Boot DATABASE-DOCKER

Spring boot docker-compose:



POSTGRESQL
```yml
version: '3.5'


services:
  postgres:
    container_name: postgres_demo
    image: postgres:latest
    environment:
      POSTGRES_USER: super_admin
      POSTGRES_PASSWORD: SomeSecretPassword
      PGDATA: /data/postgres
    volumes:
      - postgres-db:/data/postgres
    ports:
      - "5432:5432"


volumes:
  postgres-db:
    driver: local
```

MONGODB
```yml
version: '3'
services:
  myapplication:
    image: mongodb/mongodb-community-server:6.0-ubi8
    environment:
      - CONN_STR=mongodb://user:pass@mongodb
    command: '/bin/bash -c "sleep 5; mongosh $$CONN_STR --eval \"show dbs;\""'
    depends_on:
      - mongodb
  mongodb:
    image: mongodb/mongodb-community-server:6.0-ubi8
    environment:
      - MONGO_INITDB_ROOT_USERNAME=user
      - MONGO_INITDB_ROOT_PASSWORD=pass
    volumes:
      - type: bind
        source: ./data
        target: /data/db
```




Spring boot application properties:

MYSQL
```properties
spring.jpa.hibernate.ddl-auto=update
spring.datasource.url=jdbc:mysql://${MYSQL_HOST:localhost}:3306/spring_jwt_db
spring.datasource.username=root
spring.datasource.password=123456789
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
#spring.jpa.show-sql: true 
```
 
POSTGRESQL
```properties
spring.jpa.properties.hibernate.dialect = org.hibernate.dialect.PostgreSQLDialect
spring.jpa.hibernate.ddl-auto=update
spring.jpa.hibernate.show-sql=true
spring.datasource.url=jdbc:postgresql://localhost:5432/postgres
spring.datasource.username=super_admin
spring.datasource.password=SomeSecretPassword


spring.datasource.initialization-mode=always
spring.datasource.initialize=true
spring.datasource.schema=classpath:/schema.sql
spring.datasource.continue-on-error=true
```

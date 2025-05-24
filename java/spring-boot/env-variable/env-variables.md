# Override application properties


```bash
docker run -p 8080:8080 -e APP_PROPERTIES_NAME=overridden-name-cevheri-docker -e APP_ROLES_0=overridden-role-0-cevheri-docker -e APP_PATHS_0_ROLES_4=overridden-path-0-role-0-cevheri-docker soe:latest
```

https://cevheri.medium.com/java-spring-boot-overriding-environment-variables-guide-64283bee6132


# Env variables


Spring Boot configuration
Use spring.config.import in your application.properties file to import the .env file:

```yml
# support reading from .env file
spring.config.import=file:../.env[.properties],file:.env[.properties]

spring.r2dbc.name=${POSTGRES_DB}
spring.r2dbc.username=${POSTGRES_USER}
spring.r2dbc.password=${POSTGRES_PASSWORD}
spring.r2dbc.host=${POSTGRES_HOST}
spring.r2dbc.port=${POSTGRES_PORT}
spring.r2dbc.url=r2dbc:postgresql://${POSTGRES_HOST}:${POSTGRES_PORT}/${POSTGRES_DB}
```

env variable usage : https://www.surly.dev/articles/how-to-use-env-files-with-spring-boot
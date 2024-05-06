# Mongo and Mongo Express


```yml
services:

  mongo:
    image: 'mongo:7.0.5'
    ports:
      - 27017:27017
    volumes:
      - my-data:/var/lib/mongodb/data

  mongo-express:
    image: 'mongo-express:1.0.2'
    ports:
      - 8082:8081
    environment:
      ME_CONFIG_BASICAUTH_USERNAME: username
      ME_CONFIG_BASICAUTH_PASSWORD: password

```
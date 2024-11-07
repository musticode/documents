# Circuit breaker

When communication is in progress between microservices, sometimes there can be failures between them. When a failure happens, it may causes domino effects up to client. Circuit-breaker can be thought like a proxy. Communication is established between services via proxy. If number of fail response is exceeded a threshold value, then the related circuit-breaker prevents communications for a period of time. A fallback which provided by developer is executed until service up. After a while, communication is established gradually.

Hystrix
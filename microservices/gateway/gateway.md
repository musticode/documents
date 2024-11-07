# Gateway

Gateways can be thought like abstraction layer when accessing microservices. Details of all microservices must not be known by their users. A gateway works as a single entry point which is used to route requests. It also makes refactoring more easy. Because the users only communicate with gateway, so they would not be affected on any change on services as long as the gateway does its duty as promised.

Another benefit of using gateway is that it reduces security concerns which are caused accessing directly related microservice. For example, a service may need authentications. Gateways can manage authentication of the request. If request includes tokens, then the token is validated. Otherwise request is redirected to authentication service.

- Gateway: Zool

> Identity and access management systems(IAMs): Okta, Keycloak, Shiro
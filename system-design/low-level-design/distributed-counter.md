# Distributed Counter Low Level Design

## User Layer | Component: User (External Clients)
- End users interact with the system via API calls to perform opertaions like increment, decrement and get counter values.
- Users initiate operations that flow through the API and trigger updates to database, cache and message broker for real-time updates

## API Layer | Component API Layer
- This layer processes user requests and forwards them to the appropriate service(Primary node or secondary node)
- Validates and coordinates actions between the primary node service, secondary node service, distributed database, cache and message broker
- API handles the business logic like icrementing a counter, updating the cache and sending messages to notify other system or services

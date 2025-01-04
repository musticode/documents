# Event Listener in SpringBoot

ApplicationEvent class should be extended to our model class

```java
import org.springframework.context.ApplicationEvent;

public class AddEmployeeEvent extends ApplicationEvent {

  public AddEmployeeEvent(Employee employee) {
    super(employee);
  }

  @Override
  public String toString() {
    return "ApplicationEvent: New Employee Saved :: " + this.getSource();
  }
}
```

- With later versions of Spring, extending the ApplicationEvent is not mandatory.

model class : 

```java
public class AddEmployeeEvent {

  private Employee employee;

  public AddEmployeeEvent(Employee employee) {
    this.employee = employee;
  }

  public Employee getEmployee() {
    return employee;
  }

  @Override
  public String toString() {
    return "ApplicationEvent: New Employee Saved :: " + this.employee;
  }
}
```

## Publishing an Application Event

The ApplicationEventPublisher interface encapsulates the whole event publication functionality. To access the ApplicationEventPublisher in a class, we may, optionally, implement the ApplicationEventPublisherAware interface in that class; else the Spring injects it automatically in the runtime.

```java
import org.springframework.context.ApplicationEventPublisher;
import org.springframework.context.ApplicationEventPublisherAware;
import org.springframework.stereotype.Service;

@Service
public class EmployeeService implements ApplicationEventPublisherAware {

  EmployeeRepository repository;
  ApplicationEventPublisher applicationEventPublisher;

  @Override
  public void setApplicationEventPublisher(ApplicationEventPublisher applicationEventPublisher) {
    this.applicationEventPublisher = applicationEventPublisher;
  }

  public EmployeeService(EmployeeRepository repository) {
    this.repository = repository;
  }

  public Employee create(Employee employee) throws ApplicationException {
    Employee newEmployee = repository.save(employee);
    if (newEmployee != null) {
      applicationEventPublisher.publishEvent(new AddEmployeeEvent(newEmployee));   //Notify the listeners
      return newEmployee;
    }
    throw new ApplicationException("Employee could not be saved");
  }
}
```
## Receiving Application Events


To listen to certain events, a bean must implement the ApplicationListener interface and handle the events in the onApplicationEvent() method. Actually, Spring will notify a listener of all events, so you must filter the events by yourself.

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.context.ApplicationListener;
import org.springframework.stereotype.Component;

@Slf4j
@Component
public class DeleteEmployeeEventListener implements ApplicationListener<DeleteEmployeeEvent> {

  @Override
  public void onApplicationEvent(DeleteEmployeeEvent event) {
    log.info(event.toString());
  }
}
```

#### Using @EventListener annotation : 

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.context.event.EventListener;
import org.springframework.stereotype.Component;

@Slf4j
@Component
public class EmployeeEventsListener {

  @EventListener
  void handleAddEmployeeEvent(AddEmployeeEvent event) {
    log.info(event.toString());
  }

  @EventListener
  void handleDeleteEmployeeEvent(DeleteEmployeeEvent event) {
    log.info(event.toString());
  }
}
```

- Note that event type can be defined in the annotation itself. Also, we can register multiple events to a single method

```java
@EventListener({AddEmployeeEvent.class, DeleteEmployeeEvent.class})
void handleEmployeeEvents(EmployeeEvent event) {
log.info(event.toString());
}
```

In most cases, the event handler method should be void. And if the methpd returns a value, result of the method invocation is sent as a new event. Also, if the return type is either an array or a collection, each element is sent as a new individual event.

```java
@EventListener({AddEmployeeEvent.class, DeleteEmployeeEvent.class})
RemoteAuditEvent handleEmployeeEvents(EmployeeEvent event) {

  log.info(event.toString());
  return new RemoteAuditEvent(event.toString());
}
```
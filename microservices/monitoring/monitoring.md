# Monitoring

Monitoring is important to detect issues that affects the process.

> Kibana, Grafana, Splunk are the most useful tools can be used for monitoring.

Logs should be aggregated using tools like LogStash. Also metric service tools are useful to detect performance issues in the system. DropWizard, Spring Actuator, Prometheus can be used. Outputs of the monitoring tools are used to optimize the system. For example, rate limiting can be implemented based on the usage metrics of the APIs. Monitoring also can be used to billing the customers. Monitoring tools may not be sufficient to detect issues on the system. Tracing tools are also helpful in that manner. Tracing tools can be used to watch performance metrics of requests along the way.

> Zipkin is the one of the tracing tool
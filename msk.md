
# In MSK broker have two broker which one should I use

In a setup with multiple brokers in an Amazon MSK (Managed Streaming for Apache Kafka) cluster, it's a good practice to use multiple brokers for high availability and fault tolerance. In such a scenario, you typically connect to any of the available brokers in the cluster rather than specifying a particular one.

When configuring your Spring Boot application to connect to the MSK cluster, you can provide a comma-separated list of broker addresses. This way, if one broker goes down, your application can still connect to the other available brokers.

For example, in your Spring Boot `application.properties` or `application.yml` file, you might configure the `spring.kafka.bootstrap-servers` property like this:

```properties
spring.kafka.bootstrap-servers=broker1:9092,broker2:9092
```

Replace `broker1` and `broker2` with the actual DNS names or IP addresses of your MSK brokers.

By providing a list of brokers, your Spring Boot application can automatically discover and use any available broker in the cluster, providing the necessary fault tolerance and load balancing.

It's important to note that Amazon MSK handles the distribution of partitions across brokers, so your application doesn't need to be concerned about the specific broker handling a particular partition. The Kafka clients will handle this transparently.

# 

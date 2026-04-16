---
title: "Deep Dive into Kafka Offset Commit with Spring Boot"
date: 2026-04-16
draft: false
tags: ["Kafka", "Spring Boot", "Microservices", "Java"]
categories: ["Kafka"]
summary: "A comprehensive guide to understanding Kafka offset commit strategies when using Spring Boot. Learn about auto-commit, manual commit, and best practices."
cover:
  image: ""
  alt: "Kafka Offset Commit"
  relative: false
---

## Introduction

Apache Kafka uses **offsets** to track the position of a consumer within a partition. Understanding how offset commits work is crucial for building reliable event-driven applications with Spring Boot.

In this post, we'll explore:
- What are Kafka offsets?
- Auto-commit vs manual commit
- Spring Boot configuration for offset management
- Best practices and pitfalls

## What Are Kafka Offsets?

Every message in a Kafka partition has a unique sequential ID called an **offset**. When a consumer reads messages, it needs to track which messages it has already processed. This is done through **offset commits**.

```java
@KafkaListener(topics = "orders", groupId = "order-service")
public void consume(ConsumerRecord<String, Order> record) {
    log.info("Received order: {} at offset: {}", 
        record.value(), record.offset());
    orderService.process(record.value());
}
```

## Auto-Commit

By default, Spring Kafka uses auto-commit. The consumer periodically commits offsets in the background.

```yaml
spring:
  kafka:
    consumer:
      enable-auto-commit: true
      auto-commit-interval: 5000
      auto-offset-reset: earliest
      group-id: order-service
    bootstrap-servers: localhost:9092
```

### The Problem with Auto-Commit

Auto-commit can lead to **message loss** or **duplicate processing**:
- If the consumer crashes after processing but before the next auto-commit, messages will be reprocessed.
- If the auto-commit happens before processing completes, messages may be lost.

## Manual Commit with Spring Boot

For production systems, **manual commit** gives you full control:

```java
@Configuration
public class KafkaConfig {

    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, String> 
            kafkaListenerContainerFactory(ConsumerFactory<String, String> cf) {
        
        var factory = new ConcurrentKafkaListenerContainerFactory<String, String>();
        factory.setConsumerFactory(cf);
        factory.getContainerProperties()
            .setAckMode(ContainerProperties.AckMode.MANUAL_IMMEDIATE);
        return factory;
    }
}
```

```java
@KafkaListener(topics = "orders", groupId = "order-service")
public void consume(ConsumerRecord<String, Order> record, 
                    Acknowledgment acknowledgment) {
    try {
        orderService.process(record.value());
        acknowledgment.acknowledge(); // commit only after successful processing
    } catch (Exception e) {
        log.error("Failed to process order: {}", record.value(), e);
        // Don't acknowledge — message will be redelivered
    }
}
```

## Commit Strategies Comparison

| Strategy | Pros | Cons |
|----------|------|------|
| Auto-commit | Simple, no code needed | Risk of data loss/duplicates |
| Manual (MANUAL) | Batch control | Slightly more complex |
| Manual (MANUAL_IMMEDIATE) | Per-message control | Higher overhead |
| Manual (RECORD) | Auto per-record | Less flexible |

## Best Practices

1. **Use manual commit** for critical data pipelines
2. **Implement idempotency** — even with manual commits, duplicates can happen
3. **Monitor consumer lag** — use tools like Kafka Lag Exporter
4. **Set appropriate timeouts** — `max.poll.interval.ms` and `session.timeout.ms`
5. **Use Dead Letter Topics** for messages that fail repeatedly

```java
@Bean
public DefaultErrorHandler errorHandler(KafkaTemplate<String, Object> template) {
    var recoverer = new DeadLetterPublishingRecoverer(template);
    return new DefaultErrorHandler(recoverer, new FixedBackOff(1000L, 3));
}
```

## Conclusion

Choosing the right offset commit strategy depends on your application's requirements for **data consistency** and **throughput**. For most production systems, manual commit with proper error handling and idempotency is the recommended approach.

---

*If you found this helpful, share it with your team! Follow me for more deep dives into Kafka and Spring Boot.*

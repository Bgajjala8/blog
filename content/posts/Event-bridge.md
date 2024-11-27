---
title: "How I Built a System to Transfer Messages Across Cloud Providers"
date: 2024-11-27T04:04:13+00:00
tags: [Cloud, Messaging, Event-bridge]
draft: false
disqus: false
---
## The Challenge: Cross-Cloud Messaging

During a recent cloud transformation initiative, I faced a significant challenge: enabling reliable and efficient communication between different cloud providers' messaging systems. These systems—each with their own protocols and architecture—often lack native compatibility, making direct integration difficult or impossible. The goal was to create a solution that could bridge this gap, ensuring business continuity while enabling cross-cloud workflows.

## The Solution: EventBridge with Apache Camel

To address this challenge, I developed an in-house system called **EventBridge**, powered by **Apache Camel**. Hosted on a Kubernetes cluster, EventBridge allows customers to easily configure **route bridges** between messaging services, while Apache Camel handles the **underlying complexities of integration**.

This approach allowed us to connect on-premises systems, such as **Kafka**, with cloud services like **Azure Service Bus**, **AWS SQS**, and **GCP Pub/Sub**. Apache Camel's rich set of connectors ensured that the underlying protocols and configurations for each service were abstracted away, enabling seamless communication.

### Configuration-Driven Routing for Customers

With EventBridge, customers could define and manage route bridges through configuration. Apache Camel handled all the underlying components, such as connection details, protocols, and data transformation. Here's an example of how a configuring looks like:

### Example: Congfiguration

```java
package config;

import io.smallrye.config.ConfigMapping;
import io.smallrye.config.WithDefault;

import java.util.List;
import java.util.Optional;

@ConfigMapping(prefix = "config")
public interface RouteConfig {

    public List<CamelRouteConfiguration> CAMEL_ROUTE_CONFIGURATION_LIST();

    interface CamelRouteConfiguration {
        String routeName();

        @WithDefault("false")
        Boolean transformMessage();

        @WithDefault("true")
        Boolean enable ();

        Metadata source();

        Metadata target();
    }

    interface Metadata {
        Optional<String> topic ();
        CloudServiceQueue service();
        String topicUniqueIdentifier();
    }

    enum CloudServiceQueue {
        AZURE_SERVICEBUS, ONPREM;
    }
}
```

#### Example: Camel Template for Azure Service Bus

```java
package template;

import org.apache.camel.builder.RouteBuilder;

/**
 * Azure Service Bus
 * Protocols Supported: AMQP
 */
public class AzureServiceBusTemplate extends RouteBuilder implements RouteTemplate {

    private String PRODUCER_ROUTE_TEMPLATE_NAME = "AZURE_SERVICE_BUS_PRODUCER";

    private String PRODUCER_ROUTE_ENTRY_POINT = "direct:AZURE_SERVICE_BUS_PRODUCER";

    private String CONSUMER_ROUTE_TEMPLATE_NAME = "AZURE_SERVICE_BUS_CONSUMER";

    @Override
    public void configure() throws Exception {
        // Empty on purpose
    }

    @Override
    public String getProducerTemplateName() {
        return PRODUCER_ROUTE_TEMPLATE_NAME;
    }

    @Override
    public String getConsumerTemplateName() {
        return CONSUMER_ROUTE_TEMPLATE_NAME;
    }

    @Override
    public String producerRouteEntryPointName() {
        return PRODUCER_ROUTE_ENTRY_POINT;
    }

    @Override
    public void buildProducerRoute() {
        routeTemplate(this.PRODUCER_ROUTE_TEMPLATE_NAME)
                .templateParameter("topic", "TOPIC-NOT-DEFINED")

                // Definition
                .from(this.PRODUCER_ROUTE_ENTRY_POINT)
                .to("azure-servicebus:{{topic}}")
                .log("Topic: {{topic}}")
                .log("${body}");
    }

    @Override
    public void buildConsumerRoute() {
        routeTemplate(this.CONSUMER_ROUTE_TEMPLATE_NAME)
                .templateParameter("poll-topic", "POLL-TOPIC-NOT-DEFINED")
                .templateParameter("destination", "DESTINATION-NOT-DEFINED")

                // Definition
                .from("azure-servicebus:{{poll-topic}}")
                .to("{{destination}}")
                .log("Destination: {{destination}}")
                .log("${body}");

    }
}
```

#### Example: Camel Template for On premp Kafka Cluster

```java
package template;

import org.apache.camel.builder.RouteBuilder;

/**
 * On prem Kafka Cluster
 * Supported Protocol: Kafka
 */
public class OnPremKafkaTemplate extends RouteBuilder implements RouteTemplate {
    private String PRODUCER_ROUTE_TEMPLATE_NAME = "KAFKA_PRODUCER";

    private String PRODUCER_ROUTE_ENTRY_POINT = "direct:KAFKA_PRODUCER";

    private String CONSUMER_ROUTE_TEMPLATE_NAME = "KAFKA_CONSUMER";


    @Override
    public void configure() throws Exception {
        // Empty on purpose
    }

    @Override
    public String getProducerTemplateName() {
        return PRODUCER_ROUTE_TEMPLATE_NAME;
    }

    @Override
    public String getConsumerTemplateName() {
        return CONSUMER_ROUTE_TEMPLATE_NAME;
    }

    @Override
    public String producerRouteEntryPointName(){
        return PRODUCER_ROUTE_ENTRY_POINT;
    }

    @Override
    public void buildProducerRoute() {
        routeTemplate(this.PRODUCER_ROUTE_TEMPLATE_NAME)
                .templateParameter("topic", "TOPIC-NOT-DEFINED")

                // Definition
                .from(this.PRODUCER_ROUTE_ENTRY_POINT)
                .to("kafka:{{topic}}")
                .log("Topic: {{topic}}")
                .log("${body}");
    }

    @Override
    public void buildConsumerRoute() {
        routeTemplate(this.CONSUMER_ROUTE_TEMPLATE_NAME)
                .templateParameter("poll-topic", "POLL-TOPIC-NOT-DEFINED")
                .templateParameter("destination", "DESTINATION-NOT-DEFINED")

                // Definition
                .from("kafka:{{poll-topic}}")
                .to("{{destination}}")
                .log("Destination: {{destination}}")
                .log("${body}");
    }
}
```
### How It Works
### Configuration Overview

The `RouteConfig` interface allows defining **dynamic routes** for Apache Camel using configuration files (e.g., YAML) or environment variables. Each route can specify its **source**, **target**, and additional metadata, providing flexibility and modularity in setting up bridges.

---

### Key Features

1. **Dynamic Routing:**
   - Enables defining multiple routes in a centralized configuration file under the `config` prefix.
   - Each route includes details for:
     - **Source:** The originating system (e.g., Kafka).
     - **Target:** The destination system (e.g., Azure Service Bus).

2. **Metadata for Flexibility:**
   - Each route has detailed metadata, including:
     - **Topic Name (Optional):** The topic to route messages through.
     - **Cloud Service Queue:** Specifies the service type (e.g., `AZURE_SERVICEBUS`, `ONPREM`).
     - **Unique Identifier:** A string to uniquely identify each topic.

3. **Conditional Routing:**
   - Routes can be enabled or disabled using the `enable` flag, making it easy to toggle routes during testing or deployment.
   - Supports message transformation through the `transformMessage` flag, which allows modifying the message payload if needed. For example, you can dynamically specify a transformer class by including it in the classpath.


---

### How It Works

1. **Centralized Configurations:**
   - All route definitions are stored in a single configuration file, reducing complexity and improving manageability.

2. **Dynamic Deployment:**
   - As new routes are added or updated, they are picked up without modifying the core logic of the system.

3. **Seamless Integration:**
   - This configuration feeds into Camel templates to dynamically create bridges between systems such as:
     - **Kafka ↔ Azure Service Bus**
     - **On-Prem Kafka ↔ Cloud Services**

By centralizing and standardizing route configurations, `RouteConfig` ensures scalability and ease of management in multi-cloud and hybrid-cloud messaging systems.

## Conclusion

EventBridge, powered by Apache Camel, made it easier to handle cross-cloud messaging by simplifying the configuration of routes and abstracting the complexities of integration. With tools like `RouteConfig` and reusable templates, it allowed seamless communication between systems like **Kafka** and **Azure Service Bus** while supporting bidirectional flows.

This solution was lightweight, flexible, and easy to extend, making it a practical approach for connecting on-prem and cloud services without significant overhead.

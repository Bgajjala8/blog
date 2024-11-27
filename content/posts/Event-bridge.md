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
Customer-Defined Configuration (through YAML):
Users specify properties like source.topic, destination.queue, and connection details in a configuration file or environment variables.
Apache Camel Handles Integration:
Camel abstracts the connection to Kafka and Azure Service Bus, managing protocols, retries, and message transformations.
Bidirectional Communication:
Routes can be configured for Kafka ↔ Azure Service Bus, ensuring bidirectional message flow.
Why Apache Camel?
Apache Camel made it easy to:

Abstract complex integration logic and protocols.
Support a wide range of cloud and on-prem connectors.
Enable declarative, configuration-driven routing, allowing customers to focus on high-level workflows.
Applications and Benefits

EventBridge with Apache Camel offers several advantages:

Customer Flexibility: Customers can define their own route bridges between systems without needing deep technical expertise in the underlying protocols.
Seamless Integration: Apache Camel handles the complexities of connecting messaging systems, ensuring reliable communication.
Business Continuity: EventBridge ensures messages are delivered reliably, even across disparate environments.
Testing and Proofs of Concept: Teams can simulate multi-cloud architectures to validate their solutions before deployment.
Real-World Use Case: Kafka ↔ Azure Service Bus

A typical scenario involved connecting an on-prem Kafka cluster with Azure Service Bus:

Use Case: A legacy application produced events to Kafka. These events needed to be routed to Azure Service Bus for further processing in the cloud.
Solution: A configurable route was set up in EventBridge, allowing seamless event transfer:
Step 1: Kafka messages were consumed using the kafka component.
Step 2: Apache Camel transformed and forwarded the messages to Azure Service Bus using the azure-servicebus component.
Step 3: Logs and monitoring were enabled for visibility into the message flow.
This solution enabled the customer to transition their workload to the cloud without disrupting existing systems.
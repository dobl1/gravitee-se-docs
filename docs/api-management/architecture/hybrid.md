---
title: Gravitee.io API Management Self-Hosted
tags:
  - Architecture
hide:
  - footer
---

# Gravitee.io API Management Self-Hosted

## Components

![Hybrid Architecture Components](./../../../assets/gio-apim-hybrid-components.svg)

### SaaS Components

|                    Component                    | Description                                                  |
| :---------------------------------------------: | ------------------------------------------------------------ |
| Administration Console<br />(for API producers) | This web UI gives easy access to some key [APIM API](https://docs.gravitee.io/apim/3.x/apim_overview_components.html#gravitee-components-rest-api) services. [API Publishers](https://docs.gravitee.io/apim/3.x/apim_overview_concepts.html#gravitee-concepts-publisher) can use it to publish APIs.<br />Administrators can also configure global platform settings and specific portal settings. |
|    Dev / API Portal<br />(for API consumers)    | This web UI gives easy access to some key [APIM API](https://docs.gravitee.io/apim/3.x/apim_overview_components.html#gravitee-components-rest-api) services. [API Consumers](https://docs.gravitee.io/apim/3.x/apim_overview_concepts.html#gravitee-concepts-consumer) can use it to search for, view, try out and subscribe to a published API.<br />They can also use it to manage their [applications](https://docs.gravitee.io/apim/3.x/apim_overview_concepts.html#gravitee-concepts-application). |
|                 Management API                  | This RESTful API exposes services to manage and configure the [APIM Console](https://docs.gravitee.io/apim/3.x/apim_overview_components.html#gravitee-components-mgmt-ui) and [APIM Portal](https://docs.gravitee.io/apim/3.x/apim_overview_components.html#gravitee-components-portal-ui) web UIs.<br />All exposed services are restricted by authentication and authorization rules. For more information, see the [API Reference](https://docs.gravitee.io/apim/3.x/apim_installguide_rest_apis_documentation.html) section. |
|                SaaS API Gateways                | APIM Gateway is the core component of the APIM platform. You can think of it like a smart proxy.<br /><br />Unlike a traditional HTTP proxy, APIM Gateway has the capability to apply [policies](https://docs.gravitee.io/apim/3.x/apim_overview_plugins.html#gravitee-plugins-policies) (i.e., rules) to both HTTP requests and responses according to your needs. With these policies, you can enhance request and response processing by adding transformations, security, and many other exciting features. |
|                 Bridge Gateways                 | A *bridge* API Gateway exposes extra HTTP services for bridging HTTP calls to the underlying repository (which can be any of our supported repositories: MongoDB, JDBC and so on) |
|                 Config Database                 | All the API Management platform management data, such as API definitions, users, applications and plans. |
|         S3 Bucket + Analytics Database          | Analytics and logs data                                      |
|            [Enterprise]<br />Cockpit            | Cockpit is a centralized, multi-environments / organizations tool for managing all your Gravitee API Management and Access Management installations in a single place. |
|         [Enterprise]<br />API Designer          | Drag-and-Drop graphical (MindMap based) API designer to quickly and intuitively design your APIs (Swagger / OAS) and even deploy mocked APIs for quick testing. |
|         [Enterprise]<br />Alert Engine          | Alert Engine (AE) provides APIM and AM users with efficient and flexible API platform monitoring, including advanced alerting configuration and notifications sent through their preferred channels, such as email, Slack and using Webhooks.<br />AE does not require any external components or a database as it does not store anything. It receives events and sends notifications under the conditions which have been pre-configured upstream with triggers. |

### On-prem / Private cloud components

|        Component         | Description                                                  |
| :----------------------: | ------------------------------------------------------------ |
| Gravitee.io APIm Gaetway | APIM Gateway is the core component of the APIM platform, smartly proxing trafic applying policies. |
|         Logstash         | Collect and send local Gateways logs and metrics to the Gravitee.io APIM SaaS Control Plane. |
|          Redis           | Database use locally for rate limits synchronized counters (RateLimit, Quota, Spike Arrest) and optionnaly as an external cache for the [Cache policy](https://docs.gravitee.io/apim/3.x/apim_resources_cache_redis.html#redis_cache_resource). |

## Architecture Diagram

![Hybrid Architecture](./../../../assets/hybrid-architecture.svg)

<!-- 
    ``` mermaid
    flowchart LR
      subgraph "SaaS Shared"
        CP[Cockpit]
        AD[API Designer]
      end
      subgraph "SaaS Dedicated"
        CDB[(Config Database)]
        ADB[(Analytics Database)]
        S3[(S3 Bucket)]
        SGW[SaaS API Gateways]
        BGW[Bridge Gateways]
        MAPI[Management API]-->CDB
        MAPI-->ADB
        CS[Admin Console]-->MAPI
        DEVP[Dev Portal]-->MAPI
        AE[Alert Engine]<-->SGW
        SGW<-->CDB
        SGW<-->ADB
        BGW<-->CDB
        BGW<-->ADB
      end
      subgraph "On-prem / Private Cloud"
        RLDB[(Rate Limits Database)]
        LGW[API Gateways]<-->RLDB
        LOG[Logstash]-->LGW
      end
      LGW-->BGW
      CP-->SGW
      CP--->LGW
      AE<-->LGW
      LOG-->S3-->ADB
      AD-->MAPI
    ```
-->

### Self-Hosted to SaaS connections

![Hybrid Architecture Connections](./../../../assets/hybrid-architecture-connections.svg)
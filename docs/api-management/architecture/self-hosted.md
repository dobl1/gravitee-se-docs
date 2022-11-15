---
title: Gravitee.io API Management Hybrid
tags:
  - Architecture
hide:
  - footer
---

# Gravitee.io API Management Hybrid

## Components

<img src="./../../../assets/gio-apim-self-hosted-components.svg" alt="Self-Hosted Components" style="display: block; margin-left: auto; margin-right: auto; zoom: 40%;" />

|                    Component                    | Description                                                  |
| :---------------------------------------------: | ------------------------------------------------------------ |
| Administration Console<br />(for API producers) | This web UI gives easy access to some key [APIM API](https://docs.gravitee.io/apim/3.x/apim_overview_components.html#gravitee-components-rest-api) services. [API Publishers](https://docs.gravitee.io/apim/3.x/apim_overview_concepts.html#gravitee-concepts-publisher) can use it to publish APIs.<br />Administrators can also configure global platform settings and specific portal settings. |
|    Dev / API Portal<br />(for API consumers)    | This web UI gives easy access to some key [APIM API](https://docs.gravitee.io/apim/3.x/apim_overview_components.html#gravitee-components-rest-api) services. [API Consumers](https://docs.gravitee.io/apim/3.x/apim_overview_concepts.html#gravitee-concepts-consumer) can use it to search for, view, try out and subscribe to a published API.<br />They can also use it to manage their [applications](https://docs.gravitee.io/apim/3.x/apim_overview_concepts.html#gravitee-concepts-application). |
|                 Management API                  | This RESTful API exposes services to manage and configure the [APIM Console](https://docs.gravitee.io/apim/3.x/apim_overview_components.html#gravitee-components-mgmt-ui) and [APIM Portal](https://docs.gravitee.io/apim/3.x/apim_overview_components.html#gravitee-components-portal-ui) web UIs.<br />All exposed services are restricted by authentication and authorization rules. For more information, see the [API Reference](https://docs.gravitee.io/apim/3.x/apim_installguide_rest_apis_documentation.html) section.<br> This components might be installed twice if you need a separation between Dev Portal and Administration Console <br/>For example, to make the administration console accessible only internally (LAN), as opposed to the Dev Portal accessible from the outside (DMZ), in this scenario the Management API will only expose the operations that relates respectively to the Dev Portal and Administration Console. |
|                  API Gateways                   | APIM Gateway is the core component of the APIM platform. You can think of it like a smart proxy.<br /><br />Unlike a traditional HTTP proxy, APIM Gateway has the capability to apply [policies](https://docs.gravitee.io/apim/3.x/apim_overview_plugins.html#gravitee-plugins-policies) (i.e., rules) to both HTTP requests and responses according to your needs. With these policies, you can enhance request and response processing by adding transformations, security, and many other exciting features. |
|                 Config Database                 | Database use to store all the API Management platform management data, such as API definitions, users, applications and plans. |
|               Analytics Database                | Database use to store the variety of events occurring in the gateway. |
|              Rate Limits Database               | Database use locally for rate limits synchronized counters (Rate Limit, Quota, Spike Arrest) and optionnaly as an external cache for the [Cache policy](https://docs.gravitee.io/apim/3.x/apim_resources_cache_redis.html#redis_cache_resource). |
|            [Enterprise]<br />Cockpit            | Cockpit is a centralized, multi-environments / organizations tool for managing all your Gravitee API Management and Access Management installations in a single place. |
|         [Enterprise]<br />API Designer          | Drag-and-Drop graphical (MindMap based) API designer to quickly and intuitively design your APIs (Swagger / OAS) and even deploy mocked APIs for quick testing. |
|         [Enterprise]<br />Alert Engine          | Alert Engine (AE) provides APIM and AM users with efficient and flexible API platform monitoring, including advanced alerting configuration and notifications sent through their preferred channels, such as email, Slack and using Webhooks.<br />AE does not require any external components or a database as it does not store anything. It receives events and sends notifications under the conditions which have been pre-configured upstream with triggers. |

## Architecture diagram

![Self-Hosted Architecture](./../../../assets/gio-apim-self-hosted-architecture.svg)

### Install on VMs : LAN + DMZ deployment

![Self-Hosted Architecture LAN + DMZ](./../../../assets/gio-apim-self-hosted-architecture-vms.svg)
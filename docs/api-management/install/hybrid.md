---
title: APIm Hybrid Deployment Guide
tags:
  - API Management
  - OSS
  - Hybrid
---

# APIm Hybrid Deployment Guide

!!! info "Introduction"
    This documentation page relates to the installation of the client (On-Prem / Private Cloud) part of the API Management platform in a Hybrid architecture (SaaS + On-prem / Private cloud).

## Hybrid Architecture

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
|             [Optional]<br />Cockpit             | Cockpit is a centralized, multi-environment tool for managing all your Gravitee API Management and Access Management installations in a single place. |
|          [Optional]<br />API Designer           | Drag-and-Drop graphical designer to quickly and intuitively design your APIs and even deploy mock APIs for quick testing. |
|          [Optional]<br />Alert Engine           | Alert Engine (AE) provides APIM and AM users with efficient and flexible API platform monitoring, including advanced alerting configuration and notifications sent through their preferred channels, such as email, Slack and using Webhooks.<br />AE does not require any external components or a database as it does not store anything. It receives events and sends notifications under the conditions which have been pre-configured upstream with triggers. |

### On-prem / Private cloud components

|        Component         | Description                                                  |
| :----------------------: | ------------------------------------------------------------ |
|          Redis           | Databse use locally for rate limits counter (RateLimit, Quota, Spike Arrest) and optionnaly the as an external cache for the Cache policy. |
|         Logstash         | Collect and send local Gateways logs to SaaS.                |
| Gravitee.io APIm Gaetway | APIM Gateway is the core component of the APIM platform, smartly proxing trafic applying policies. |

## Hybrid gateway

### Installation

=== "Kubernetes (Helm)"

    !!! note "Prerequisites"
        - [Kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl)
        - [Helm v3](https://helm.sh/docs/intro/install)
    
    Steps :
    
    1. Add the Gravitee.io Helm charts repository.
      ```bash
      helm repo add graviteeio https://helm.gravitee.io
      ```
    2. Install using the `values.yaml` file.
    <br>[Here is the full `values.yaml` example](https://artifacthub.io/packages/helm/graviteeio/apim3?modal=values), please customize it following the [Configuration sections](#configuration).
      ```bash
      helm install graviteeio-apim3x graviteeio/apim3 -f values.yaml
      ```
    
    !!! info "Online documentation and assets"
        - [Install APIM on Kubernetes with the Helm Chart](https://docs.gravitee.io/apim/3.x/apim_installguide_kubernetes.html)
        - [Deploy a Hybrid architecture in Kubernetes](https://docs.gravitee.io/apim/3.x/apim_installguide_hybrid_kubernetes.html)
        - [Gravitee.io Helm Charts](https://artifacthub.io/packages/helm/graviteeio/apim3)

=== "Docker "

    !!! warning "Soon 🚧"
        This section is not redacted yet.

=== "Binaries"

    !!! warning "Soon 🚧"
        This section is not redacted yet.
    
    !!! info "Online documentation"
        - [APIM hybrid deployment](https://docs.gravitee.io/apim/3.x/apim_installguide_hybrid_deployment.html#apim_gateway_http_repository_client)

### Configuration

There is at least 3 connections to configure :

-  The connection to the SaaS Management plane with the Bridge Gateway.
-  The connection to push Analytics and Logs with file or tcp reporter pushing data for logstash to send them to the SaaS storage.
-  The connection the local rate limits database.
-  [Optional] The connection to the SaaS Alert Engine.
-  [Optional] connection to the SaaS Cockpit to enable org & multi-env support, API promotion and operational monitoring.

#### Management

=== "Kubernetes (Helm with `values.yaml` file)"

    Into the `values.yaml` configuration file :
    
    ```yaml title="values.yaml" linenums="1"
    management:
      type: http
    gateway:
      management:
        http:
          url: https://bridge-gateway-url:bridge-gateway-port
          username: kubernetes://<namespace>/secrets/<my-secret-name>/<my-secret-key>
          password: kubernetes://<namespace>/secrets/<my-secret-name>/<my-secret-key>
          # ssl:
          #   trustall: true
          #   verifyHostname: true
          #   keystore:
          #     type: jks # Supports jks, pem, pkcs12
          #     path: ${gravitee.home}/security/keystore.jks
          #     password: secret
          #   truststore:
          #     type: jks # Supports jks, pem, pkcs12
          #     path: ${gravitee.home}/security/truststore.jks
          #     password: secret
          # proxy:
          #   host:
          #   port:
          #   type: http
          #   username:
          #   password:
    ```
    
    !!! note "Online documentation"
        - [Install APIM on Kubernetes with the Helm Chart](https://docs.gravitee.io/apim/3.x/apim_installguide_kubernetes.html)
        - [Deploy a Hybrid architecture in Kubernetes](https://docs.gravitee.io/apim/3.x/apim_installguide_hybrid_kubernetes.html)
        - [Gravitee.io Helm Charts](https://artifacthub.io/packages/helm/graviteeio/apim3)

=== "Docker"

    !!! note "Soon 🚧"
        This section is not redacted yet.

=== "Gateway with `gravitee.yml` file"

    Into the `gravitee.yml` configuration file :
    
    ```yaml title="gravitee.yml" linenums="1"
    management:
      type: http
      http:
        url: https://bridge-gateway-url:bridge-gateway-port
        keepAlive: true
        idleTimeout: 30000
        connectTimeout: 10000
        authentication:
          basic:
            username: bridge-gateway-username
            password: bridge-gateway-password
        ssl:
          trustAll: true
          verifyHostname: true
          keystore:
            type: # can be jks / pem / pkcs12
            path:
            password:
          trustore:
            type: # can be jks / pem / pkcs12
            path:
            password:
    ```
    
    !!! note "Online documentation"
        - [APIM hybrid deployment](https://docs.gravitee.io/apim/3.x/apim_installguide_hybrid_deployment.html#apim_gateway_http_repository_client)

#### Analytics and Logs

=== "Kubernetes (Helm)"

    ##### Files

    Into the `values.yaml` configuration file :
    
    ```yaml title="values.yaml" linenums="1"
    gateway:
      reporters:
        elasticsearch:
          enabled: false
        file:
          enabled: true
          fileName: ${gravitee.home}/metrics/%s-yyyy_mm_dd
          output: elasticsearch # Can be csv, json, elasticsearch or message_pack
    ```

    ##### Direct (TCP)

    !!! warning
        Choosing the direct connection may result in a loss of data. If the connection between the gateway and logstash is broken the newly generated analytics and logs data will be lost. Prefer the File reporter.
    
    Into the `values.yaml` configuration file :
    
    ```yaml title="values.yaml" linenums="1"
    gateway:
      reporters:
        elasticsearch:
          enabled: false
        tcp:
          enabled: true
          host: logstash
          port: 8379
          output: elasticsearch
    ```

    !!! info "Online documentation"
          - [APIM hybrid deployment](https://docs.gravitee.io/apim/3.x/apim_installguide_hybrid_deployment.html#configuration)
          - [Full `values.yaml` example](https://artifacthub.io/packages/helm/graviteeio/apim3?modal=values)

=== "Docker"

    !!! note "Soon 🚧"
        This section is not redacted yet.

=== "Gateway with `gravitee.yml` file"

    !!! note "Soon 🚧"
        This section is not redacted yet.


#### Rate limits

!!! warning "Soon 🚧"
    This section is not redacted yet.

#### Alert Engine

=== "Kubernetes (Helm)"

    Into the `values.yaml` configuration file :
    
    ```yaml title="values.yaml" linenums="1"
    alerts:
      enabled: true
      endpoints:
        - https://alert-engine-url:alert-engine-port
      security:
        enabled: true
        username: alert-engine-username
        password: alert-engine-password
    ```
    
    !!! note "Online documentation"
        - [Integrate AE with API Management](https://docs.gravitee.io/ae/apim_installation.html#configuration)
        - [Install APIM on Kubernetes with the Helm Chart](https://docs.gravitee.io/apim/3.x/apim_installguide_kubernetes.html)
        - [Deploy a Hybrid architecture in Kubernetes](https://docs.gravitee.io/apim/3.x/apim_installguide_hybrid_kubernetes.html)
        - [Gravitee.io Helm Charts](https://artifacthub.io/packages/helm/graviteeio/apim3?modal=values&path=alerts)

=== "Docker"

    !!! note "Soon 🚧"
        This section is not redacted yet.

=== "Gateway with `gravitee.yml` file"

    !!! note "Soon 🚧"
        This section is not redacted yet.

#### Cockpit

!!! info "Follow cockpit instructions"
    Please follow directly the instruction you have on cockpit.
    `https://cockpit.gravitee.io/accounts/YOUR-ACCOUNT-HRID/installations/how-to`

#### Full example

=== "Kubernetes (Helm)"

    Into the `values.yaml` configuration file :
    
    ```yaml title="values.yaml" linenums="1"
    management:
      type: http
    gateway:
      management:
        http:
          url: https://bridge-gateway-url:bridge-gateway-port
          username: kubernetes://<namespace>/secrets/<my-secret-name>/<my-secret-key>
          password: kubernetes://<namespace>/secrets/<my-secret-name>/<my-secret-key>
      reporters:
        elasticsearch:
          enabled: false
        tcp:
          enabled: true
          host: logstash
          port: 8379
          output: elasticsearch
    alerts:
      enabled: true
      endpoints:
        - https://alert-engine-url:alert-engine-port
      security:
        enabled: true
        username: alert-engine-username
        password: alert-engine-password
    ```
    
    !!! note "Online documentation"
        - [Install APIM on Kubernetes with the Helm Chart](https://docs.gravitee.io/apim/3.x/apim_installguide_kubernetes.html)
        - [Deploy a Hybrid architecture in Kubernetes](https://docs.gravitee.io/apim/3.x/apim_installguide_hybrid_kubernetes.html)
        - [Gravitee.io Helm Charts - Values Template](https://artifacthub.io/packages/helm/graviteeio/apim3?modal=values)

=== "Docker"

    !!! note "Soon 🚧"
        This section is not redacted yet.

=== "Gateway with `gravitee.yml` file"

    !!! note "Soon 🚧"
        This section is not redacted yet.

## Redis

### Installation

- [Bitnami helm charts](https://artifacthub.io/packages/helm/bitnami/redis)

### Configuration

    !!! note "Soon 🚧"
        This section is not redacted yet.

## Logstash

### Installation

- [Official helm charts](https://artifacthub.io/packages/helm/elastic/logstash#how-to-install-oss-version-of-logstash)
- [Bitnami helm charts](https://bitnami.com/stack/logstash/helm)

### Configuration

=== "Input File - Output Stdout"
    
    ```text title="uptime.conf" linenums="1"
    
    input { 
      file {
        start_position => "beginning"
        path => "/gravitee/reporter/*.json"
      }
    }
    
    filter {
      mutate {
        add_field => { "environment" => "ugap_<ENV>_apim" }
      }
    }
    
    output {
      stdout {}
    }
    ```

=== "Input TCP - Output Elastc Search"
    
    ```text title="uptime.conf" linenums="1"
    input {
      tcp {
          port => 8379
          codec => "json"
      }
    }
    
    filter {
        if [type] != "request" {
            mutate { remove_field => ["path", "host"] }
        }
    }
    
    output {
      elasticsearch {
        hosts => ["${ES_HOSTS}"]
        user => "${ES_USER}"
        password => "${ES_PASSWORD}"
        index => "%{[@metadata][target_index]}"
      }
    }
    ```

=== "Input File - Output S3 bucket"

    ```text title="uptime.conf" linenums="1"
    input { 
      file {
        start_position => "beginning"
        path => "/gravitee/reporter/*.json"
      }
    }
    
    filter {
        if [type] != "request" {
            mutate { remove_field => ["path", "host"] }
        }
    }
    
    output {
      s3 {
        access_key_id => "${S3_ACEESS_KEY_ID}"
        secret_access_key => "${S3_SECRET_ACCESS_KEY}"
        region => "${S3_REGION}"
        bucket => "${S3_BUCKET_NAME}"
        size_file => 10485760
        codec => "json_lines"
      }
    }
    ```

=== "Input TCP - Output S3 bucket"

    ```text title="uptime.conf" linenums="1"
    input {
      tcp {
          port => 8379
          codec => "json"
      }
    }
    
    filter {
        if [type] != "request" {
            mutate { remove_field => ["path", "host"] }
        }
    }
    
    output {
      s3 {
        access_key_id => "${S3_ACEESS_KEY_ID}"
        secret_access_key => "${S3_SECRET_ACCESS_KEY}"
        region => "${S3_REGION}"
        bucket => "${S3_BUCKET_NAME}"
        size_file => 10485760
        codec => "json_lines"
      }
    }
    ```

!!! note "Online documentation"
    - [Configuring Logstash](https://www.elastic.co/guide/en/logstash/current/configuration.html)
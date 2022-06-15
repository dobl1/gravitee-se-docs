---
title: APIm Hybrid Deployment Guide
tags:
  - API Management
  - OSS
  - 3.18
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

!!! warning "Soon 🚧"
    This section is not redacted yet.

- Administration Console (for API producers)
- Dev / API Portal (for API consumers)
- Management API
- SaaS API Gateways
- Bridge Gateways
- Config Database
- S3 Bucket + Analytics Database
- [Optional] Cockpit
- [Optional] API Designer
- [Optional] Alert Engine

### On-prem / Private cloud components

|        Component         | Description                                                  |
| :----------------------: | ------------------------------------------------------------ |
|          Redis           | Databse use locally for rate limits counter (RateLimit, Quota, Spike Arrest) and optionnaly the as an external cache for the Cache policy. |
|         Logstash         | Collect and send local Gateways logs to SaaS.                |
| Gravitee.io APIm Gaetway | APIM Gateway is the core component of the APIM platform, smartly proxing trafic applying policies. |

## Hybrid gateway

### Installation

=== "Binaries (.zip)"

    !!! warning "Soon 🚧"
        This section is not redacted yet.
    
    !!! info "Online documentation"
        - [APIM hybrid deployment](https://docs.gravitee.io/apim/3.x/apim_installguide_hybrid_deployment.html#apim_gateway_http_repository_client)

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

=== "Docker 🚧 "

    !!! warning "Soon 🚧"
        This section is not redacted yet.

### Configuration

There is at least 3 connections to configure :

-  The connection to the SaaS Management plane with the Bridge Gateway.
-  The connection to push Analytics and Logs with file or tcp reporter pushing data for logstash to send them to the SaaS storage.
-  The connection the local rate limits database.
-  [Optional] The connection to the SaaS Alert Engine.
-  [Optional] connection to the SaaS Cockpit to enable org & multi-env support, API promotion and operational monitoring.

#### Management

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

=== "Docker 🚧 "

    !!! note "Soon 🚧"
        This section is not redacted yet.

#### Analytics and Logs

=== "Kubernetes (Helm) Files"

    Into the `values.yaml` configuration file :
    
    ```yaml title="values.yaml" linenums="1"
    reporters:
      elasticsearch:
        enabled: false
      file:
        enabled: true
        fileName: ${gravitee.home}/metrics/%s-yyyy_mm_dd
        output: elasticsearch # Can be csv, json, elasticsearch or message_pack
    ```

=== "Kubernetes (Helm) Direct (TCP)"

    !!! warning
        Choosing the direct connection may result in a loss of data. If the connection between the gateway and logstash is broken the newly generated analytics and logs data will be lost. Prefer the File reporter.

    Into the `values.yaml` configuration file :
    
    ```yaml title="values.yaml" linenums="1"
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







#### Rate limits

!!! warning "Soon 🚧"
    This section is not redacted yet.

#### Alert Engine

=== "Gateway with `gravitee.yml` file"

    !!! note "Soon 🚧"
        This section is not redacted yet.

=== "Kubernetes (Helm with `values.yaml` file)"

    Into the `values.yaml` configuration file :
    
    ```yaml title="values.yaml" linenums="1"
    alerts:
      enabled: true
      endpoints:
        - https://env.ae.customer.nl.az.gravitee.cloud
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

=== "Docker 🚧 "

    !!! note "Soon 🚧"
        This section is not redacted yet.

#### Cockpit

!!! info "Follow cockpit instructions"
    Please follow directly the instruction you have on cockpit.
    `https://cockpit.gravitee.io/accounts/YOUR-ACCOUNT-HRID/installations/how-to`

## Redis

### Installation

(Bitnami helm charts)[https://artifacthub.io/packages/helm/bitnami/redis]

### Configuration

Doc : https://docs.gravitee.io/ae/apim_installation.html#configuration

Into values.yml files for both the Gateway and the API :

```yaml title="values.yml"
alerts:
  enabled: true
  endpoints:
    - https://dev.ae.ugap.nl.az.gravitee.cloud
  security:
    enabled: true
    username: admin
    password: <PASS>
```

## Logstash

### Installation

Official helm charts : https://artifacthub.io/packages/helm/elastic/logstash#how-to-install-oss-version-of-logstash
Bitnami helm charts : https://bitnami.com/stack/logstash/helm

### Configuration

=== "Input File - Output Stdout"

    Into the `uptime.conf` configuration file :
    
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

    Into the `uptime.conf` configuration file :

    Doc : [Configuring Logstash]([https://](https://www.elastic.co/guide/en/logstash/current/configuration.html))
    
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

into values.yml passed to helm when installing :

```yaml title="values.yml"
logstashConfig:
  logstash.yml: |
    http.host: 0.0.0.0
    xpack.monitoring.enabled: false
    pipeline.ecs_compatibility: disabled

logstashPipeline:
  uptime.conf: |
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

## Full APIm values.yml config file example

```yaml title="values.yml"
management:
    type: http
gateway:
  replicaCount: 1
  management:
    http:
      url: 
      username: 
      password: 
  ingress:
    path: /
    hosts:
      - demo-hybrid-apim-gw.cloud.gravitee.io
    # tls:
    # -   hosts:
    #         - demo-hybrid-apim-gw.cloud.gravitee.io
    #     secretName: cloud-gravitee-cert
  reporters:
    elasticsearch:
      enabled: false
    file: # https://docs.gravitee.io/apim/3.x/apim_installguide_gateway_configuration.html
      enabled: true # Is the reporter enabled or not (default to false)
      fileName: ${gravitee.home}/metrics/%s-yyyy_mm_dd
      output: elasticsearch # Can be csv, json, elasticsearch or message_pack
ratelimit:
  type: redis             
  redis:                    # redis repository
    host:                   # redis host (default localhost)
    port:                   # redis port (default 6379)
    password:               # redis password (default null)
    timeout:                # redis timeout (default -1)
elasticsearch:
  enabled: false
api:
  enabled: false
ui:
  enabled: false
portal:
  enabled: false
```
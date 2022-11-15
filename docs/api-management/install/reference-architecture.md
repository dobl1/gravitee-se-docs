---
title: Gravitee.io API Management Reference Architecture
tags:
  - Architecture
hide:
  - footer
---

# Gravitee.io API Management Reference Architecture

## Architecture

!!! Info "Architecture"
    You can find all architecture information (components descriptions, diagrams) in the [architecture section](../architecture/index.md).

## Production Best Practices

### High Availability : increasing resilience and uptime

The **3 Main Principles** to reduce scheduled and unscheduled downtime :

1. **Elimination of single points of failure** (SPOF)
   **→** Adding **redundancy** by running at least 2 G.io APIm gateways and 2 Alert engine instances

2. **Reliable crossover**
   **→** Reliable **load-balancer** in front of G.io APIm gateways
   (Nginx, HAproxy, F5, Traefik, Squid, Kemp, LinuxHA, …).

3. **Detection of failures** as they occur
   **→** **Active monitoring** of G.io APIm gateways.

#### 1. Elimination of single points of failure

Adding redundancy by running at least 2 G.io APIm gateways and 2 Alert engine instances in Active/Active or Active/Passive mode.

<img src="./../../../assets/load-balancing.svg" alt="Load Balancing" style="display: block; margin-left: auto; margin-right: auto; width: 60%;" />

!!! Info "Installation on VMs"
    (If you're installing in VMs) Use dedicated VMs for the gateways and alert engine instances.

#### 2. Reliable crossover 
Use a reliable load-balancer in front of G.io APIm gateways (Nginx, HAproxy, F5, Traefik, Squid, Kemp, LinuxHA, …) and Active or Passive health checks.

|                         | **Active health checks**                                     | **Passive health checks (circuit breakers)**                 |
| ----------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Re-enable a backend** | Active health checks can automatically re-enable a backend  in the backend group as soon as it is healthy again. | Passive health checks cannot.                                |
| **Additional traffic**  | Active health checks do produce additional traffic to the  target. | Passive health checks do not produce additional traffic to  the target. |
| **Probe endpoint**      | An active health checker demands a known URL with a  reliable status response in the backend to be configured as a request  endpoint (which may be as simple as "/"). By providing a  custom probe endpoint for an active health checker, an backend may determine  its own health metrics and produce a status code to be consumed by Gravitee.  Even though a target continues to serve traffic which looks healthy to the  passive health checker, it would be able to respond to the active probe with  a failure status, essentially requesting to be relieved from taking new  traffic. | Passive health checks do not demand such configuration.      |


#### 3. Detection of failures as they occur

Active monitoring of G.io APIm gateways and mAPI health using :

- [Gateway internal API](https://docs.gravitee.io/apim/3.x/apim_installguide_gateway_technical_api.html), a RESTful endpoint to get node status :
    - **GET /_node** : Gets generic node information : version, revision, name, …
    - **GET /_node/health?probes=#probe1,...** : Gets the health status. Probes can be filtered using the optional probes query param.
    - **GET /_node/monitor** : Gets monitoring information from the JVM and the server.
    - **GET /_node/apis** : Gets the APIs deployed and their config on this APIM Gateway instance.
- An API with [mock policy](https://docs.gravitee.io/apim/3.x/apim_policies_mock.html) to perform active health checks.
- [Prometheus](https://docs.gravitee.io/apim/3.x/apim_installguide_gateway_metrics.html) to expose metrics : (/_node/metrics/prometheus), to get Vert.x 4 metrics (customizables with labels)
- [OpenTracing](https://docs.gravitee.io/apim/3.x/apim_enable_opentracing_with_jaeger.html) with Jaeger to trace every request that comes through the APIM Gateway. Creating a deep level of insight on API policies and making debugging a cinch.

## Capacity Planning

### Storage

Storage is mostly a concern at the analytics database level and depends on :

- The architecture requirements (redundancy, backups)
- The APIs configurations (are advanced logs activated on requests and responses payloads especially)
- The APIs rate (RPS : Requests Per Second)
- The APIs payload sizes

!!! warning "Avoid systematic advanced logs on all APIs requests and responses"
    If you have activated the advanced logs on requests and responses payloads, with an average (requests + responses) payload size of 10kB, and you have 10 RPS, and you want to keep the logs for 6 months, it will take 1.5 TB of storage. It might be fine for some use cases, but keep in mind that activating the advanced logs on all API requests and responses will generate a lot of data and reduce the gateway capacities as well.

### Memory

Memory is highly dependent on use cases.
The more APIs you will have that load payloads in memory (encryption policy, payload transformation, advanced logs, ...) the more the memory consumption will increase.

### CPU

The CPU load is directly related to the API traffic.
It is the metric to follow to evaluate the load level of the gateways and the one to use to determine the horizontal scalability (CPU > 75% for example)

## Hardware recommendations for self-hosted deployment

| **Component**                                                |                           **vCPU**                           |                         **RAM (GB)**                         |                   **Disk (Go)**                   |
| :----------------------------------------------------------- | :----------------------------------------------------------: | :----------------------------------------------------------: | :-----------------------------------------------: |
| Dev Portal + REST API (dev portal only)                      |                              1                               |                              2                               |                        20                         |
| Console + REST API (console only)                            |                              1                               |                              2                               |                        20                         |
| Dev Portal + Console +  REST API                             |                              2                               |                              4                               |                        20                         |
| APIm Gateway instance<br>Production best practice (HA) is 2 nodes |                           0.25 - 4                           |                          512 MB - 8                          |                        20                         |
| Alert Engine instance<br>Production best practice (HA) is 2 nodes |                           0.25 - 4                           |                          512 MB - 8                          |                        20                         |
| Analytics DB instance (ElasticSearch)<br>[Production best practice is 3 nodes](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/setup.html)<br>[Offical hardware recommendations](https://www.elastic.co/guide/en/elasticsearch/guide/master/hardware.html) | [1](https://github.com/elastic/helm-charts/blob/main/elasticsearch/values.yaml#L90) - [8](https://www.elastic.co/guide/en/elasticsearch/guide/master/hardware.html#_cpus) | [2](https://github.com/elastic/helm-charts/blob/main/elasticsearch/values.yaml#L90) - [8](https://www.elastic.co/guide/en/elasticsearch/guide/master/hardware.html#_memory)  or more | 20 + 0.5 per million requests for default metrics |
| Config DB instance (MongoDB or JDBC DB)<br/>[Production best practice is 3 nodes](https://www.mongodb.com/docs/manual/administration/production-notes) |                              1                               |                              2                               |                        30                         |
| Rate Limit DB instance (Redis)<br/>[Production best practice is 3 nodes](https://docs.redis.com/latest/rs/installing-upgrading/hardware-requirements/#productionenvironment) |                              2                               |                              4                               |                        20                         |




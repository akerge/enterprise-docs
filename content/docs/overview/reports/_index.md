---
title: "Anchore Enterprise Reports"
linkTitle: "Reports"
weight: 4
---

### Overview

Added in Anchore Enterprise v2.0

Anchore Enterprise Reports is a new service introduced in v2.0. It aggregates data from Anchore Engine to provide insightful analytics and metrics for account-wide artifacts. The service employs GraphQL to expose a rich API for querying the aggregated data and metrics.  

> NOTE: This service captures a snapshot of artifacts in Anchore Engine at a given point in time. Therefore analytics and metrics computed by the service are not in real time, and may not reflect most up-to-date state in Anchore Engine  


### How it works

One of the main functions of Anchore Enterprise Reports is aggregating data from Anchore Engine. The service keeps a summary of all current and historical images and tags for every account known to Anchore Engine. It also maintains vulnerability reports and policy evaluations generated using the active bundle for all the images and tags respectively. 

> WARNING: Anchore Enterprise Reports shares persistence layer with Anchore Engine. Ensure sufficient storage is provisioned

#### Data ingress

Reports service handles data ingress from Anchore Engine via the following asynchronous processes triggered periodically - 

- **Loader** Compares the working set of images and tags in Anchore Engine with its own records. Based on the difference, images and tags along with the vulnerability report and policy evaluations are loaded into the service. Artifacts deleted from Anchore Engine are marked inactive in the service. This process is triggered periodically at 10 minute (600 seconds) intervals, the interval can be configured by overriding the `ANCHORE_ENTERPRISE_REPORTS_DATA_LOAD_INTERVAL_SEC` environment variable in the container.    
- **Refresher** Refreshes the vulnerability report and policy evaluations of all the images and tags actively maintained by the service. This process is triggered periodically at a 2 hour (7200 seconds) interval, the interval can be configured by overriding the `ANCHORE_ENTERPRISE_REPORTS_DATA_REFRESH_INTERVAL_SEC` environment variable in the container.

Data ingress is enabled by default. It can be turned off by overriding `ANCHORE_ENTERPRISE_REPORTS_ENABLE_DATA_INGRESS` environment variable to `false` in the container. With the ingress turned off, Reports service will not aggregate data from Anchore Engine and halt metric value computation. The service, however, will continue to serve API requests with the existing data.

> WARNING: Reports service may miss updates to artifacts if they are added and deleted in between consecutive ingress process runs 

#### Metrics

Reports service comes loaded with a few pre-defined/canned metric definitions. A metric definition is an identifier, readable name, description and the type of the metric. The type is loosely based on statsd metric types. All the pre-defined metrics are of type 'counter' - a measure of the number of items that match certain criteria. A value for each of these metric definitions is computed using the data aggregated by the service. All metric values are computed periodically at a 1 hour interval, the interval can be configured by overriding `ANCHORE_ENTERPRISE_REPORTS_METRICS_INTERVAL_SEC` environment variable in the container


### Installation and Configuration

- Anchore Enterprise Reports is included with Anchore Enterprise, and is installed by default when deploying a trial quickstart with [Docker Compose]({{< ref "/docs/installation/docker_compose" >}}) or a production deployment [Kubernetes]({{< ref "/docs/installation/helm" >}}).
- In an Anchore Enterprise RBAC enabled deployment, any non-admin account user must at least have _listImages_ permission to execute queries against Reports API


### See it in action

To see the Anchore Enterprise Reports in action open the [Dashboard]({{< ref "/docs/using/ui_usage/dashboard" >}}) in the Enterprise UI and explore the available widgets




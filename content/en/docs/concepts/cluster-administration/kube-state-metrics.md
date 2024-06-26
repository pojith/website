---
title: Metrics for Kubernetes Object States
content_type: concept
weight: 75
description: >-
   kube-state-metrics, an add-on agent to generate and expose cluster-level metrics.
---

The state of Kubernetes objects in the Kubernetes API can be exposed as metrics.
An add-on agent called [kube-state-metrics](https://github.com/kubernetes/kube-state-metrics) can connect to the Kubernetes API server and expose a HTTP endpoint with metrics generated from the state of individual objects in the cluster.
It exposes various information about the state of objects like labels and annotations, startup and termination times, status or the phase the object currently is in.
For example, containers running in pods create a `kube_pod_container_info` metric.
This includes the name of the container, the name of the pod it is part of, the {{< glossary_tooltip text="namespace" term_id="namespace" >}} the pod is running in, the name of the container image, the ID of the image, the image name from the spec of the container, the ID of the running container and the ID of the pod as labels.

{{% thirdparty-content single="true" %}}

An external component that is able and capable to scrape the endpoint of kube-state-metrics (for example via Prometheus) can now be used to enable the following use cases.

## Example: using metrics from kube-state-metrics to query the cluster state {#example-kube-state-metrics-query-1}

Metric series generated by kube-state-metrics are helpful to gather further insights into the cluster, as they can be used for querying.

If you use Prometheus or another tool that uses the same query language, the following PromQL query returns the number of pods that are not ready:

```
count(kube_pod_status_ready{condition="false"}) by (namespace, pod)
```

## Example: alerting based on from kube-state-metrics {#example-kube-state-metrics-alert-1}

Metrics generated from kube-state-metrics also allow for alerting on issues in the cluster.

If you use Prometheus or a similar tool that uses the same alert rule language, the following alert will fire if there are pods that have been in a `Terminating` state for more than 5 minutes:

```yaml
groups:
- name: Pod state
  rules:
  - alert: PodsBlockedInTerminatingState
    expr: count(kube_pod_deletion_timestamp) by (namespace, pod) * count(kube_pod_status_reason{reason="NodeLost"} == 0) by (namespace, pod) > 0
    for: 5m
    labels:
      severity: page
    annotations:
      summary: Pod {{$labels.namespace}}/{{$labels.pod}} blocked in Terminating state.
```

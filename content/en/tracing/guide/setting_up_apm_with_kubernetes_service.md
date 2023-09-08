---
title: Setting up APM with Kubernetes Service
kind: guide
further_reading:
- link: "/containers/kubernetes/apm/"
  tag: "Documentation"
  text: "Set up trace collection"
- link: "/containers/cluster_agent/admission_controller"
  tag: "Documentation"
  text: "Admission Controller"
---

## Overview

There are 3 main ways the Datadog Tracers send their data to the Datadog Agent for the proper handling. Those are: Host IP, Unix Domain Socket (UDS), and the Kubernetes service. These options are all setup in a way to ensure the APM data the application pod sends arrive to the Datadog Agent pod on the same node. This strategy is meant to properly balance the traffic and ensure the correct tagging of your data.

Datadog recommends that you use UDS to send data. However, if the `hostPath` volumes required for UDS (and the `hostPort` ports required for using host IP) are not available, you can use a Kubernetes service as an alternative option.

## Service setup

In Kubernetes you can take advantage of [services][1] to help route traffic to downstream pods. One limitation of the default service configuration is that this routes the traffic to the downstream pods at random. In Kubernetes 1.22 the [Internal Traffic Policy feature][2] was added, giving the option to set the configuration `internalTrafficPolicy: Local` on the service. When set this directs the traffic from an application pod to the service's downstream pod *on the same node*. Maintaining that strategy of routing the APM data to the Datadog Agent on the same node.

This service is created for you automatically in our Datadog Helm Chart and Datadog Operator on Kubernetes clusters with version 1.22.0 or above. You additionally need to enable the APM port option for your Agent with the below configuration.

### Agent configuration
{{< tabs >}}
{{% tab "Helm" %}}

```yaml
datadog:
  apm:
    portEnabled: true
```    

{{% /tab %}}
{{% tab "Operator" %}}

```yaml
apiVersion: datadoghq.com/v2alpha1
kind: DatadogAgent
metadata:
  name: datadog
spec:
  #(...)
  features:
    apm:
      enabled: true
```
{{% /tab %}}
{{< /tabs >}}

## Application configuration
You can configure your application to use the Kubernetes service manually, or by using the [Cluster Agent Admission Controller][3].

### Admission controller
The [Cluster Agent's Admission Controller][3] can simplify the configuration for APM by injecting the environment variables for connectivity. This has the configuration mode of `hostip`, `socket`, or `service` relative to the 3 options. When choosing the configuration mode of `service` the Admission Controller adds the `DD_AGENT_HOST` with the proper service name and namespace format.

{{< tabs >}}
{{% tab "Helm" %}}

```yaml
clusterAgent:
  admissionController:
    enabled: true
    configMode: service
```    

{{% /tab %}}
{{% tab "Operator" %}}

```yaml
apiVersion: datadoghq.com/v2alpha1
kind: DatadogAgent
metadata:
  name: datadog
spec:
  #(...)
  features:
    apm:
      enabled: true
    admissionController:
      enabled: true
      agentCommunicationMode: service
```

{{% /tab %}}
{{< /tabs >}}

**Note:** In mixed node (Linux/Windows) environments, the Cluster Agent and its Admission Controller are relative to the Linux deployment. This may inject the wrong environment variables for the service connectivity in the Windows pods. 

### Manual configuration
For manual configuration, set the environment variable `DD_AGENT_HOST` within your pod manifest, with a value of `<SERVICE_NAME>.<SERVICE_NAMESPACE>.svc.cluster.local`.

```yaml
    #(...)
    spec:
      containers:
      - name: "<CONTAINER_NAME>"
        image: "<CONTAINER_IMAGE>"
        env:
          - name: DD_AGENT_HOST
            value: <SERVICE_NAME>.<SERVICE_NAMESPACE>.svc.cluster.local
```

Replace `<SERVICE_NAME>` with the service's name, and replace `<SERVICE_NAMESPACE>` with the service's namespace.

 For example, if your service definition looks like the following:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: datadog
  namespace: monitoring
  labels:
    #(...)
spec:
  selector:
    app: datadog
  ports:
    - protocol: UDP
      port: 8125
      targetPort: 8125
      name: dogstatsdport
    - protocol: TCP
      port: 8126
      targetPort: 8126
      name: traceport
  internalTrafficPolicy: Local
```

Then set the value of `DD_AGENT_HOST` to `datadog.monitoring.svc.cluster.local`.

```yaml
    #(...)
    spec:
      containers:
      - name: "<CONTAINER_NAME>"
        image: "<CONTAINER_IMAGE>"
        env:
          - name: DD_AGENT_HOST
            value: datadog.monitoring.svc.cluster.local

## Further reading

{{< partial name="whats-next/whats-next.html" >}}

[1]: https://kubernetes.io/docs/concepts/services-networking/service/
[2]: https://kubernetes.io/docs/concepts/services-networking/service-traffic-policy/
[3]: /containers/cluster_agent/admission_controller
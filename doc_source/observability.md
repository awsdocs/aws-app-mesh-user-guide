# App Mesh observability<a name="observability"></a>

The Envoy proxy and App Mesh offer the following tools to help you gain a clearer view of your applications and proxies:
+ Access logs
+ Statistics
+ Proxy logs

## Access logs<a name="envoy-logs"></a>

When you create your virtual nodes, you have the option to configure Envoy access logs\. In the console, this is in the **Advanced configuration** section of the virtual node create or update workflows\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/app-mesh/latest/userguide/images/logging.png)

The preceding image shows a logging path of `/dev/stdout` for Envoy access logs\. The following code block shows the JSON representation that you can use in the AWS CLI\.

```
      "logging": { 
         "accessLog": { 
            "file": { 
               "path": "/dev/stdout"
            }
         }
      }
```

When you send Envoy access logs to `/dev/stdout`, they are mixed in with the Envoy container logs\. You can export them to a log storage and processing service like CloudWatch Logs using standard Docker log drivers such as `awslogs`\. For more information, see [Using the awslogs Log Driver](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/using_awslogs.html) in the Amazon ECS Developer Guide\. To export only the Envoy access logs \(and ignore the other Envoy container logs\), you can set the `ENVOY_LOG_LEVEL` to `off`\. For more information, see [Access logging](https://www.envoyproxy.io/docs/envoy/latest/configuration/observability/access_log/access_log.html) in the Envoy documentation\.

**Enable access logs on Kubernetes**  
When using the [App Mesh Controller for Kubernetes](https://docs.aws.amazon.com/app-mesh/latest/userguide/mesh-k8s-integration.html), you can configure virtual nodes with access logging by adding the logging configuration to the virtual node spec, as shown in the following example\.

```
---
apiVersion: appmesh.k8s.aws/v1beta2
kind: VirtualNode
metadata:
  name: virtual-node-name
  namespace: namespace
spec:
  listeners:
    - portMapping:
        port: 9080
        protocol: http
  serviceDiscovery:
    dns:
      hostName: hostname
  logging:
    accessLog:
      file:
        path: "/dev/stdout"
```

Your cluster must have a log forwarder to collect these logs, such as Fluentd\. For more information see, [Set up Fluentd as a DaemonSet to send logs to CloudWatch Logs](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Container-Insights-setup-logs.html)\.

## Statistics<a name="logs"></a>

Envoy emits many statistics on both its own operation and various dimensions on inbound and outbound traffic\. To learn more about Envoy statistics, see [Statistics](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/observability/statistics) in the Envoy documentation\. These metrics are available through the `/stats` endpoint on the proxy’s administration port, which is typically `9901`\. For more information about the stats endpoint, see [Statistics endpoint](https://www.envoyproxy.io/docs/envoy/latest/operations/admin#get--stats) in the Envoy documentation\. For more information about the administration interface, see [Enable the Envoy proxy administration interface](troubleshooting-best-practices.md#ts-bp-enable-proxy-admin-interface)\.

**Prometheus on Kubernetes**  
Prometheus is an open\-source monitoring and alerting toolkit\. One of its capabilities is to specify a format for emitting metrics that can be consumed by other systems\. For more information about Prometheus, see [Overview](https://prometheus.io/docs/introduction/overview/) in the Prometheus documentation\. Envoy can emit its metrics via its stats endpoint by passing in the parameter `/stats?format=prometheus`\.

Amazon CloudWatch supports both the discovery and collection of Prometheus metrics\. For more information, see [Container Insights Prometheus Metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ContainerInsights-Prometheus.html)\. You can enable your Kubernetes and Amazon EKS clusters to emit metrics to CloudWatch\. As of July, 2020, CloudWatch support for Prometheus metrics is in the beta stage\.

**Emitting Envoy stats to CloudWatch from Amazon EKS**  
You can install the CloudWatch Agent to your cluster and configure it to collect a subset of metrics from your proxies\. If you do not already have an Amazon EKS cluster, then you can create one with the steps in [Walkthrough: App Mesh with Amazon EKS](https://github.com/aws/aws-app-mesh-examples/tree/master/walkthroughs/eks) on GitHub\. You can install a sample application onto the cluster by following the same walkthrough\.

To set the appropriate IAM permissions for your cluster and install the agent, follow the steps in [Install the CloudWatch Agent with Prometheus Metrics Collection](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ContainerInsights-Prometheus-Setup.html)\. The default installation contains a Prometheus scrape configuration which pulls a useful subset of Envoy stats\. For more information, see [Prometheus Metrics for App Mesh](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ContainerInsights-Prometheus-metrics.html#ContainerInsights-Prometheus-metrics-appmesh)\.

To create an App Mesh custom CloudWatch dashboard configured to display the metrics that the agent is collecting, follow the steps in the [Viewing Your Prometheus Metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ContainerInsights-Prometheus-viewmetrics.html) tutorial\. Your graphs will begin to populate with the corresponding metrics as traffic enters the App Mesh application\.

## Proxy logs<a name="proxy-logs"></a>

Envoy writes various debugging logs from its filters to `stdout`\. These logs are useful for gaining insights into both Envoy’s communication with App Mesh and service\-to\-service traffic\. Your specific logging level can be configured using the `ENVOY_LOG_LEVEL` environment variable\. For example, the following text is from an example debug log showing the cluster that Envoy matched for a particular HTTP request\.

```
[debug][router] [source/common/router/router.cc:434] [C4][S17419808847192030829] cluster 'cds_ingress_howto-http2-mesh_color_client_http_8080' match for URL '/ping'
```
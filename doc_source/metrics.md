# Exporting metrics<a name="metrics"></a>

Envoy emits many statistics on both its own operation and various dimensions on inbound and outbound traffic\. To learn more about Envoy statistics, see [Statistics](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/observability/statistics) in the Envoy documentation\. These metrics are available through the `/stats` endpoint on the proxy’s administration port, which is typically `9901`\. For more information about the stats endpoint, see [Statistics endpoint](https://www.envoyproxy.io/docs/envoy/latest/operations/admin#get--stats) in the Envoy documentation\. For more information about the administration interface, see [Enable the Envoy proxy administration interface](troubleshooting-best-practices.md#ts-bp-enable-proxy-admin-interface)\.

## CloudWatch for App Mesh<a name="cloudwatch"></a>

**Emitting Envoy stats to CloudWatch from Amazon EKS**  
You can install the CloudWatch Agent to your cluster and configure it to collect a subset of metrics from your proxies\. If you do not already have an Amazon EKS cluster, then you can create one with the steps in [Walkthrough: App Mesh with Amazon EKS](https://github.com/aws/aws-app-mesh-examples/tree/master/walkthroughs/eks) on GitHub\. You can install a sample application onto the cluster by following the same walkthrough\.

To set the appropriate IAM permissions for your cluster and install the agent, follow the steps in [Install the CloudWatch Agent with Prometheus Metrics Collection](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ContainerInsights-Prometheus-Setup.html)\. The default installation contains a Prometheus scrape configuration which pulls a useful subset of Envoy stats\. For more information, see [Prometheus Metrics for App Mesh](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ContainerInsights-Prometheus-metrics.html#ContainerInsights-Prometheus-metrics-appmesh)\.

To create an App Mesh custom CloudWatch dashboard configured to display the metrics that the agent is collecting, follow the steps in the [Viewing Your Prometheus Metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ContainerInsights-Prometheus-viewmetrics.html) tutorial\. Your graphs will begin to populate with the corresponding metrics as traffic enters the App Mesh application\.

### CloudWatch Example<a name="cloudwatch-sample"></a>

You can find a sample configuration of CloudWatch in our [AWS Samples repository](https://github.com/aws-samples/aws-app-mesh-cloudwatch-agent)\.

### Walkthroughs for using the CloudWatch<a name="cloudwatch-walkthrough"></a>
+ [Add monitoring and logging capabilities](https://www.appmeshworkshop.com/monitoring/) in our [App Mesh workshop](https://www.appmeshworkshop.com/introduction/)\.
+ [App Mesh with EKS—Observability: CloudWatch](https://github.com/aws/aws-app-mesh-examples/blob/master/walkthroughs/eks/o11y-cloudwatch.md)

## Prometheus for App Mesh with Amazon EKS<a name="prometheus"></a>

Prometheus is an open\-source monitoring and alerting toolkit\. One of its capabilities is to specify a format for emitting metrics that can be consumed by other systems\. For more information about Prometheus, see [Overview](https://prometheus.io/docs/introduction/overview/) in the Prometheus documentation\. Envoy can emit its metrics via its stats endpoint by passing in the parameter `/stats?format=prometheus`\.

### Installing Prometheus<a name="installing-prometheus"></a>

1. Add the EKS repository to Helm:

   ```
   helm repo add eks https://aws.github.io/eks-charts
   ```

1. Install App Mesh Prometheus

   ```
   helm upgrade -i appmesh-prometheus eks/appmesh-prometheus \
   --namespace appmesh-system
   ```

### Prometheus Example<a name="prometheus-sample"></a>

The following is an example of creating a `PersistentVolumeClaim` for Prometheus persistent storage\.

```
helm upgrade -i appmesh-prometheus eks/appmesh-prometheus \
--namespace appmesh-system \
--set retention=12h \
--set persistentVolumeClaim.claimName=prometheus
```

### Walkthrough for using the Prometheus<a name="prometheus-walkthrough"></a>
+ [App Mesh with EKS—Observability: Prometheus](https://github.com/aws/aws-app-mesh-examples/blob/master/walkthroughs/eks/o11y-prometheus.md)

### To learn more about Prometheus and Prometheus with Amazon EKS<a name="prometheus-eks"></a>
+ [Prometheus Documentation](https://prometheus.io/docs/introduction/overview/)
+ **EKS \-** [Control plane metrics with Prometheus](https://docs.aws.amazon.com/eks/latest/userguide/prometheus.html)

## Datadog for App Mesh<a name="datadog"></a>

Datadog is a monitoring and security applicaton for end to end monitoring, metrics, and logging of cloud applications\. Datadog makes your infrastructure, applications, and third\-party applications completely observable\.

### Installing Datadog<a name="installing-datadog"></a>
+ EKS \- To setup Datadog with EKS, follow these steps from the [Datadog docs](https://docs.datadoghq.com/integrations/amazon_app_mesh/?tab=eks)\.
+ ECS EC2 \- To set up Datadog with ECS EC2, follow these steps from the [Datadog docs](https://docs.datadoghq.com/integrations/amazon_app_mesh/?tab=ecsec2)\.

### To learn more about Datadog<a name="datadog-learn-more"></a>
+ [Datadog Documentation](https://docs.datadoghq.com/)
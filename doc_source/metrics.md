# Exporting metrics<a name="metrics"></a>

Envoy emits many statistics on both its own operation and various dimensions on inbound and outbound traffic\. To learn more about Envoy statistics, see [Statistics](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/observability/statistics) in the Envoy documentation\. These metrics are available through the `/stats` endpoint on the proxy’s administration port, which is typically `9901`\.

The `stat` prefix will be different depending on if you're using single or multiple listeners\. Below are some examples to illustrate the differences\.

**Warning**  
If you update your single listener to the multiple listener feature, you can face a breaking change due to the updated stat prefix illustrated in the following table\.  
 We suggest you use Envoy image `1.22.2.1-prod` or later\. This allows you to see similar metric names in your Prometheus endpoint\.


| Single Listener \(SL\)/Existing stats with "ingress" listener prefix | Multiple Listeners \(ML\)/New stats with "ingress\.<protocol>\.<port>" listener prefix | 
| --- | --- | 
|  `http.*ingress*.rds.rds_ingress_http_5555.version_text`  |  `http.*ingress.http.5555*.rds.rds_ingress_http_5555.version_text` `http.*ingress.http.6666*.rds.rds_ingress_http_6666.version_text`  | 
|  `listener.0.0.0.0_15000.http.*ingress*.downstream_rq_2xx`  |  `listener.0.0.0.0_15000.http.*ingress.http.5555*.downstream_rq_2xx` `listener.0.0.0.0_15000.http.*ingress.http.6666*.downstream_rq_2xx`  | 
|  `http.*ingress*.downstream_cx_length_ms`  |  `http.*ingress.http.5555*.downstream_cx_length_ms` `http.*ingress.http.6666*.downstream_cx_length_ms`  | 

For more information about the stats endpoint, see [Statistics endpoint](https://www.envoyproxy.io/docs/envoy/latest/operations/admin#get--stats) in the Envoy documentation\. For more information about the administration interface, see [Enable the Envoy proxy administration interface](troubleshooting-best-practices.md#ts-bp-enable-proxy-admin-interface)\.

## Prometheus for App Mesh with Amazon EKS<a name="prometheus"></a>

Prometheus is an open\-source monitoring and alerting toolkit\. One of its capabilities is to specify a format for emitting metrics that can be consumed by other systems\. For more information about Prometheus, see [Overview](https://prometheus.io/docs/introduction/overview/) in the Prometheus documentation\. Envoy can emit its metrics via its stats endpoint by passing in the parameter `/stats?format=prometheus`\.

For customers that are using Envoy image build v1\.22\.2\.1\-prod, there are two additional dimensions to indicate ingress listener specific stats:
+ `appmesh.listener_protocol`
+ `appmesh.listener_port`

Below is a comparison between Prometheus existing stats vs new stats\.
+ Existing stats with "ingress" listener prefix

  ```
  envoy_http_downstream_rq_xx{appmesh_mesh="multiple-listeners-mesh",appmesh_virtual_node="foodteller-vn",envoy_response_code_class="2",envoy_http_conn_manager_prefix="ingress"} 931433
  ```
+ New stats with "ingress\.<protocol>\.<port>" \+ Appmesh Envoy Image v1\.22\.2\.1\-prod or later

  ```
  envoy_http_downstream_rq_xx{appmesh_mesh="multiple-listeners-mesh",appmesh_virtual_node="foodteller-vn",envoy_response_code_class="2",appmesh_listener_protocol="http",appmesh_listener_port="5555",envoy_http_conn_manager_prefix="ingress"} 20
  ```
+ New stats with "ingress\.<protocol>\.<port>" \+ custom Envoy Imagebuild

  ```
  envoy_http_http_5555_downstream_rq_xx{appmesh_mesh="multiple-listeners-mesh",appmesh_virtual_node="foodteller-vn",envoy_response_code_class="2",envoy_http_conn_manager_prefix="ingress"} 15983
  ```

For multiple listeners, the `cds_ingress_<mesh name>_<virtual gateway name>_self_redirect_<ingress_listener_port>_<protocol>_<port>` special cluster will be listener specific\.
+ Existing stats with "ingress" listener prefix

  ```
  envoy_cluster_assignment_stale{appmesh_mesh="multiple-listeners-mesh",appmesh_virtual_gateway="tellergateway-vg",Mesh="multiple-listeners-mesh",VirtualGateway="tellergateway-vg",envoy_cluster_name="cds_ingress_multiple-listeners-mesh_tellergateway-vg_self_redirect_http_15001"} 0
  ```
+ New stats with "ingress\.<protocol>\.<port>"

  ```
  envoy_cluster_assignment_stale{appmesh_mesh="multiple-listeners-mesh",appmesh_virtual_gateway="tellergateway-vg",envoy_cluster_name="cds_ingress_multiple-listeners-mesh_tellergateway-vg_self_redirect_1111_http_15001"} 0
  envoy_cluster_assignment_stale{appmesh_mesh="multiple-listeners-mesh",appmesh_virtual_gateway="tellergateway-vg",envoy_cluster_name="cds_ingress_multiple-listeners-mesh_tellergateway-vg_self_redirect_2222_http_15001"} 0
  ```

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

### Walkthrough for using Prometheus<a name="prometheus-walkthrough"></a>
+ [App Mesh with EKS—Observability: Prometheus](https://github.com/aws/aws-app-mesh-examples/blob/main/walkthroughs/eks/o11y-prometheus.md)

### To learn more about Prometheus and Prometheus with Amazon EKS<a name="prometheus-eks"></a>
+ [Prometheus Documentation](https://prometheus.io/docs/introduction/overview/)
+ **EKS \-** [Control plane metrics with Prometheus](https://docs.aws.amazon.com/eks/latest/userguide/prometheus.html)

## CloudWatch for App Mesh<a name="cloudwatch"></a>

**Emitting Envoy stats to CloudWatch from Amazon EKS**  
You can install the CloudWatch Agent to your cluster and configure it to collect a subset of metrics from your proxies\. If you do not already have an Amazon EKS cluster, then you can create one with the steps in [Walkthrough: App Mesh with Amazon EKS](https://github.com/aws/aws-app-mesh-examples/tree/main/walkthroughs/eks) on GitHub\. You can install a sample application onto the cluster by following the same walkthrough\.

To set the appropriate IAM permissions for your cluster and install the agent, follow the steps in [Install the CloudWatch Agent with Prometheus Metrics Collection](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ContainerInsights-Prometheus-Setup.html)\. The default installation contains a Prometheus scrape configuration which pulls a useful subset of Envoy stats\. For more information, see [Prometheus Metrics for App Mesh](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ContainerInsights-Prometheus-metrics.html#ContainerInsights-Prometheus-metrics-appmesh)\.

To create an App Mesh custom CloudWatch dashboard configured to display the metrics that the agent is collecting, follow the steps in the [Viewing Your Prometheus Metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ContainerInsights-Prometheus-viewmetrics.html) tutorial\. Your graphs will begin to populate with the corresponding metrics as traffic enters the App Mesh application\.

### Filtering metrics for CloudWatch<a name="filtering-metrics"></a>

The App Mesh [metrics extension](https://docs.aws.amazon.com/app-mesh/latest/userguide/metrics.html#metrics-extension) provides a subset of useful metrics that give you insights into the behaviors of the resources you define in your mesh\. Since the CloudWatch agent supports scraping Prometheus metrics, you can provide a scrape configuration to select the metrics you want to pull from Envoy and send to CloudWatch\.

You can find an example of scraping metrics using Prometheus in our [Metrics Extension](https://github.com/aws/aws-app-mesh-examples/tree/main/walkthroughs/howto-metrics-extension-ecs) walkthrough\.

### CloudWatch Example<a name="cloudwatch-sample"></a>

You can find a sample configuration of CloudWatch in our [AWS Samples repository](https://github.com/aws-samples/aws-app-mesh-cloudwatch-agent)\.

### Walkthroughs for using CloudWatch<a name="cloudwatch-walkthrough"></a>
+ [Add monitoring and logging capabilities](https://www.appmeshworkshop.com/monitoring/) in our [App Mesh workshop](https://www.appmeshworkshop.com/introduction/)\.
+ [App Mesh with EKS—Observability: CloudWatch](https://github.com/aws/aws-app-mesh-examples/blob/main/walkthroughs/eks/o11y-cloudwatch.md)
+ [Using App Mesh's metrics extension on ECS](https://github.com/aws/aws-app-mesh-examples/tree/main/walkthroughs/howto-metrics-extension-ecs)

## Metrics extension for App Mesh<a name="metrics-extension"></a>

Envoy generates hundreds of metrics broken down into a few different dimensions\. The metrics aren't straightforward in the way they relate back to App Mesh\. In the case of virtual services, there is no mechanism to know for sure which virtual service is communicating to a given virtual node or virtual gateway\.

The App Mesh metrics extension enhances Envoy proxies running in your mesh\. This enhancement allows the proxies to emit additional metrics that are aware of the resources you define\. This small subset of additional metrics will help give you greater insight into the behavior of those resources you defined in App Mesh\.

To enable the App Mesh metrics extension, set the environment variable `APPMESH_METRIC_EXTENSION_VERSION` to `1`\.

```
APPMESH_METRIC_EXTENSION_VERSION=1
```

For more information about Envoy configuration variables, see [Envoy configuration variables](envoy-config.md)\.

### Metrics Related to Inbound Traffic<a name="inbound-metrics"></a>
+ `ActiveConnectionCount`
  + `envoy.appmesh.ActiveConnectionCount` — Number of active TCP connections\.
  + *Dimensions* — Mesh, VirtualNode, VirtualGateway
+ **`NewConnectionCount`**
  + `envoy.appmesh.NewConnectionCount` — Total number of TCP connections\.
  + *Dimensions* — Mesh, VirtualNode, VirtualGateway
+ **`ProcessedBytes`**
  + `envoy.appmesh.ProcessedBytes` — Total TCP bytes sent to and received from downstream clients\.
  + *Dimensions* — Mesh, VirtualNode, VirtualGateway
+ **`RequestCount`**
  + `envoy.appmesh.RequestCount` — The number of processed HTTP requests\.
  + *Dimensions* — Mesh, VirtualNode, VirtualGateway
+ **`GrpcRequestCount`**
  + `envoy.appmesh.GrpcRequestCount` — The number of processed gPRC requests\.
  + *Dimensions* — Mesh, VirtualNode, VirtualGateway

### Metrics Related to Outbound Traffic<a name="outbound-metrics"></a>

You will see different dimensions on your outbound metrics based on if they come from a virtual node or a virtual gateway\.
+ `TargetProcessedBytes`
  + `envoy.appmesh.TargetProcessedBytes` — Total TCP bytes sent to and received from targets upstream of Envoy\.
  + 

    *Dimensions*:
    + Virtual node dimensions — Mesh, VirtualNode, TargetVirtualService, TargetVirtualNode
    + Virtual gateway dimensions — Mesh, VirtualGateway, TargetVirtualService, TargetVirtualNode
+ **`HTTPCode_Target_2XX_Count`**
  + `envoy.appmesh.HTTPCode_Target_2XX_Count` — The number of HTTP requests to a target upstream of Envoy that resulted in a 2xx HTTP response\.
  + 

    *Dimensions*:
    + Virtual node dimensions — Mesh, VirtualNode, TargetVirtualService, TargetVirtualNode
    + Virtual gateway dimensions — Mesh, VirtualGateway, TargetVirtualService, TargetVirtualNode
+ **`HTTPCode_Target_3XX_Count`**
  + `envoy.appmesh.HTTPCode_Target_3XX_Count` — The number of HTTP requests to a target upstream of Envoy that resulted in a 3xx HTTP response\.
  + 

    *Dimensions*:
    + Virtual node dimensions — Mesh, VirtualNode, TargetVirtualService, TargetVirtualNode
    + Virtual gateway dimensions — Mesh, VirtualGateway, TargetVirtualService, TargetVirtualNode
+ **`HTTPCode_Target_4XX_Count`**
  + `envoy.appmesh.HTTPCode_Target_4XX_Count` — The number of HTTP requests to a target upstream of Envoy that resulted in a 4xx HTTP response\.
  + 

    *Dimensions*:
    + Virtual node dimensions — Mesh, VirtualNode, TargetVirtualService, TargetVirtualNode
    + Virtual gateway dimensions — Mesh, VirtualGateway, TargetVirtualService, TargetVirtualNode
+ **`HTTPCode_Target_5XX_Count`**
  + `envoy.appmesh.HTTPCode_Target_5XX_Count` — The number of HTTP requests to a target upstream of Envoy that resulted in a 5xx HTTP response\.
  + 

    *Dimensions*:
    + Virtual node dimensions — Mesh, VirtualNode, TargetVirtualService, TargetVirtualNode
    + Virtual gateway dimensions — Mesh, VirtualGateway, TargetVirtualService, TargetVirtualNode
+ **`RequestCountPerTarget`**
  + `envoy.appmesh.RequestCountPerTarget` — The number of requests sent to a target upstream of Envoy\.
  + 

    *Dimensions*:
    + Virtual node dimensions — Mesh, VirtualNode, TargetVirtualService, TargetVirtualNode
    + Virtual gateway dimensions — Mesh, VirtualGateway, TargetVirtualService, TargetVirtualNode
+ **`TargetResponseTime`**
  + `envoy.appmesh.TargetResponseTime` — The time elapsed from when a request is made to a target upstream of Envoy to when the full response is received\.
  + 

    *Dimensions*:
    + Virtual node dimensions — Mesh, VirtualNode, TargetVirtualService, TargetVirtualNode
    + Virtual gateway dimensions — Mesh, VirtualGateway, TargetVirtualService, TargetVirtualNode

## Datadog for App Mesh<a name="datadog"></a>

Datadog is a monitoring and security applicaton for end to end monitoring, metrics, and logging of cloud applications\. Datadog makes your infrastructure, applications, and third\-party applications completely observable\.

### Installing Datadog<a name="installing-datadog"></a>
+ EKS \- To setup Datadog with EKS, follow these steps from the [Datadog docs](https://docs.datadoghq.com/integrations/amazon_app_mesh/?tab=eks)\.
+ ECS EC2 \- To set up Datadog with ECS EC2, follow these steps from the [Datadog docs](https://docs.datadoghq.com/integrations/amazon_app_mesh/?tab=ecsec2)\.

### To learn more about Datadog<a name="datadog-learn-more"></a>
+ [Datadog Documentation](https://docs.datadoghq.com/)
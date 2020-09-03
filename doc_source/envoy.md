# Envoy image<a name="envoy"></a>

AWS App Mesh is a service mesh based on the [Envoy](https://www.envoyproxy.io/) proxy\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/app-mesh/latest/userguide/images/proxy.png)

You must add an Envoy proxy to the Amazon ECS task, Kubernetes pod, or Amazon EC2 instance represented by your App Mesh virtual nodes\. App Mesh vends an Envoy proxy Docker container image and ensures that this container image is patched with the latest vulnerability and performance patches\. App Mesh tests a new Envoy proxy release against the App Mesh feature set before making a new container image available to you\.
+ All [supported](https://docs.aws.amazon.com/general/latest/gr/appmesh.html) Regions other than `me-south-1` and `ap-east-1`\. You can replace *region\-code* with any Region other than `me-south-1` and `ap-east-1`\. 

  ```
  840364872350.dkr.ecr.region-code.amazonaws.com/aws-appmesh-envoy:v1.15.0.0-prod
  ```
+ `me-south-1` Region:

  ```
  772975370895.dkr.ecr.me-south-1.amazonaws.com/aws-appmesh-envoy:v1.15.0.0-prod
  ```
+ `ap-east-1` Region:

  ```
  856666278305.dkr.ecr.ap-east-1.amazonaws.com/aws-appmesh-envoy:v1.15.0.0-prod
  ```

Access to this container image in Amazon ECR is controlled by AWS Identity and Access Management, so you must use IAM to ensure that you have read access to Amazon ECR\. For example, when using Amazon ECS, you can assign an appropriate task execution role to an Amazon ECS task\. Further, if you use IAM policies that limit access to specific Amazon ECR resources, then you must ensure that you allow access to the Region\-specific Amazon Resource Name \(ARN\) that identifies the `aws-appmesh-envoy` repository\. For example, in the `us-west-2` region, you'd allow access to the following resource: `arn:aws:ecr:us-west-2:840364872350:repository/aws-appmesh-envoy`\. For more information, see [Amazon ECR Managed Policies](https://docs.aws.amazon.com/AmazonECR/latest/userguide/ecr_managed_policies.html)\. If you're using Docker on an Amazon EC2 instance, then you need to authenticate Docker to the repository\. For more information, see [Registry authentication](https://docs.aws.amazon.com/AmazonECR/latest/userguide/Registries.html#registry_auth)\.

We occasionally release new App Mesh features that depend on Envoy changes that have not been merged to the upstream Envoy images yet\. To use these new App Mesh features before the Envoy changes are merged upstream, you must use the App Mesh\-vended Envoy container image\. For a list of changes, see the [App Mesh GitHub roadmap issues](https://github.com/aws/aws-app-mesh-roadmap/labels/Envoy%20Upstream) with the `Envoy Upstream` label\. Otherwise, while we recommend that you use the App Mesh Envoy container image as the best supported option, you may use your own Envoy image\.

## Envoy configuration variables<a name="envoy-config"></a>

The following environment variables enable you to configure the Envoy containers for your App Mesh virtual node task groups\.

### Required variables<a name="envoy-required-config"></a>

The following environment variable is required for all App Mesh Envoy containers\.

`APPMESH_VIRTUAL_NODE_NAME`  
When you add the Envoy container to a task group, set this environment variable to the name of the virtual node that the task group represents: for example, `mesh/meshName/virtualNode/virtualNodeName`\.

### Optional variables<a name="envoy-optional-config"></a>

The following environment variable is optional for App Mesh Envoy containers\.

`ENVOY_LOG_LEVEL`  
Specifies the log level for the Envoy container\.  
Valid values: `trace`, `debug`, `info`, `warning`, `error`, `critical`, `off`  
Default: `info`

### Tracing variables<a name="tracing-variables"></a>

You can configure none, or one of the following tracing drivers\.

#### AWS X\-Ray variables<a name="envoy-xray-config"></a>

The following environment variables help you to configure App Mesh with AWS X\-Ray\. For more information, see the [AWS X\-Ray Developer Guide](https://docs.aws.amazon.com/xray/latest/devguide/)\.

`ENABLE_ENVOY_XRAY_TRACING`  
Enables X\-Ray tracing using `127.0.0.1:2000` as the default daemon endpoint\. To enable, set the value to `1` \(default value is `0`\)\.

`XRAY_DAEMON_PORT`  
Specify a port value to override the default X\-Ray daemon port: `2000`\.

#### Datadog tracing variables<a name="datadog-tracing"></a>

The following environment variables help you configure App Mesh with the Datadog agent tracer\. For more information, see [Agent Configuration](https://docs.datadoghq.com/tracing/send_traces/) in the Datadog documentation\.

`ENABLE_ENVOY_DATADOG_TRACING`  
Enables Datadog trace collection using `127.0.0.1:8126` as the default Datadog agent endpoint\. To enable, set the value to `1` \(default value is `0`\)\.

`DATADOG_TRACER_PORT`  
Specify a port value to override the default Datadog agent port: `8126`\.

`DATADOG_TRACER_ADDRESS`  
Specify an IP address to override the default Datadog agent address: `127.0.0.1`\.

#### Envoy tracing variable<a name="envoy-tracing"></a>

The following environment variable enables you to use your own tracing configuration\. 

`ENVOY_TRACING_CFG_FILE`  
Specify a file path in the Envoy container file system\. For more information, see [https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/trace/v2/http_tracer.proto](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/trace/v2/http_tracer.proto) in the Envoy documentation\.

### DogStatsD variables<a name="envoy-dogstatsd-config"></a>

The following environment variables help you to configure App Mesh with DogStatsD\. For more information, see the [DogStatsD](https://docs.datadoghq.com/developers/dogstatsd/) documentation\.

`ENABLE_ENVOY_DOG_STATSD`  
Enables DogStatsD stats using `127.0.0.1:8125` as the default daemon endpoint\. To enable, set the value to `1`\.

`STATSD_PORT`  
Specify a port value to override the default DogStatsD daemon port\.

`ENVOY_STATS_SINKS_CFG_FILE`  
Specify a file path in the Envoy container file system to override the default DogStatsD configuration with your own\. For more information, see [config\.metrics\.v2\.DogStatsdSink](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/metrics/v2/stats.proto#envoy-api-msg-config-metrics-v2-dogstatsdsink) in the Envoy documentation\.

### Envoy stats variables<a name="envoy-stats-config"></a>

The following environment variables help you to configure App Mesh with Envoy Stats\. For more information, see the [Envoy Stats](https://www.envoyproxy.io/docs/envoy/v1.6.0/api-v2/config/metrics/v2/stats.proto) documentation\.

`ENABLE_ENVOY_STATS_TAGS`  
Enables the use of App Mesh defined tags `appmesh.mesh` and `appmesh.virtual_node`\. For more information, see [config\.metrics\.v2\.TagSpecifier](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/metrics/v2/stats.proto#envoy-api-msg-config-metrics-v2-tagspecifier) in the Envoy documentation\. To enable, set the value to `1`\.

`ENVOY_STATS_CONFIG_FILE`  
Specify a file path in the Envoy container file system to override the default Stats tags configuration file with your own\.

## Default route retry policy<a name="default-retry-policy"></a>

If you had no meshes in your account before July 29, 2020, then App Mesh automatically creates a default Envoy route retry policy for all HTTP, HTTP/2, and gRPC requests in any mesh in your account on or after July 29, 2020\. If you had any meshes in your account before July 29, 2020, then no default policy is created for any Envoy routes that existed before, on, or after July 29, 2020, unless you [open a ticket with AWS support](https://console.aws.amazon.com/support/home#/case/create)\. Once support processes the ticket, then the default policy will be created for any future Envoy routes that App Mesh creates on or after the date that the ticket was processed\. For more information about Envoy route retry policies, see [config\.route\.v3\.RetryPolicy](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/route_components.proto#envoy-v3-api-msg-config-route-v3-retrypolicy) in the Envoy documentation\.

App Mesh creates an Envoy route when you either create an App Mesh [route](routes.md) or define a virtual node provider for an App Mesh [virtual service](virtual_services.md)\. Though you can create an App Mesh route retry policy, you can't create an App Mesh retry policy for a virtual node provider\.

The default policy is not visible through the App Mesh API\. The default policy is only visible through Envoy\. To view the configuration, [enable the administration interface](troubleshooting-best-practices.md#ts-bp-enable-proxy-admin-interface) and send a request to Envoy for a `config_dump`\. The default policy includes the following settings:
+ **Max retries** – `2`
+ **gRPC retry events** – `UNAVAILABLE`
+ **HTTP retry events** – `503`
**Note**  
It's not possible to create an App Mesh route retry policy that looks for a specific HTTP error code, however an App Mesh route retry policy can look for `server-error` or `gateway-error`, which both include `503` errors\. For more information, see [Routes](routes.md)\.
+ **TCP retry event** – `connect-failure` and `refused-stream`
**Note**  
It's not possible to create an App Mesh route retry policy that looks for either of these events, however an App Mesh route retry policy can look for `connection-error`, which is equivalent to `connect-failure`\. For more information, see [Routes](routes.md)\.

## Default circuit breaker<a name="default-ciruit-breaker"></a>

When you deploy an Envoy in App Mesh, Envoy default values are set for some of the circuit breaker settings\. For more information, see [cluster\.CircuitBreakers\.Thresholds](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/cluster/circuit_breaker.proto#cluster-circuitbreakers-thresholds) in the Envoy documentation\. The settings are not visible through the App Mesh API\. The settings are only visible through Envoy\. To view the configuration, [enable the administration interface](troubleshooting-best-practices.md#ts-bp-enable-proxy-admin-interface) and send a request to Envoy for a `config_dump`\.

If you had no meshes in your account before July 29, 2020, then for each Envoy that you deploy in a mesh created on or after July 29, 2020, App Mesh effectively disables circuit breakers by changing the Envoy default values for the settings that follow\. If you had any meshes in your account before July 29, 2020, then the Envoy default values are set for any Envoy that you deploy in App Mesh on, or after July 29, 2020, unless you [open a ticket with AWS support](https://console.aws.amazon.com/support/home#/case/create)\. Once support processes the ticket, then the App Mesh default values for the following Envoy settings are set by App Mesh on all Envoys that you deploy after the date that the ticket is processed:
+ `max_requests` – `2147483647`
+ `max_pending_requests` – `2147483647`
+ `max_connections` – `2147483647`
+ `max_retries` – `2147483647`

**Note**  
Whether your Envoys have the Envoy or App Mesh default circuit breaker values, you are not able to modify the values\.
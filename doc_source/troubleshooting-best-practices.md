# App Mesh troubleshooting best practices<a name="troubleshooting-best-practices"></a>

We recommend that you follow the best practices in this topic to troubleshoot issues when using App Mesh\.

## Enable the Envoy proxy administration interface<a name="ts-bp-enable-proxy-admin-interface"></a>

The Envoy proxy ships with an administration interface that you can use to discover configuration and statistics and to perform other administrative functions such as connection draining\. For more information, see [Administration interface](https://www.envoyproxy.io/docs/envoy/latest/operations/admin) in the Envoy documentation\.

If you use the managed [ Envoy imageEnvoy  AWS App Mesh is a service mesh based on the [Envoy](https://www.envoyproxy.io/) proxy\. 

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/app-mesh/latest/userguide/images/proxy.png) You must add an Envoy proxy to the Amazon ECS task, Kubernetes pod, or Amazon EC2 instance represented by your App Mesh endpoint, such as a virtual node or virtual gateway\. App Mesh vends an Envoy proxy Docker container image and ensures that this container image is patched with the latest vulnerability and performance patches\. App Mesh tests a new Envoy proxy release against the App Mesh feature set before making a new container image available to you\. You can choose either a Regional image from the list below or an image from our [public repository](https://gallery.ecr.aws/appmesh/aws-appmesh-envoy) named `aws-appmesh-envoy`\.   All [supported](https://docs.aws.amazon.com/general/latest/gr/appmesh.html) Regions other than `me-south-1` and `ap-east-1`, `eu-south-1`, `af-south-1`\. You can replace *Region\-code* with any Region other than `me-south-1` and `ap-east-1`, `eu-south-1`, `af-south-1`\.  

  ```
  840364872350.dkr.ecr.region-code.amazonaws.com/aws-appmesh-envoy:v1.16.1.1-prod
  ```   `me-south-1` Region: 

  ```
  772975370895.dkr.ecr.me-south-1.amazonaws.com/aws-appmesh-envoy:v1.16.1.1-prod
  ```   `ap-east-1` Region: 

  ```
  856666278305.dkr.ecr.ap-east-1.amazonaws.com/aws-appmesh-envoy:v1.16.1.1-prod
  ```   `eu-south-1` Region: 

  ```
  422531588944.dkr.ecr.eu-south-1.amazonaws.com/aws-appmesh-envoy:v1.16.1.1-prod
  ```   `af-south-1` Region: 

  ```
  924023996002.dkr.ecr.af-south-1.amazonaws.com/aws-appmesh-envoy:v1.16.1.1-prod
  ```   `Public repository` 

  ```
  public.ecr.aws/appmesh/aws-appmesh-envoy:v1.16.1.0-prod
  ```   Only version v1\.9\.0\.0\-prod or later is supported for use with App Mesh\. Access to this container image in Amazon ECR is controlled by AWS Identity and Access Management, so you must use IAM to ensure that you have read access to Amazon ECR\. For example, when using Amazon ECS, you can assign an appropriate task execution role to an Amazon ECS task\. Further, if you use IAM policies that limit access to specific Amazon ECR resources, then you must ensure that you allow access to the Region\-specific Amazon Resource Name \(ARN\) that identifies the `aws-appmesh-envoy` repository\. For example, in the `us-west-2` Region, you'd allow access to the following resource: `arn:aws:ecr:us-west-2:840364872350:repository/aws-appmesh-envoy`\. For more information, see [Amazon ECR Managed Policies](https://docs.aws.amazon.com/AmazonECR/latest/userguide/ecr_managed_policies.html)\. If you're using Docker on an Amazon EC2 instance, then you need to authenticate Docker to the repository\. For more information, see [Registry authentication](https://docs.aws.amazon.com/AmazonECR/latest/userguide/Registries.html#registry_auth)\. We occasionally release new App Mesh features that depend on Envoy changes that have not been merged to the upstream Envoy images yet\. To use these new App Mesh features before the Envoy changes are merged upstream, you must use the App Mesh\-vended Envoy container image\. For a list of changes, see the [App Mesh GitHub roadmap issues](https://github.com/aws/aws-app-mesh-roadmap/labels/Envoy%20Upstream) with the `Envoy Upstream` label\. Otherwise, while we recommend that you use the App Mesh Envoy container image as the best supported option, you may use your own Envoy image\.  Envoy configuration variables  The following environment variables enable you to configure the Envoy containers for your App Mesh virtual node task groups\.  Required variables  The following environment variable is required for all App Mesh Envoy containers\. This variable can only be used with version `1.15.0` or later of the Envoy image\. If you're using an earlier version of the image, then you must set the `APPMESH_VIRTUAL_NODE_NAME` variable instead\.  

`APPMESH_RESOURCE_ARN`  
 When you add the Envoy container to a task group, set this environment variable to the ARN of the virtual node or the virtual gateway that the task group represents\. The following list contains example ARNs:  
+ **Virtual node** – arn:aws:appmesh:*Region\-code*:*111122223333*:mesh/*meshName*/virtualNode/*virtualNodeName*
+ **Virtual gateway** – arn:aws:appmesh:*Region\-code*:*111122223333*:mesh/*meshName*/virtualGateway/*virtualGatewayName*
When using the [App Mesh Preview Channel](preview.md), ARNs must use the *us\-west\-2* Region and use `appmesh-preview`, instead of `appmesh`\. For example, the ARN of a virtual node in the App Mesh Preview Channel is `arn:aws:appmesh-preview:us-west-2:111122223333:mesh/meshName/virtualNode/virtualNodeName`\.    Optional variables  The following environment variable is optional for App Mesh Envoy containers\.  

`ENVOY_LOG_LEVEL`  
Specifies the log level for the Envoy container\.  
Valid values: `trace`, `debug`, `info`, `warning`, `error`, `critical`, `off`  
Default: `info` 

`ENVOY_INITIAL_FETCH_TIMEOUT`  
Specifies the amount of time an Envoy will wait for first configuration response from the management server during the initialization process\.  
For more information, see [Configuration sources](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/core/v3/config_source.proto#envoy-v3-api-field-config-core-v3-configsource-initial-fetch-timeout) in Envoy Documentation\. 0 indicates there is no timeout\.  
Default: `0`   Admin variables  These environment variables let you configure Envoy's administrative interface\.  

`ENVOY_ADMIN_ACCESS_PORT`  
Specify a custom admin port for Envoy to listen on\. Default: `9901`\. 

`ENVOY_ADMIN_ACCESS_LOG_FILE`  
Specify a custom path to write Envoy access logs to\. Default: `/tmp/envoy_admin_access.log`\.    Tracing variables  You can configure none, or one of the following tracing drivers\.  AWS X\-Ray variables  The following environment variables help you to configure App Mesh with AWS X\-Ray\. For more information, see the [AWS X\-Ray Developer Guide](https://docs.aws.amazon.com/xray/latest/devguide/)\.  

`ENABLE_ENVOY_XRAY_TRACING`  
Enables X\-Ray tracing using `127.0.0.1:2000` as the default daemon endpoint\. To enable, set the value to `1` \(default value is `0`\)\. 

`XRAY_DAEMON_PORT`  
Specify a port value to override the default X\-Ray daemon port: `2000`\.    Datadog tracing variables  The following environment variables help you configure App Mesh with the Datadog agent tracer\. For more information, see [Agent Configuration](https://docs.datadoghq.com/tracing/send_traces/) in the Datadog documentation\.  

`ENABLE_ENVOY_DATADOG_TRACING`  
Enables Datadog trace collection using `127.0.0.1:8126` as the default Datadog agent endpoint\. To enable, set the value to `1` \(default value is `0`\)\. 

`DATADOG_TRACER_PORT`  
Specify a port value to override the default Datadog agent port: `8126`\. 

`DATADOG_TRACER_ADDRESS`  
Specify an IP address to override the default Datadog agent address: `127.0.0.1`\.    Jaeger tracing variables  The following environment variables help you configure App Mesh with Jaeger tracing\. For more information, see [Getting Started](https://www.jaegertracing.io/docs/1.21/getting-started/) in the Jaeger documentation\. These variables are supported with `v1.16.1.0-prod` or later version of the Envoy image\.  

`ENABLE_ENVOY_JAEGER_TRACING`  
Enables Jaeger trace collection using `127.0.0.1:9411` as the default Jaeger endpoint\. To enable, set the value to `1` \(default value is `0`\)\. 

`JAEGER_TRACER_PORT`  
Specify a port value to override the default Jaeger port: `9411`\. 

`JAEGER_TRACER_ADDRESS`  
Specify an IP address to override the default Jaeger address: `127.0.0.1`\.    Envoy tracing variable  The following environment variable enables you to use your own tracing configuration\.   

`ENVOY_TRACING_CFG_FILE`  
Specify a file path in the Envoy container file system\. For more information, see [https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/trace/v2/http_tracer.proto](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/trace/v2/http_tracer.proto) in the Envoy documentation\.     DogStatsD variables  The following environment variables help you to configure App Mesh with DogStatsD\. For more information, see the [DogStatsD](https://docs.datadoghq.com/developers/dogstatsd/) documentation\.  

`ENABLE_ENVOY_DOG_STATSD`  
Enables DogStatsD stats using `127.0.0.1:8125` as the default daemon endpoint\. To enable, set the value to `1`\. 

`STATSD_PORT`  
Specify a port value to override the default DogStatsD daemon port\. 

`STATSD_ADDRESS`  
Specify an IP address value to override the default DogStatsD daemon IP address\. Default: `127.0.0.1`\. This variable can only be used with version `1.15.0` or later of the Envoy image\. 

`ENVOY_STATS_SINKS_CFG_FILE`  
Specify a file path in the Envoy container file system to override the default DogStatsD configuration with your own\. For more information, see [config\.metrics\.v2\.DogStatsdSink](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/metrics/v2/stats.proto#envoy-api-msg-config-metrics-v2-dogstatsdsink) in the Envoy documentation\.    App Mesh variables  The following variables help you configure App Mesh\.  

`APPMESH_PREVIEW`  
Set the value to `1` to connect to the App Mesh Preview Channel endpoint\. For more information about using the App Mesh Preview Channel, see [App Mesh Preview Channel](preview.md)\. 

`APPMESH_RESOURCE_CLUSTER`  
By default App Mesh uses the name of the resource you specified in `APPMESH_RESOURCE_ARN` when Envoy is referring to itself in metrics and traces\. You can override this behavior by setting the `APPMESH_RESOURCE_CLUSTER` environment variable with your own name\. This variable can only be used with version `1.15.0` or later of the Envoy image\.    Envoy stats variables  The following environment variables help you to configure App Mesh with Envoy Stats\. For more information, see the [Envoy Stats](https://www.envoyproxy.io/docs/envoy/v1.6.0/api-v2/config/metrics/v2/stats.proto) documentation\.  

`ENABLE_ENVOY_STATS_TAGS`  
Enables the use of App Mesh defined tags `appmesh.mesh` and `appmesh.virtual_node`\. For more information, see [config\.metrics\.v2\.TagSpecifier](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/metrics/v2/stats.proto#envoy-api-msg-config-metrics-v2-tagspecifier) in the Envoy documentation\. To enable, set the value to `1`\. 

`ENVOY_STATS_CONFIG_FILE`  
Specify a file path in the Envoy container file system to override the default Stats tags configuration file with your own\.    Deprecated variables  The environment variables `APPMESH_VIRTUAL_NODE_NAME` and `APPMESH_RESOURCE_NAME` are deprecated in Envoy version `1.15.0` or later, but are still supported for existing meshes\. Instead of using these variables with Envoy version `1.15.0` or later, use `APPMESH_RESOURCE_ARN` for all App Mesh endpoints\.     Envoy defaults set by App Mesh  The following sections provide information on the Envoy defaults for the route retry policy and circuit breaker that are set by App Mesh\.  Default route retry policy  If you had no meshes in your account before July 29, 2020, then App Mesh automatically creates a default Envoy route retry policy for all HTTP, HTTP/2, and gRPC requests in any mesh in your account on or after July 29, 2020\. If you had any meshes in your account before July 29, 2020, then no default policy is created for any Envoy routes that existed before, on, or after July 29, 2020, unless you [open a ticket with AWS support](https://console.aws.amazon.com/support/home#/case/create)\. Once support processes the ticket, then the default policy will be created for any future Envoy routes that App Mesh creates on or after the date that the ticket was processed\. For more information about Envoy route retry policies, see [config\.route\.v3\.RetryPolicy](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/route_components.proto#envoy-v3-api-msg-config-route-v3-retrypolicy) in the Envoy documentation\. App Mesh creates an Envoy route when you either create an App Mesh [route](routes.md) or define a virtual node provider for an App Mesh [virtual service](virtual_services.md)\. Though you can create an App Mesh route retry policy, you can't create an App Mesh retry policy for a virtual node provider\. The default policy is not visible through the App Mesh API\. The default policy is only visible through Envoy\. To view the configuration, [enable the administration interface](troubleshooting-best-practices.md#ts-bp-enable-proxy-admin-interface) and send a request to Envoy for a `config_dump`\. The default policy includes the following settings:   **Max retries** – `2`   **gRPC retry events** – `UNAVAILABLE`   **HTTP retry events** – `503`  It's not possible to create an App Mesh route retry policy that looks for a specific HTTP error code, however an App Mesh route retry policy can look for `server-error` or `gateway-error`, which both include `503` errors\. For more information, see [Routes](routes.md)\.    **TCP retry event** – `connect-failure` and `refused-stream`  It's not possible to create an App Mesh route retry policy that looks for either of these events, however an App Mesh route retry policy can look for `connection-error`, which is equivalent to `connect-failure`\. For more information, see [Routes](routes.md)\.    **Reset** – Envoy will attempt a retry if the upstream server does not respond at all \(disconnect/reset/read timeout\)\.     Default circuit breaker  When you deploy an Envoy in App Mesh, Envoy default values are set for some of the circuit breaker settings\. For more information, see [cluster\.CircuitBreakers\.Thresholds](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/cluster/circuit_breaker.proto#cluster-circuitbreakers-thresholds) in the Envoy documentation\. The settings are not visible through the App Mesh API\. The settings are only visible through Envoy\. To view the configuration, [enable the administration interface](troubleshooting-best-practices.md#ts-bp-enable-proxy-admin-interface) and send a request to Envoy for a `config_dump`\. If you had no meshes in your account before July 29, 2020, then for each Envoy that you deploy in a mesh created on or after July 29, 2020, App Mesh effectively disables circuit breakers by changing the Envoy default values for the settings that follow\. If you had any meshes in your account before July 29, 2020, then the Envoy default values are set for any Envoy that you deploy in App Mesh on, or after July 29, 2020, unless you [open a ticket with AWS support](https://console.aws.amazon.com/support/home#/case/create)\. Once support processes the ticket, then the App Mesh default values for the following Envoy settings are set by App Mesh on all Envoys that you deploy after the date that the ticket is processed:   `max_requests` – `2147483647`   `max_pending_requests` – `2147483647`   `max_connections` – `2147483647`   `max_retries` – `2147483647`    Whether your Envoys have the Envoy or App Mesh default circuit breaker values, you are not able to modify the values\.     ](envoy.md), the administration endpoint is enabled by default on port 9901\. Examples provided in [App Mesh troubleshooting best practices](#troubleshooting-best-practices) display the example administration endpoint URL as `http://my-app.default.svc.cluster.local:9901/`\. 

**Note**  
The administration endpoint should never be exposed to the public internet\. Additionally, we recommend monitoring the administration endpoint logs, which are set by the `ENVOY_ADMIN_ACCESS_LOG_FILE` environment variable to `/tmp/envoy_admin_access.log` by default\. 

## Enable Envoy DogStatsD integration for metric offload<a name="ts-bp-enable-envoy-statsd-integration"></a>

The Envoy proxy can be configured to offload statistics for OSI Layer 4 and Layer 7 traffic and for internal process health\. While this topic shows how to use these statistics without offloading the metrics to sinks like CloudWatch metrics and Prometheus\., having these statistics in a centralized location for all of your applications can help you diagnose issues and confirm behavior more quickly\. For more information, see [Using Amazon CloudWatch Metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/working_with_metrics.html) and the [Prometheus documentation](https://prometheus.io/docs/introduction/overview/)\. 

You can configure DogStatsD metrics by setting the parameters defined in [DogStatsD variables](envoy-config.md#envoy-dogstatsd-config)\. For more information about DogStatsD, see the [DogStatsD](https://docs.datadoghq.com/developers/dogstatsd/?tab=hostagent) documentation\. You can find a demonstration of metric offload to AWS CloudWatch metrics in the [App Mesh with Amazon ECS basics walk\-through](https://github.com/aws/aws-app-mesh-examples/tree/master/walkthroughs/howto-ecs-basics) on GitHub\.

## Enable access logs<a name="ts-bp-enable-access-logs"></a>

We recommend enabling access logs on your [Virtual nodes](virtual_nodes.md) and [Virtual gateways](virtual_gateways.md) to discover details about traffic transiting between your applications\. For more information, see [Access logging](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/observability/access_logging) in the Envoy documentation\. The logs provide detailed information on OSI Layer 4 and Layer 7 traffic behavior\. When you use Envoy’s default format, you can analyze the access logs with [CloudWatch Logs Insights](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/AnalyzingLogData.html) using the following parse statement\.

```
parse @message "[*] \"* * *\" * * * * * * * * * * *" as StartTime, Method, Path, Protocol, ResponseCode, ResponseFlags, BytesReceived, BytesSent, DurationMillis, UpstreamServiceTimeMillis, ForwardedFor, UserAgent, RequestId, Authority, UpstreamHost
```

## Enable Envoy debug logging in pre\-production environments<a name="ts-bp-enable-envoy-debug-logging"></a>

We recommend setting the Envoy proxy’s log level to `debug` in a pre\-production environment\. Debug logs can help you identify issues before you graduate the associated App Mesh configuration to your production environment\. 

If you’re using the [Envoy image](envoy.md), you can set the log level to `debug` through the `ENVOY_LOG_LEVEL` environment variable\. 

**Note**  
We do not recommend using the `debug` level in production environments\. Setting the level to `debug` increases the logging and may affect performance and the overall cost of logs offloaded to solutions like [CloudWatch Logs](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html)\. 

When you use Envoy’s default format, you can analyze the process logs with [CloudWatch Logs Insights](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/AnalyzingLogData.html) using the following parse statement: 

```
parse @message "[*][*][*][*] [*] *" as Time, Thread, Level, Name, Source, Message
```
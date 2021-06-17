# Envoy configuration variables<a name="envoy-config"></a>

The following environment variables enable you to configure the Envoy containers for your App Mesh virtual node task groups\.

**Note**  
Envoy 1\.17 will no longer support Envoy’s **v2 xDS** API\. If you are using [Envoy configuration variables](https://docs.aws.amazon.com/app-mesh/latest/userguide/envoy-config.html) that accept Envoy config files, they must be updated to the latest** v3 xDS** API\.

## Required variables<a name="envoy-required-config"></a>

The following environment variable is required for all App Mesh Envoy containers\. This variable can only be used with version `1.15.0` or later of the Envoy image\. If you're using an earlier version of the image, then you must set the `APPMESH_VIRTUAL_NODE_NAME` variable instead\.

`APPMESH_RESOURCE_ARN`  
 When you add the Envoy container to a task group, set this environment variable to the ARN of the virtual node or the virtual gateway that the task group represents\. The following list contains example ARNs:  
+ **Virtual node** – arn:aws:appmesh:*Region\-code*:*111122223333*:mesh/*meshName*/virtualNode/*virtualNodeName*
+ **Virtual gateway** – arn:aws:appmesh:*Region\-code*:*111122223333*:mesh/*meshName*/virtualGateway/*virtualGatewayName*
When using the [App Mesh Preview Channel](preview.md), ARNs must use the *us\-west\-2* Region and use `appmesh-preview`, instead of `appmesh`\. For example, the ARN of a virtual node in the App Mesh Preview Channel is `arn:aws:appmesh-preview:us-west-2:111122223333:mesh/meshName/virtualNode/virtualNodeName`\.

## Optional variables<a name="envoy-optional-config"></a>

The following environment variable is optional for App Mesh Envoy containers\.

`ENVOY_LOG_LEVEL`  
Specifies the log level for the Envoy container\.  
Valid values: `trace`, `debug`, `info`, `warning`, `error`, `critical`, `off`  
Default: `info`

`ENVOY_INITIAL_FETCH_TIMEOUT`  
Specifies the amount of time an Envoy will wait for first configuration response from the management server during the initialization process\.  
For more information, see [Configuration sources](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/core/v3/config_source.proto#envoy-v3-api-field-config-core-v3-configsource-initial-fetch-timeout) in Envoy Documentation\. 0 indicates there is no timeout\.  
Default: `0`

### Admin variables<a name="envoy-admin-variables"></a>

These environment variables let you configure Envoy's administrative interface\.

`ENVOY_ADMIN_ACCESS_PORT`  
Specify a custom admin port for Envoy to listen on\. Default: `9901`\.

`ENVOY_ADMIN_ACCESS_LOG_FILE`  
Specify a custom path to write Envoy access logs to\. Default: `/tmp/envoy_admin_access.log`\.

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

#### Jaeger tracing variables<a name="jaeger-tracing"></a>

The following environment variables help you configure App Mesh with Jaeger tracing\. For more information, see [Getting Started](https://www.jaegertracing.io/docs/1.21/getting-started/) in the Jaeger documentation\. These variables are supported with `v1.16.1.0-prod` or later version of the Envoy image\.

`ENABLE_ENVOY_JAEGER_TRACING`  
Enables Jaeger trace collection using `127.0.0.1:9411` as the default Jaeger endpoint\. To enable, set the value to `1` \(default value is `0`\)\.

`JAEGER_TRACER_PORT`  
Specify a port value to override the default Jaeger port: `9411`\.

`JAEGER_TRACER_ADDRESS`  
Specify an IP address to override the default Jaeger address: `127.0.0.1`\.

#### Envoy tracing variable<a name="envoy-tracing"></a>

The following environment variable enables you to use your own tracing configuration\. 

`ENVOY_TRACING_CFG_FILE`  
Specify a file path in the Envoy container file system\. For more information, see [https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/trace/v3/http_tracer.proto#envoy-v3-api-msg-config-trace-v3-tracing](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/trace/v3/http_tracer.proto#envoy-v3-api-msg-config-trace-v3-tracing) in the Envoy documentation\.

### DogStatsD variables<a name="envoy-dogstatsd-config"></a>

The following environment variables help you to configure App Mesh with DogStatsD\. For more information, see the [DogStatsD](https://docs.datadoghq.com/developers/dogstatsd/) documentation\.

`ENABLE_ENVOY_DOG_STATSD`  
Enables DogStatsD stats using `127.0.0.1:8125` as the default daemon endpoint\. To enable, set the value to `1`\.

`STATSD_PORT`  
Specify a port value to override the default DogStatsD daemon port\.

`STATSD_ADDRESS`  
Specify an IP address value to override the default DogStatsD daemon IP address\. Default: `127.0.0.1`\. This variable can only be used with version `1.15.0` or later of the Envoy image\.

### App Mesh variables<a name="envoy-appmesh-variables"></a>

The following variables help you configure App Mesh\.

`APPMESH_PREVIEW`  
Set the value to `1` to connect to the App Mesh Preview Channel endpoint\. For more information about using the App Mesh Preview Channel, see [App Mesh Preview Channel](preview.md)\.

`APPMESH_RESOURCE_CLUSTER`  
By default App Mesh uses the name of the resource you specified in `APPMESH_RESOURCE_ARN` when Envoy is referring to itself in metrics and traces\. You can override this behavior by setting the `APPMESH_RESOURCE_CLUSTER` environment variable with your own name\. This variable can only be used with version `1.15.0` or later of the Envoy image\.

### Envoy stats variables<a name="envoy-stats-config"></a>

The following environment variables help you to configure App Mesh with Envoy Stats\. For more information, see the [Envoy Stats](https://www.envoyproxy.io/docs/envoy/v1.6.0/api-v2/config/metrics/v2/stats.proto) documentation\.

`ENABLE_ENVOY_STATS_TAGS`  
Enables the use of App Mesh defined tags `appmesh.mesh` and `appmesh.virtual_node`\. For more information, see [config\.metrics\.v3\.TagSpecifier](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/metrics/v3/stats.proto#config-metrics-v3-tagspecifier) in the Envoy documentation\. To enable, set the value to `1`\.

`ENVOY_STATS_CONFIG_FILE`  
Specify a file path in the Envoy container file system to override the default Stats tags configuration file with your own\. For more information, see [config\.metrics\.v3\.StatsConfig](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/metrics/v3/stats.proto#config-metrics-v3-statsconfig)\.

`ENVOY_STATS_SINKS_CFG_FILE`  
Specify a file path in the Envoy container file system to override the default configuration with your own\. For more information, see [config\.metrics\.v3\.StatsSink](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/metrics/v3/stats.proto#config-metrics-v3-statssink) in the Envoy documentation\.

### Deprecated variables<a name="envoy-deprecated-variables"></a>

The environment variables `APPMESH_VIRTUAL_NODE_NAME` and `APPMESH_RESOURCE_NAME` are no longer supported in Envoy version `1.15.0` or later, but are still supported for existing meshes\. Instead of using these variables with Envoy version `1.15.0` or later, use `APPMESH_RESOURCE_ARN` for all App Mesh endpoints\.
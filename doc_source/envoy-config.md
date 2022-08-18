# Envoy configuration variables<a name="envoy-config"></a>

Use the following environment variables to configure the Envoy containers for your App Mesh virtual node task groups\.

**Note**  
App Mesh Envoy 1\.17 doesn't supports Envoy’s **v2 xDS** API\. If you're using [Envoy configuration variables](https://docs.aws.amazon.com/app-mesh/latest/userguide/envoy-config.html) that accept Envoy config files, they must be updated to the latest** v3 xDS** API\.

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
Specifies the amount of time Envoy waits for the first configuration response from the management server during the initialization process\.  
For more information, see [Configuration sources](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/core/v3/config_source.proto#envoy-v3-api-field-config-core-v3-configsource-initial-fetch-timeout) in Envoy Documentation\. When set to `0`, there is no timeout\.  
Default: `0`

### Admin variables<a name="envoy-admin-variables"></a>

Use these environment variables to configure Envoy's administrative interface\.

`ENVOY_ADMIN_ACCESS_PORT`  
Specify a custom admin port for Envoy to listen on\. Default: `9901`\.

`ENVOY_ADMIN_ACCESS_LOG_FILE`  
Specify a custom path to write Envoy access logs to\. Default: `/tmp/envoy_admin_access.log`\.

`ENVOY_ADMIN_ACCESS_ENABLE_IPV6`  
Toggles Envoy’s administration interface to accept `IPv6` traffic, which allows this interface to accept both `IPv4` and `IPv6` traffic\. By default this flag is set to false, and Envoy only listens to `IPv4` traffic\. This variable can only be used with Envoy image version 1\.22\.0 or later\.

### Agent variables<a name="agent-variables"></a>

Use these environment variables to configure the AWS App Mesh Agent for Envoy\. For more information, see App Mesh [Agent for Envoy](https://docs.aws.amazon.com/app-mesh/latest/userguide/appnet-agent.html)\.

`APPNET_ENVOY_RESTART_COUNT`  
Specifies the number of times that the Agent restarts the Envoy proxy process within a running task or pod if it exits\. The Agent also logs the exit status every time Envoy exits to ease troubleshooting\. The default value of this variable is `0`\. When the default value is set, the Agent doesn't attempt to restart the process\.  
Default: `0`  
Maximum: `10`

`PID_POLL_INTERVAL_MS`  
Specifies the interval in milliseconds at which the Envoy proxy’s process state is checked by the Agent\. The default value is `100`\.  
Default: `100`  
Minimum: `100`  
Maximum: `1000`

`LISTENER_DRAIN_WAIT_TIME_S`  
Specifies the amount of time in seconds the Envoy proxy waits for active connections to close before the process exits\.  
Default: `20`  
Minimum: `5`  
Maximum: `110`

### Tracing variables<a name="tracing-variables"></a>

You can configure none or one of the following tracing drivers\.

#### AWS X\-Ray variables<a name="envoy-xray-config"></a>

Use the following environment variables to configure App Mesh with AWS X\-Ray\. For more information, see the [AWS X\-Ray Developer Guide](https://docs.aws.amazon.com/xray/latest/devguide/)\.

`ENABLE_ENVOY_XRAY_TRACING`  
Enables X\-Ray tracing using `127.0.0.1:2000` as the default daemon endpoint\. To enable, set the value to `1`\. The default value is `0`\.

`XRAY_DAEMON_PORT`  
Specify a port value to override the default X\-Ray daemon port: `2000`\.

`XRAY_SAMPLING_RATE`  
Specify a sampling rate to override the X\-Ray tracer's default sampling rate of `0.05` \(5%\)\. Specify the value as a decimal between `0` and `1.00` \(100%\)\. This value is overridden if `XRAY_SAMPLING_RULE_MANIFEST` is specified\. This variable is supported with Envoy images of version `v1.19.1.1-prod` and later\.

`XRAY_SAMPLING_RULE_MANIFEST`  
Specify a file path in the Envoy container file system to configure the localized custom sampling rules for the X\-Ray tracer\. For more information, see [Sampling rules](https://docs.aws.amazon.com/xray/latest/devguide/xray-sdk-go-configuration.html#xray-sdk-go-configuration-sampling) in the *AWS X\-Ray Developer Guide*\. This variable is supported with Envoy images of version `v1.19.1.0-prod` and later\.

#### Datadog tracing variables<a name="datadog-tracing"></a>

The following environment variables help you configure App Mesh with the Datadog agent tracer\. For more information, see [Agent Configuration](https://docs.datadoghq.com/tracing/send_traces/) in the Datadog documentation\.

`ENABLE_ENVOY_DATADOG_TRACING`  
Enables Datadog trace collection using `127.0.0.1:8126` as the default Datadog agent endpoint\. To enable, set the value to `1` \(default value is `0`\)\.

`DATADOG_TRACER_PORT`  
Specify a port value to override the default Datadog agent port: `8126`\.

`DATADOG_TRACER_ADDRESS`  
Specify an IP address to override the default Datadog agent address: `127.0.0.1`\.

`DD_SERVICE`  
Specify a service name for traces to override the default Datadog service name: `envoy-meshName`/`virtualNodeName`\. This variable is supported with Envoy images of version `v1.18.3.0-prod` and later\.

#### Jaeger tracing variables<a name="jaeger-tracing"></a>

Use the following environment variables to configure App Mesh with Jaeger tracing\. For more information, see [Getting Started](https://www.jaegertracing.io/docs/1.21/getting-started/) in the Jaeger documentation\. These variables are supported with Envoy images of version `1.16.1.0-prod` and later\.

`ENABLE_ENVOY_JAEGER_TRACING`  
Enables Jaeger trace collection using `127.0.0.1:9411` as the default Jaeger endpoint\. To enable, set the value to `1` \(default value is `0`\)\.

`JAEGER_TRACER_PORT`  
Specify a port value to override the default Jaeger port: `9411`\.

`JAEGER_TRACER_ADDRESS`  
Specify an IP address to override the default Jaeger address: `127.0.0.1`\.

#### Envoy tracing variable<a name="envoy-tracing"></a>

Set the following environment variable to use your own tracing configuration\. 

`ENVOY_TRACING_CFG_FILE`  
Specify a file path in the Envoy container file system\. For more information, see [https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/trace/v3/http_tracer.proto#envoy-v3-api-msg-config-trace-v3-tracing](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/trace/v3/http_tracer.proto#envoy-v3-api-msg-config-trace-v3-tracing) in the Envoy documentation\.  
If the tracing configuration requires specifying a tracing cluster, make sure to configure the associated cluster configuration under `static_resources` in the same tracing config file\. For example, Zipkin has a [https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/trace/v3/zipkin.proto#config-trace-v3-zipkinconfig](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/trace/v3/zipkin.proto#config-trace-v3-zipkinconfig) field for the cluster name that hosts the trace collectors, and that cluster needs to be statically defined\.

### DogStatsD variables<a name="envoy-dogstatsd-config"></a>

Use the following environment variables to configure App Mesh with DogStatsD\. For more information, see the [DogStatsD](https://docs.datadoghq.com/developers/dogstatsd/) documentation\.

`ENABLE_ENVOY_DOG_STATSD`  
Enables DogStatsD stats using `127.0.0.1:8125` as the default daemon endpoint\. To enable, set the value to `1`\.

`STATSD_PORT`  
Specify a port value to override the default DogStatsD daemon port\.

`STATSD_ADDRESS`  
Specify an IP address value to override the default DogStatsD daemon IP address\. Default: `127.0.0.1`\. This variable can only be used with version `1.15.0` or later of the Envoy image\.

`STATSD_SOCKET_PATH`  
Specify a unix domain socket for the DogStatsD daemon\. If this variable isn't specified and DogStatsD is enabled, then this value defaults to the DogStatsD daemon IP address port of `127.0.0.1:8125`\. If the `ENVOY_STATS_SINKS_CFG_FILE` variable is specified containing a stats sinks configuration, it overrides all of the DogStatsD variables\. This variable is supported with Envoy image version `v1.19.1.0-prod` or later\.

### App Mesh variables<a name="envoy-appmesh-variables"></a>

The following variables help you configure App Mesh\.

`APPMESH_PREVIEW`  
Set the value to `1` to connect to the App Mesh Preview Channel endpoint\. For more information about using the App Mesh Preview Channel, see [App Mesh Preview Channel](preview.md)\.

`APPMESH_RESOURCE_CLUSTER`  
By default, App Mesh uses the name of the resource that you specified in `APPMESH_RESOURCE_ARN` when Envoy is referring to itself in metrics and traces\. You can override this behavior by setting the `APPMESH_RESOURCE_CLUSTER` environment variable with your own name\. This variable can only be used with version `1.15.0` or later of the Envoy image\.

`APPMESH_METRIC_EXTENSION_VERSION`  
Set the value to `1` to enable the App Mesh metrics extension\. For more information about using the App Mesh metrics extension, see [Metrics extension for App Mesh](metrics.md)\.

`APPMESH_DUALSTACK_ENDPOINT`  
Set the value to `1` to connect to App Mesh Dual Stack endpoint\. When this flag is set, Envoy uses our dual stack capable domain\. By default this flag is set to false and only connects to our `IPv4` domain\. This variable can only be used with Envoy image version 1\.22\.0 or later\.

### Envoy stats variables<a name="envoy-stats-config"></a>

Use the following environment variables to configure App Mesh with Envoy Stats\. For more information, see the [Envoy Stats](https://www.envoyproxy.io/docs/envoy/v1.6.0/api-v2/config/metrics/v2/stats.proto) documentation\.

`ENABLE_ENVOY_STATS_TAGS`  
Enables the use of App Mesh defined tags `appmesh.mesh` and `appmesh.virtual_node`\. For more information, see [config\.metrics\.v3\.TagSpecifier](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/metrics/v3/stats.proto#config-metrics-v3-tagspecifier) in the Envoy documentation\. To enable, set the value to `1`\.

`ENVOY_STATS_CONFIG_FILE`  
Specify a file path in the Envoy container file system to override the default Stats tags configuration file with your own\. For more information, see [config\.metrics\.v3\.StatsConfig](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/metrics/v3/stats.proto#config-metrics-v3-statsconfig)\.  
Setting a customized stats configuration that includes stats filters might lead Envoy to enter a state where it will no longer properly synchronize with the App Mesh state of the world\. This is a [bug](https://github.com/envoyproxy/envoy/issues/9856) in Envoy\. Our recommendation is to not perform any filtering of statistics in Envoy\. If filtering is absolutely necessary, we have a listed a couple of workarounds in this [issue](https://github.com/aws/aws-app-mesh-roadmap/issues/283) on our roadmap\.

`ENVOY_STATS_SINKS_CFG_FILE`  
Specify a file path in the Envoy container file system to override the default configuration with your own\. For more information, see [config\.metrics\.v3\.StatsSink](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/metrics/v3/stats.proto#config-metrics-v3-statssink) in the Envoy documentation\.

### Deprecated variables<a name="envoy-deprecated-variables"></a>

The environment variables `APPMESH_VIRTUAL_NODE_NAME` and `APPMESH_RESOURCE_NAME` are no longer supported in Envoy version `1.15.0` or later\. However, they're still supported for existing meshes\. Instead of using these variables with Envoy version `1.15.0` or later, use `APPMESH_RESOURCE_ARN` for all App Mesh endpoints\.
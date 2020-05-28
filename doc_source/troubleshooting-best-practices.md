# App Mesh troubleshooting best practices<a name="troubleshooting-best-practices"></a>

We recommend that you follow the best practices in this topic to troubleshoot issues when using App Mesh\.

## Enable the Envoy proxy administration interface<a name="ts-bp-enable-proxy-admin-interface"></a>

The Envoy proxy ships with an administration interface that you can use to discover configuration and statistics and to perform other administrative functions such as connection draining\. For more information, see [Administration interface](https://www.envoyproxy.io/docs/envoy/latest/operations/admin) in the Envoy documentation\.

If you use the managed [App Mesh Envoy image](envoy.md), the administration endpoint is enabled by default on port 9901\. Examples provided in [App Mesh troubleshooting best practices](#troubleshooting-best-practices) display the example administration endpoint URL as `http://my-app.default.svc.cluster.local:9901/`\. 

**Note**  
The administration endpoint should never be exposed to the public internet\. Additionally, we recommend monitoring the administration endpoint logs, which are set by the `ENVOY_ADMIN_ACCESS_LOG_FILE` environment variable to `/tmp/envoy_admin_access.log` by default\. 

## Enable Envoy DogStatsD integration for metric offload<a name="ts-bp-enable-envoy-statsd-integration"></a>

The Envoy proxy can be configured to offload statistics for OSI Layer 4 and Layer 7 traffic and for internal process health\. While this topic shows how to use these statistics without offloading the metrics to sinks like CloudWatch metrics and Prometheus\., having these statistics in a centralized location for all of your applications can help you diagnose issues and confirm behavior more quickly\. For more information, see [Using Amazon CloudWatch Metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/working_with_metrics.html) and the [Prometheus documentation](https://prometheus.io/docs/introduction/overview/)\. 

You can configure DogStatsD metrics by setting the parameters defined in [App Mesh Envoy image](envoy.md#envoy-dogstatsd-config)\. For more information about DogStatsD, see the [DogStatsD](https://docs.datadoghq.com/developers/dogstatsd/?tab=hostagent) documentation\. You can find a demonstration of metric offload to AWS CloudWatch metrics in the [App Mesh with Amazon ECS basics walk\-through](https://github.com/aws/aws-app-mesh-examples/tree/master/walkthroughs/howto-ecs-basics) on GitHub\.

## Enable access logs<a name="ts-bp-enable-access-logs"></a>

We recommend enabling access logs on your [Virtual nodes](virtual_nodes.md) and [Virtual gateways](virtual_gateways.md) to discover details about traffic transiting between your applications\. For more information, see [Access logging](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/observability/access_logging) in the Envoy documentation\. The logs provide detailed information on OSI Layer 4 and Layer 7 traffic behavior\. When you use Envoy’s default format, you can analyze the access logs with [CloudWatch Logs Insights](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/AnalyzingLogData.html) using the following parse statement\.

```
parse @message "[*] \"* * *\" * * * * * * * * * * *" as StartTime, Method, Path, Protocol, ResponseCode, ResponseFlags, BytesReceived, BytesSent, DurationMillis, UpstreamServiceTimeMillis, ForwardedFor, UserAgent, RequestId, Authority, UpstreamHost
```

## Enable Envoy debug logging in pre\-production environments<a name="ts-bp-enable-envoy-debug-logging"></a>

We recommend setting the Envoy proxy’s log level to `debug` in a pre\-production environment\. Debug logs can help you identify issues before you graduate the associated App Mesh configuration to your production environment\. 

If you’re using the [App Mesh Envoy image](envoy.md), you can set the log level to `debug` through the `ENVOY_LOG_LEVEL` environment variable\. 

**Note**  
We do not recommend using the `debug` level in production environments\. Setting the level to `debug` increases the logging and may affect performance and the overall cost of logs offloaded to solutions like [CloudWatch Logs](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html)\. 

When you use Envoy’s default format, you can analyze the process logs with [CloudWatch Logs Insights](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/AnalyzingLogData.html) using the following parse statement: 

```
parse @message "[*][*][*][*] [*] *" as Time, Thread, Level, Name, Source, Message
```
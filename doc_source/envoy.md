# Envoy Image<a name="envoy"></a>

AWS App Mesh is a service mesh based on the [Envoy](https://www.envoyproxy.io/) proxy\. 

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/app-mesh/latest/userguide/images/proxy.png)

After you create your service mesh, virtual nodes, virtual routers, routes, and virtual services, you must update your microservices to be compatible with App Mesh\.

App Mesh vends the following custom envoy container image that you must add to the task groups that represent your virtual nodes:
+ App Mesh Envoy container image: `111345817488.dkr.ecr.us-west-2.amazonaws.com/aws-appmesh-envoy:v1.9.0.0-prod`

## Envoy Configuration Variables<a name="envoy-config"></a>

The following environment variables enable you to configure the Envoy containers for your App Mesh virtual node task groups\.

### Required Variables<a name="envoy-required-config"></a>

The following environment variable is required for all App Mesh Envoy containers\.

`APPMESH_VIRTUAL_NODE_NAME`  
When you add the Envoy container to a task group, set this environment variable to the name of the virtual node that the task group represents: for example, `mesh/meshName/virtualNode/virtualNodeName`\.

### Optional Variables<a name="envoy-optional-config"></a>

The following environment variable is optional for App Mesh Envoy containers\.

`ENVOY_LOG_LEVEL`  
Specifies the log level for the Envoy container\.  
Valid values: `trace`, `debug`, `info`, `warning`, `error`, `critical`, `off`  
Default: `info`

### AWS X\-Ray Variables<a name="envoy-xray-config"></a>

The following environment variables help you to configure App Mesh with AWS X\-Ray\. For more information, see the [AWS X\-Ray Developer Guide](https://docs.aws.amazon.com/xray/latest/devguide/)\.

`ENABLE_ENVOY_XRAY_TRACING`  
Enables X\-Ray tracing using `127.0.0.1:2000` as the default daemon endpoint\.

`XRAY_DAEMON_PORT`  
Specify a port value to override the default X\-Ray daemon port\.

### DogStatsD Variables<a name="envoy-dogstatsd-config"></a>

The following environment variables help you to configure App Mesh with DogStatsD\. For more information, see the [DogStatsD](https://docs.datadoghq.com/developers/dogstatsd/) documentation\.

`ENABLE_ENVOY_DOG_STATSD`  
Enables DogStatsD stats using `127.0.0.1:8125` as the default daemon endpoint\.

`STATSD_PORT`  
Specify a port value to override the default DogStatsD daemon port\.

`ENVOY_STATS_SINKS_CFG_FILE`  
Specify a file path in the Envoy container file system to override the default DogStatsD configuration with your own\. For more information, see [config\.metrics\.v2\.DogStatsdSink](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/metrics/v2/stats.proto#envoy-api-msg-config-metrics-v2-dogstatsdsink) in the Envoy documentation\.

### Envoy Stats Variables<a name="envoy-stats-config"></a>

The following environment variables help you to configure App Mesh with Envoy Stats\. For more information, see the [Envoy Stats](https://www.envoyproxy.io/docs/envoy/v1.6.0/api-v2/config/metrics/v2/stats.proto) documentation\.

`ENABLE_ENVOY_STATS_TAGS`  
Enables the use of App Mesh defined tags `appmesh.mesh` and `appmesh.virtual_node`\. For more information, see [config\.metrics\.v2\.TagSpecifier](https://www.envoyproxy.io/docs/envoy/v1.6.0/api-v2/config/metrics/v2/stats.proto#envoy-api-msg-config-metrics-v2-tagspecifier) in the Envoy documentation\.

`ENVOY_STATS_CONFIG_FILE`  
Specify a file path in the Envoy container file system to override the default Stats tags configuration file with your own\.

## Access Logs<a name="envoy-logs"></a>

When you create your virtual nodes, you have the option to configure Envoy access logs\. In the console, this is in the **Advanced configuration** section of the virtual node create or update workflows\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/app-mesh/latest/userguide/images/logging.png)

The above image shows a logging path of `/dev/stdout` for Envoy access logs\. The code block below shows the JSON representation that you could use in the AWS CLI\.

```
      "logging": { 
         "accessLog": { 
            "file": { 
               "path": "/dev/stdout"
            }
         }
      }
```

When you send Envoy access logs to `/dev/stdout`, they are mixed in with the Envoy container logs, so you can export them to a log storage and processing service like CloudWatch Logs using standard Docker log drivers \(such as `[awslogs](https://docs.docker.com/config/containers/logging/awslogs/)`\)\. To export only the Envoy access logs \(and ignore the other Envoy container logs\), you can set the `ENVOY_LOG_LEVEL` to `off`\. For more information, see [Access logging](https://www.envoyproxy.io/docs/envoy/latest/configuration/access_log) in the Envoy documentation\.
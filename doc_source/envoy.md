# Envoy image<a name="envoy"></a>

AWS App Mesh is a service mesh based on the [Envoy](https://www.envoyproxy.io/) proxy\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/app-mesh/latest/userguide/images/proxy.png)

You must add an Envoy proxy to the Amazon ECS task, Kubernetes pod, or Amazon EC2 instance represented by your App Mesh virtual nodes\. App Mesh vends an Envoy proxy Docker container image and ensures that this container image is patched with the latest vulnerability and performance patches\. App Mesh tests a new Envoy proxy release against the App Mesh feature set before making a new container image available to you\.

You can retrieve the latest version of the container image using the AWS CLI, by replacing *region\-code* in the following command with one of the [App Mesh supported regions\.](https://docs.aws.amazon.com/general/latest/gr/appmesh.html)

```
aws ssm get-parameter --name /aws/service/appmesh/envoy --region region-code --query "Parameter.Value" --output text
```

Output

```
account-id.dkr.ecr.region-code.amazonaws.com/aws-appmesh-envoy:envoy-image-version
```

If you prefer to use the AWS Management Console, then select the region that you want to use the Envoy proxy in\. The AWS Management Console will open and show you the latest version of the Envoy container image in the **Value** attribute\.
+ [US East \(Ohio\) \(`us-east-2`\)](https://console.aws.amazon.com/systems-manager/parameters/aws/service/appmesh/envoy/description?region=us-east-2)
+ [US East \(N\. Virginia\) \(`us-east-1`\)](https://console.aws.amazon.com/systems-manager/parameters/aws/service/appmesh/envoy/description?region=us-east-1)
+ [US West \(N\. California\) \(`us-west-1`\)](https://console.aws.amazon.com/systems-manager/parameters/aws/service/appmesh/envoy/description?region=us-west-1)
+ [US West \(Oregon\) \(`us-west-2`\)](https://console.aws.amazon.com/systems-manager/parameters/aws/service/appmesh/envoy/description?region=us-west-2)
+ [Asia Pacific \(Hong Kong\) \(`ap-east-1`\)](https://console.aws.amazon.com/systems-manager/parameters/aws/service/appmesh/envoy/description?region=ap-east-1)
+ [Asia Pacific \(Mumbai\) \(`ap-south-1`\)](https://console.aws.amazon.com/systems-manager/parameters/aws/service/appmesh/envoy/description?region=ap-south-1)
+ [Asia Pacific \(Seoul\) \(`ap-northeast-2`\)](https://console.aws.amazon.com/systems-manager/parameters/aws/service/appmesh/envoy/description?region=ap-northeast-2)
+ [Asia Pacific \(Singapore\) \(`ap-southeast-1`\)](https://console.aws.amazon.com/systems-manager/parameters/aws/service/appmesh/envoy/description?region=ap-southeast-1)
+ [Asia Pacific \(Sydney\) \(`ap-southeast-2`\)](https://console.aws.amazon.com/systems-manager/parameters/aws/service/appmesh/envoy/description?region=ap-southeast-2)
+ [Asia Pacific \(Tokyo\) \(`ap-northeast-1`\)](https://console.aws.amazon.com/systems-manager/parameters/aws/service/appmesh/envoy/description?region=us-ap-northeast-1)
+ [Canada \(Central\) \(`ca-central-1`\)](https://console.aws.amazon.com/systems-manager/parameters/aws/service/appmesh/envoy/description?region=ca-central-1)
+ [Europe \(Frankfurt\) \(`eu-central-1`\)](https://console.aws.amazon.com/systems-manager/parameters/aws/service/appmesh/envoy/description?region=eu-central-1)
+ [Europe \(Ireland\) \(`eu-west-1`\)](https://console.aws.amazon.com/systems-manager/parameters/aws/service/appmesh/envoy/description?region=eu-west-1)
+ [Europe \(London\) \(`eu-west-2`\)](https://console.aws.amazon.com/systems-manager/parameters/aws/service/appmesh/envoy/description?region=eu-west-2)
+ [Europe \(Paris\) \(`eu-west-3`\)](https://console.aws.amazon.com/systems-manager/parameters/aws/service/appmesh/envoy/description?region=eu-west-3)
+ [Europe \(Stockholm\) \(`eu-north-1`\)](https://console.aws.amazon.com/systems-manager/parameters/aws/service/appmesh/envoy/description?region=eu-north-1)
+ [Middle East \(Bahrain\) \(`me-south-1`\)](https://console.aws.amazon.com/systems-manager/parameters/aws/service/appmesh/envoy/description?region=me-south-1)
+ [South America \(São Paulo\) \(`sa-east-1`\)](https://console.aws.amazon.com/systems-manager/parameters/aws/service/appmesh/envoy/description?region=sa-east-1)

Access to this container image in Amazon ECR is controlled by AWS Identity and Access Management, so you must use IAM to ensure that you have read access to Amazon ECR\. For example, when using Amazon ECS, you can assign an appropriate task execution role to an Amazon ECS task\. Further, if you use IAM policies that limit access to specific Amazon ECR resources, then you must ensure that you allow access to the Region\-specific Amazon Resource Name \(ARN\) that identifies the `aws-appmesh-envoy` repository\. For example, in the `us-west-2` region, you'd allow access to the following resource: `arn:aws:ecr:us-west-2:840364872350:repository/aws-appmesh-envoy`\. For more information, see [Amazon ECR Managed Policies](https://docs.aws.amazon.com/AmazonECR/latest/userguide/ecr_managed_policies.html)\. If you're using Docker on an Amazon EC2 instance, then you need to authenticate Docker to the repository\. For more information, see [Registry authentication](https://docs.aws.amazon.com/AmazonECR/latest/userguide/Registries.html#registry_auth)\.

We occassionally release new App Mesh features that depend on Envoy changes that have not been merged to the upstream Envoy images yet\. To use these new App Mesh features before the Envoy changes are merged upstream, you must use the App Mesh\-vended Envoy container image\. For a list of changes, see the [App Mesh GitHub roadmap issues](https://github.com/aws/aws-app-mesh-roadmap/labels/Envoy%20Upstream) with the `Envoy Upstream` label\. Otherwise, while we recommend that you use the App Mesh Envoy container image as the best supported option, you may use your own Envoy image\.

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

### AWS X\-Ray variables<a name="envoy-xray-config"></a>

The following environment variables help you to configure App Mesh with AWS X\-Ray\. For more information, see the [AWS X\-Ray Developer Guide](https://docs.aws.amazon.com/xray/latest/devguide/)\.

`ENABLE_ENVOY_XRAY_TRACING`  
Enables X\-Ray tracing using `127.0.0.1:2000` as the default daemon endpoint\.

`XRAY_DAEMON_PORT`  
Specify a port value to override the default X\-Ray daemon port\.

### DogStatsD variables<a name="envoy-dogstatsd-config"></a>

The following environment variables help you to configure App Mesh with DogStatsD\. For more information, see the [DogStatsD](https://docs.datadoghq.com/developers/dogstatsd/) documentation\.

`ENABLE_ENVOY_DOG_STATSD`  
Enables DogStatsD stats using `127.0.0.1:8125` as the default daemon endpoint\.

`STATSD_PORT`  
Specify a port value to override the default DogStatsD daemon port\.

`ENVOY_STATS_SINKS_CFG_FILE`  
Specify a file path in the Envoy container file system to override the default DogStatsD configuration with your own\. For more information, see [config\.metrics\.v2\.DogStatsdSink](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/metrics/v2/stats.proto#envoy-api-msg-config-metrics-v2-dogstatsdsink) in the Envoy documentation\.

### Envoy stats variables<a name="envoy-stats-config"></a>

The following environment variables help you to configure App Mesh with Envoy Stats\. For more information, see the [Envoy Stats](https://www.envoyproxy.io/docs/envoy/v1.6.0/api-v2/config/metrics/v2/stats.proto) documentation\.

`ENABLE_ENVOY_STATS_TAGS`  
Enables the use of App Mesh defined tags `appmesh.mesh` and `appmesh.virtual_node`\. For more information, see [config\.metrics\.v2\.TagSpecifier](https://www.envoyproxy.io/docs/envoy/v1.6.0/api-v2/config/metrics/v2/stats.proto#envoy-api-msg-config-metrics-v2-tagspecifier) in the Envoy documentation\.

`ENVOY_STATS_CONFIG_FILE`  
Specify a file path in the Envoy container file system to override the default Stats tags configuration file with your own\.

## Access logs<a name="envoy-logs"></a>

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

When you send Envoy access logs to `/dev/stdout`, they are mixed in with the Envoy container logs, so you can export them to a log storage and processing service like CloudWatch Logs using standard Docker log drivers \(such as `[awslogs](https://docs.docker.com/config/containers/logging/awslogs/)`\)\. To export only the Envoy access logs \(and ignore the other Envoy container logs\), you can set the `ENVOY_LOG_LEVEL` to `off`\. For more information, see [Access logging](https://www.envoyproxy.io/docs/envoy/latest/configuration/observability/access_log/access_log.html) in the Envoy documentation\.
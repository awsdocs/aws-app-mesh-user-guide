# Logging<a name="envoy-logs"></a>

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

Envoy also writes various debugging logs from its filters to `stdout`\. These logs are useful for gaining insights into both Envoy’s communication with App Mesh and service\-to\-service traffic\. Your specific logging level can be configured using the `ENVOY_LOG_LEVEL` environment variable\. For example, the following text is from an example debug log showing the cluster that Envoy matched for a particular HTTP request\.

```
[debug][router] [source/common/router/router.cc:434] [C4][S17419808847192030829] cluster 'cds_ingress_howto-http2-mesh_color_client_http_8080' match for URL '/ping'
```

## Firelens and Cloudwatch<a name="firelens-cloudwatch-logging"></a>

[Firelens](https://aws.amazon.com/about-aws/whats-new/2019/11/aws-launches-firelens-log-router-for-amazon-ecs-and-aws-fargate/#:~:text=FireLens%20is%20a%20container%20log,for%20log%20analytics%20and%20storage.&text=This%20means%20you%20can%20use,your%20own%20Fluentd%20output%20plugin.) is a container log router you can use to collect logs for Amazon ECS and AWS Fargate\. You can find an example of using Firelens in our [AWS Samples repository](https://github.com/aws-samples/amazon-ecs-firelens-examples)\.

You can use CloudWatch to gather logging information as well as metrics\. You can find more information on CloudWatch in our [Metrics](https://docs.aws.amazon.com/app-mesh/latest/userguide/metrics.html#cloudwatch) section of the App Mesh docs\.
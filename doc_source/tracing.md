# Tracing<a name="tracing"></a>

**Important**  
To fully implement tracing, you'll need to update your application\.  
To see all the available data from your chosen service, you'll have to instrument your application using the applicable libraries\.

## Monitor App Mesh with AWS X\-Ray<a name="x-ray"></a>

AWS X\-Ray is a service that provides tools that let you view, filter, and gain insights into data collected from the requests your application serves\. These insights help you identify issues and opportunities to optimize your app\. You can see detailed information about requests and responses, and downstream calls your application makes to other AWS services\.

X\-Ray integrates with App Mesh to manage your Envoy microservices\. Trace data from Envoy is sent to the X\-Ray daemon running in your container\.

Implement X\-Ray in your application code using the [SDK](https://docs.aws.amazon.com/xray/index.html) guide specific to your language\.

### Enable X\-Ray tracing through App Mesh<a name="enable-x-ray"></a>
+ 

**Depending on the type of service:**
  + **ECS \-** In the Envoy proxy container definition, set the `ENABLE_ENVOY_XRAY_TRACING` environment variable to `1` and the `XRAY_DAEMON_PORT` environment variable to `2000`\.
  + **EKS \-** In the App Mesh Controller configuration, include `--set tracing.enabled=true` and `--set tracing.provider=x-ray`\.
+ In your X\-Ray container, expose port `2000` and run as user `1337`\.

### X\-Ray examples<a name="x-ray-examples"></a>

An Envoy container definition for Amazon ECS

```
      {
        "name": "envoy",
        "image": "840364872350.dkr.ecr.us-west-2.amazonaws.com/aws-appmesh-envoy:v1.15.1.0-prod",
        "essential": true,
        "environment": [
          {
            "name": "APPMESH_VIRTUAL_NODE_NAME",
            "value": "mesh/myMesh/virtualNode/myNode"
          },
          {
            "name": "ENABLE_ENVOY_XRAY_TRACING",
            "value": "1"
           }
        ],
        "healthCheck": {
          "command": [
            "CMD-SHELL",
            "curl -s http://localhost:9901/server_info | cut -d' ' -f3 | grep -q live"
            ],
           "startPeriod": 10,
           "interval": 5,
           "timeout": 2,
           "retries": 3
      }
```

Updating the App Mesh controller for Amazon EKS

```
helm upgrade -i appmesh-controller eks/appmesh-controller \
--namespace appmesh-system \
--set region=${AWS_REGION} \
--set serviceAccount.create=false \
--set serviceAccount.name=appmesh-controller \
--set tracing.enabled=true \
--set tracing.provider=x-ray
```

### Walkthroughs for using the X\-Ray<a name="x-ray-walkthrough"></a>
+ [Monitor with AWS X\-Ray](https://github.com/aws/aws-app-mesh-examples/tree/master/examples/apps/colorapp#monitor-with-aws-x-ray)
+ [App Mesh with Amazon EKS \- Observability: X\-Ray](https://github.com/aws/aws-app-mesh-examples/blob/master/walkthroughs/eks/o11y-xray.md)
+ [Distributed tracing with X\-Ray](https://www.appmeshworkshop.com/x-ray/) in the AWS App Mesh [Workshop](https://www.appmeshworkshop.com/introduction/)

### To learn more about AWS X\-Ray<a name="x-ray-learn-more"></a>
+ [AWS X\-Ray documentation](https://docs.aws.amazon.com/xray/index.html)

### Troubleshooting AWS X\-Ray with App Mesh<a name="x-ray-troubleshooting"></a>
+ [Unable to see AWS X\-Ray traces for my applications\.](https://docs.aws.amazon.com/app-mesh/latest/userguide/troubleshoot-observability.html)

## Jaeger for App Mesh with Amazon EKS<a name="jaeger"></a>

Jaeger is an open source, end to end distributed tracing system\. It can be used to profile networks and for monitoring\. Jaeger can also help you troubleshoot complex cloud native applications\.

To implement Jaeger into your application code, you can find the guide specific to your language in the Jaeger documentation [tracing libraries](https://www.jaegertracing.io/docs/1.21/client-libraries/)\.

### Installing Jaeger using Helm<a name="installing-jaeger"></a>

1. Add the EKS repository to Helm:

   ```
   helm repo add eks https://aws.github.io/eks-charts
   ```

1. Install App Mesh Jaeger

   ```
   helm upgrade -i appmesh-jaeger eks/appmesh-jaeger \
   --namespace appmesh-system
   ```

### Jaeger Example<a name="jaeger-sample"></a>

The following is an example of creating a `PersistentVolumeClaim` for Jaeger persistent storage\.

```
helm upgrade -i appmesh-controller eks/appmesh-controller \
--namespace appmesh-system \
--set tracing.enabled=true \
--set tracing.provider=jaeger \
--set tracing.address=appmesh-jaeger.appmesh-system \
--set tracing.port=9411
```

### Walkthrough for using the Jaeger<a name="jaeger-walkthrough"></a>
+ [App Mesh with EKS—Observability: Jaeger](https://github.com/aws/aws-app-mesh-examples/blob/master/walkthroughs/eks/o11y-jaeger.md)

### To learn more about Jaeger<a name="jaeger-eks"></a>
+ [Jaeger Documentation](https://www.jaegertracing.io/)

## Datadog for tracing<a name="datadog-tracing"></a>

Datadog can be used for tracing as well as metrics\. For more information and installation instructions, find the guide specific to your application language in the [Datadog documentation](https://docs.datadoghq.com/tracing/setup_overview/)\.
# App Mesh setup troubleshooting<a name="troubleshooting-setup"></a>

This topic details common issues that you may experience with App Mesh setup\.

## Cannot pull Envoy container image<a name="ts-setup-cannot-pull-envoy"></a>

**Symptoms**  
You receive the following error message in an Amazon ECS task\. The Amazon ECR *account ID* and *Region* in the following message may be different, depending on which Amazon ECR repository that you pulled the container image from\.

```
CannotPullContainerError: Error response from daemon: pull access denied for 840364872350.dkr.ecr.us-west-2.amazonaws.com/aws-appmesh-envoy, repository does not exist or may require 'docker login'
```

**Resolution**  
This error indicates that the task execution role being used does not have permission to communicate to Amazon ECR and cannot pull the Envoy container image from the repository\. The task execution role assigned to your Amazon ECS task needs an IAM policy with the following statements:

```
{
  "Action": [
    "ecr:BatchCheckLayerAvailability",
    "ecr:GetDownloadUrlForLayer",
    "ecr:BatchGetImage"
  ],
  "Resource": "arn:aws:ecr:us-west-2:111122223333:repository/aws-appmesh-envoy",
  "Effect": "Allow"
},
{
  "Action": "ecr:GetAuthorizationToken",
  "Resource": "*",
  "Effect": "Allow"
}
```

If you still can't pull the image, then consider opening a [GitHub issue](https://github.com/aws/aws-app-mesh-roadmap/issues/new?assignees=&labels=Bug&template=issue--bug-report.md&title=Bug%3A+describe+bug+here) or contact [AWS Support](https://aws.amazon.com/premiumsupport/)\.

## Cannot connect to App Mesh Envoy management service<a name="ts-setup-cannot-connect-ems"></a>

**Symptoms**  
Your Envoy proxy is unable to connect to the App Mesh Envoy management service\. You are seeing:
+ Connection refused errors
+ Connection timeouts
+ Errors resolving the App Mesh Envoy management service endpoint

**Resolution**  
Make sure that your Envoy proxy has access to the internet or to a private [VPC endpoint](vpc-endpoints.md) and that your [security groups](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html) allow egress traffic on port 443\. App Mesh’s public Envoy management service endpoints follow the fully qualified domain name \(FQDN\) format\.

```
# App Mesh Production Endpoint
appmesh-envoy-management.region-code.amazonaws.com

# App Mesh Preview Endpoint
appmesh-preview-envoy-management.region-code.amazonaws.com
```

If you still can't connect to the App Mesh Envoy management service, then consider opening a [GitHub issue](https://github.com/aws/aws-app-mesh-roadmap/issues/new?assignees=&labels=Bug&template=issue--bug-report.md&title=Bug%3A+describe+bug+here) or contact [AWS Support](https://aws.amazon.com/premiumsupport/)\.

## Envoy disconnected from App Mesh Envoy management service with error text<a name="ts-setup-grpc-error-codes"></a>

**Symptoms**  
Your Envoy proxy is unable to connect to the App Mesh Envoy management service and receive its configuration\. Your Envoy proxy logs contain a log entry like the following\.

```
gRPC config stream closed: gRPC status code, message
```

**Resolution**  
In most cases, the message portion of the log should indicate the problem\. The following table lists the most common gRPC status codes that you might see, their causes, and their resolutions\.


| gRPC status code | Cause | Resolution | 
| --- | --- | --- | 
| 0 | Graceful disconnect from the Envoy management service\. | There is no issue\. App Mesh occasionally disconnects Envoy proxies with this status code\. Envoy will reconnect and continue receiving updates\. | 
| 3 | The mesh endpoint \(virtual node or virtual gateway\), or one of its associated resources, could not be found\. | Double check your Envoy configuration to make sure that it has the appropriate name of the App Mesh resource that it represents\. If your App Mesh resource is integrated with other AWS resources, such as AWS Cloud Map namespaces or ACM certificates, then make sure that those resources exist\. | 
| 7 | The Envoy proxy is unauthorized to perform an action, such as connect to the Envoy management service, or retrieve associated resources\. | Make sure that you [create an IAM policy ](proxy-authorization.md#create-iam-policy) that has the appropriate policy statements for App Mesh and other services and attach that policy to the IAM user or role that your Envoy proxy is using to connect to the Envoy management service\.  | 
| 8 | The number of Envoy proxies for a given App Mesh resource exceeds the account\-level service quota\. | See [App Mesh service quotas](service-quotas.md) for information on default account quotas and how to request a quota increase\. | 
| 16 | The Envoy proxy does not have valid authentication credentials for AWS\. | Make sure that the Envoy has appropriate credentials to connect to AWS services through an IAM user or role\. | 

You can observe the status codes and messages from your Envoy Proxy with [CloudWatch Logs Insights](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/AnalyzingLogData.html) by using the following query:

```
filter @message like /gRPC config stream closed/
| parse @message "gRPC config stream closed: *, *" as StatusCode, Message
```

If the provided error message was not helpful, or your issue is still not resolved, then consider opening a [GitHub issue](https://github.com/aws/aws-app-mesh-roadmap/issues/new?assignees=&labels=Bug&template=issue--bug-report.md&title=Bug%3A+describe+bug+here)\.

## Envoy container health check, readiness probe, or liveliness probe failing<a name="ts-setup-envoy-container-checks"></a>

**Symptoms**  
Your Envoy proxy is failing health checks in an Amazon ECS task, Amazon EC2 instance, or Kubernetes pod\. For example, you query the Envoy administration interface with the following command and receive a status other than `LIVE`\.

```
curl -s http://my-app.default.svc.cluster.local:9901/server_info | jq '.state'
```

**Resolution**  
The following is a list of remediation steps depending on the status returned by the Envoy proxy\.
+ `PRE_INITIALIZING` or `INITIALIZING` – The Envoy proxy has yet to receive configuration, or cannot connect and retrieve configuration from App Mesh Envoy management service\. The Envoy may be receiving an error from the Envoy management service when trying to connect\. For more information, see the errors in [Envoy disconnected from App Mesh Envoy management service with error text](#ts-setup-grpc-error-codes)\.
+ `DRAINING` – The Envoy proxy has begun draining connections in response to a `/healthcheck/fail` or `/drain_listeners` request on the Envoy administration interface\. We do not recommend invoking these paths on the administration interface unless you are about to terminate your Amazon ECS task, Amazon EC2 instance, or Kubernetes pod\.

If your Envoy proxy is still failing health checks, then consider opening a [GitHub issue](https://github.com/aws/aws-app-mesh-roadmap/issues/new?assignees=&labels=Bug&template=issue--bug-report.md&title=Bug%3A+describe+bug+here) or contact [AWS Support](https://aws.amazon.com/premiumsupport/)\.

## Health check from the load balancer to the mesh endpoint failing<a name="ts-setup-endpoint-lb-checks"></a>

**Symptoms**
Your mesh endpoint is considered healthy by the container health check or readiness probe, but the health check from the load balancer to the mesh endpoint is failing.

**Resolution**
+ Make sure the [Security Group](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html) associated with your mesh endpoint accepts ingress traffic on the port you've configured for your health check.
+ Make sure the health check is succeeding consistently when requested manually (for example, from a [bastion host within your VPC](https://aws.amazon.com/quickstart/architecture/linux-bastion/))
+ If you are configuring a health check for a Virtual Node, we recommend implementing a health check endpoint in your application (e.g. /ping for HTTP). This ensures that both the Envoy proxy and your application are routable from the load balancer. 
  + You can use any Elastic Load Balancer type for the Virtual Node, depending on the features you need. See [Elastic Load Balancing features](https://aws.amazon.com/elasticloadbalancing/features/#compare).
+ If you are configuring a health check for a Virtual Gateway, we recommend using a [Network Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/network-load-balancers.html) with a TCP health check on the Virtual Gateway's listener port. This ensures that the Virtual Gateway listener is bootstrapped and ready to accept connections.

If your mesh endpoint is still failing health checks from the load balancer, then consider opening a [GitHub issue](https://github.com/aws/aws-app-mesh-roadmap/issues/new?assignees=&labels=Bug&template=issue--bug-report.md&title=Bug%3A+describe+bug+here) or contact [AWS Support](https://aws.amazon.com/premiumsupport/)\.

## HTTP 503 errors during application deployment<a name="ts-setup-503-during-deployment"></a>

**Symptoms**  
When deploying a new revision of an application in App Mesh, each client application’s Envoy has to be made aware of the new endpoints\. In effect, all virtual nodes in a service mesh are redirecting traffic to a new destination\. As you might expect, these are exactly the kinds of scenarios where failed requests can creep into your system\.

While in some cases these failures can occur due to failed in\-flight requests, they can also occur because Envoy can enter a brief state where it has no viable place to send traffic\. For more information about implementing resilient retries, see [Instrument all routes with retries](bp-minimizing-downtime.md#route-retries)\.

For example, suppose that you're seeing a significant number of errors during a deployment of a virtual node named `virtual-node-a` in a mesh named `prod`\. You can look at the metrics from one of the applications reporting failures, and you'll see something like this:

```
cluster.cds_egress_prod_virtual-node-a_http_3000.health_check.attempt: 1960
cluster.cds_egress_prod_virtual-node-a_http_3000.health_check.degraded: 0
cluster.cds_egress_prod_virtual-node-a_http_3000.*health_check.failure*: 7
cluster.cds_egress_prod_virtual-node-a_http_3000.health_check.healthy: 3
cluster.cds_egress_prod_virtual-node-a_http_3000.health_check.network_failure: 2
cluster.cds_egress_prod_virtual-node-a_http_3000.health_check.passive_failure: 0
cluster.cds_egress_prod_virtual-node-a_http_3000.health_check.success: 1953
cluster.cds_egress_prod_virtual-node-a_http_3000.health_check.verify_cluster: 0
cluster.cds_egress_prod_virtual-node-a_http_3000.*lb_healthy_panic*: 559
cluster.cds_egress_prod_virtual-node-a_http_3000.membership_healthy: 3
cluster.cds_egress_prod_virtual-node-a_http_3000.*upstream_cx_none_healthy*: 495
```

While the number of health failures is small \(you’re not seeing a persistent issue\) in the previous example metrics, you see there is a significant number of times when the cluster was in panic because less than 50% of destinations were healthy\. Worse, there was a period where no hosts were healthy\. This tells you that Envoy saw a period where no destination for `virtual-node-a` traffic was healthy\. This typically occurs because App Mesh is spinning up and tearing down applications faster than it is able to reconcile these changes at the Envoy\.

**Resolution**  
To mitigate against this, we recommend setting a gradual rolling deployment:
+ For Amazon ECS, set a minimum healthy percentage of 100% and a maximum healthy percentage of 125% \(if 4 or more tasks\) or 150% \(if less than 4 tasks\)
+ For Kubernetes, set a minimum healthy percentage of 100% and a maximum healthy percentage of 125%

We also recommend setting a retry strategy on your application with the following configuration:

```
retryPolicy:
   maxRetries: 3
   perRetryTimeout:
     value: 1
     unit: s
   grpcRetryEvents:
     - cancelled
     - unavailable
   httpRetryEvents:
     - server-error
     - gateway-error
   tcpRetryEvents:
     - connection-error
```

It is important not to set your `perRetryTimeout` too low, or the elevation in the `upstream_rq_per_try_timeout` metric for the virtual node can actually make your failed request worse\. For example, if you accidentally set a timeout of 5 ms rather than 5s, you would see the following metrics for a virtual node named `virtual-node-a` in a mesh named `prod`\.

```
cluster.cds_egress_prod_user_http_3000.upstream_rq_504: 1230
cluster.cds_egress_prod_user_http_3000.upstream_rq_5xx: 1230
cluster.cds_egress_prod_user_http_3000.upstream_rq_per_try_timeout: 1230
```

As you can see in the previous example, every timed out request was categorized as a `504` and sent back to the application as such\. Correcting the units error fixes the issue, and you would no longer see timeouts\.

If your issue is still not resolved, then consider opening a [GitHub issue](https://github.com/aws/aws-app-mesh-roadmap/issues/new?assignees=&labels=Bug&template=issue--bug-report.md&title=Bug%3A+describe+bug+here) or contact [AWS Support](https://aws.amazon.com/premiumsupport/)\.
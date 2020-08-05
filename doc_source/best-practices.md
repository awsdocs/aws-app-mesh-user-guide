# App Mesh best practices<a name="best-practices"></a>

To achieve the goal of zero failed requests during planned deployments and during the unplanned loss of some hosts, the best practices in this topic implement the following strategy:
+ Increase the likelihood that a request will succeed from the perspective of the application by using a safe default retry strategy\. For more information, see [Instrument all routes with retries](#route-retries)\.
+ Increase the likelihood that a retried request succeeds by maximizing the likelihood that the retried request is sent to an actual destination\. For more information, see [Adjust deployment velocity](#reduce-deployment-velocity), [Scale out before scale in](#scale-out), and [Implement container health checks](#health-checks)\.

To significantly reduce or eliminate failures, we recommend that you implement the recommendations in all of the following practices\.

## Instrument all routes with retries<a name="route-retries"></a>

Configure all virtual services to use a virtual router and set a default retry policy for all routes\. This will mitigate failed requests by reselecting a host and sending a new request\.  For retry policies, we recommend a value of at least two for `maxRetries`, and specifying the following options for each type of retry event in each route type that supports the retry event type:
+ **TCP** – `connection-error`
+ **HTTP and HTTP/2** – `stream-error` and `gateway-error`
+ **gRPC** – `cancelled` and `unavailable`

Other retry events need to be considered on a case\-by\-case basis as they may not be safe, such as if the request isn’t idempotent\. You will need to consider and test values for `maxRetries` and `perRetryTimeout` that make the appropriate trade off between the maximum latency of a request \(`maxRetries` \* `perRetryTimeout`\) versus the increased success rate of more retries\. Additionally, when Envoy attempts to connect to an endpoint that is no longer present, you should expect that request to consume the full `perRetryTimeout`\. To configure a retry policy, see [Creating a route](routes.md#create-route) and then select the protocol that you want to route\.

**Note**  
If you implemented a route on or after July 29, 2020 and didn't specify a retry policy, then App Mesh may have automatically created a default retry policy similar to the previous policy for each route you created on or after July 29, 2020\. For more information, see [Default route retry policy](envoy.md#default-retry-policy)\.

## Adjust deployment velocity<a name="reduce-deployment-velocity"></a>

When using rolling deployments, reduce the overall deployment velocity\. By default, Amazon ECS configures a deployment strategy of a minimum of 100 percent healthy tasks and 200 percent total tasks\. On deployment, this results in two points of high drift:
+ The 100 percent fleet size of new tasks may be visible to Envoys prior to being ready to complete requests \(see [Implement container health checks](#health-checks) for mitigations\)\.
+ The 100 percent fleet size of old tasks may be visible to Envoys while the tasks are being terminated\.

When configured with these deployment constraints, container orchestrators may enter a state where they are simultaenously hiding all old destinations and making all new destinations visible\. Because your Envoy dataplane is eventually consistent, this can result in periods where the set of destinations visible in your dataplane have diverged from the orchestrator’s point of view\. To mitigate this, we recommend maintaining a minimum of 100 percent healthy tasks, but lowering total tasks to 125 percent\. This will reduce divergence and improve the reliability of retries\. We recommend the following settings for different container runtimes:

**Amazon ECS**  
If your service has a desired count of two or three, set `maximumPercent` to 150 percent\. Otherwise, set `maximumPercent` to 125 percent\.

**Kubernetes**  
Configure your deployment's [update strategy](https://v1-16.docs.kubernetes.io/docs/reference/generated/kubernetes-api/v1.16/#deploymentstrategy-v1beta2-apps), setting `maxUnavailable` to 0 percent and `maxSurge` to 25 percent\. 

## Scale out before scale in<a name="scale-out"></a>

Scale out and scale in can both result in some probability of failed requests in retries\. While there are task recommendations that mitigate scale out, the only recommendation for scale in is to minimize the percentage of scaled in tasks at any one time\. We recommend that you use a deployment strategy that scales out new Amazon ECS tasks or Kubernetes deployments prior to scaling in old tasks or deployments\. This scaling strategy keeps your percentage of scaled in tasks or deployments lower, while maintaining the same velocity\. This practice applies to both Amazon ECS tasks and Kubernetes deployments\.

## Implement container health checks<a name="health-checks"></a>

In the scale up scenario, containers in an Amazon ECS task may come up out of order and may not be initially responsive\. We recommend the following suggestions for different container runtimes:

**Amazon ECS**  
To mitigate this, we recommend using container health checks and container dependency ordering to ensure that Envoy is running and healthy prior to any containers requiring outbound network connectivity starting\. To correctly configure an application container and Envoy container in a task definition, see [Container dependency](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/example_task_definitions.html#example_task_definition-containerdependency)\.

**Kubernetes**  
None, because Kubernetes [liveness and readiness](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/) probes are not being considered in registration and de\-registration of AWS Cloud Map instances in the [App Mesh controller for Kubernetes](https://github.com/aws/aws-app-mesh-controller-for-k8s)\. For more information, see GitHub issue [\#132](https://github.com/aws/aws-app-mesh-controller-for-k8s/issues/132)\.
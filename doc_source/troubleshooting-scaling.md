# App Mesh scaling troubleshooting<a name="troubleshooting-scaling"></a>

This topic details common issues that you may experience with App Mesh scaling\.

## Connectivity fails and container health checks fail when scaling beyond 10 replicas for a virtual node<a name="ts-scaling-exceed-virtual-node-envoy-quota"></a>

**Symptoms**  
When you are scaling the number of replicas, such as Amazon ECS tasks, Kubernetes pods, or Amazon EC2 instances, for a virtual node beyond 10, Envoy container health checks for new and currently running Envoys begin to fail\. Downstream applications sending traffic to the virtual node begin seeing request failures with HTTP status code `503`\.

**Resolution**  
App Mesh's default quota for the number of Envoys per virtual node is 10\. When the number of running Envoys exceeds this quota, new and currently running Envoys fail to connect to App Mesh's Envoy management service with gRPC status code `8` \(`RESOURCE_EXHAUSTED`\)\. This quota can be raised\. For more information, see [App Mesh service quotas](service-quotas.md)\.

If your issue is still not resolved, then consider opening a [GitHub issue](https://github.com/aws/aws-app-mesh-roadmap/issues/new?assignees=&labels=Bug&template=issue--bug-report.md&title=Bug%3A+describe+bug+here) or contact [AWS Support](http://aws.amazon.com/premiumsupport/)\.

## Requests fail with `503` when a virtual service backend horizontally scales out or in<a name="ts-scaling-out-in"></a>

**Symptoms**  
When a backend virtual service is horizontally scaled out or in, requests from downstream applications fail with an `HTTP 503` status code\.

**Resolution**  
App Mesh recommends several approaches to mitigate failure cases while scaling applications horizontally\. For detailed information about how to prevent these failures, see [App Mesh best practices](best-practices.md)\.

If your issue is still not resolved, then consider opening a [GitHub issue](https://github.com/aws/aws-app-mesh-roadmap/issues/new?assignees=&labels=Bug&template=issue--bug-report.md&title=Bug%3A+describe+bug+here) or contact [AWS Support](http://aws.amazon.com/premiumsupport/)\.

## Envoy container crashes with segfault under increased load<a name="ts-scaling-segfault"></a>

**Symptoms**  
Under a high traffic load, the Envoy proxy crashes due to a segmentation fault \(Linux exit code `139`\)\. The Envoy process logs contain a statement like the following\.

```
Caught Segmentation fault, suspect faulting address 0x0"
```

**Resolution**  
The Envoy proxy has likely breached the operating system's default nofile ulimit, the limit on the number of files a process can have open at a time\. This breach is due to the traffic causing more connections, which consume additional operating system sockets\. To resolve this issue, increase the ulimit nofile value on the host operating system\. If you are using Amazon ECS, this limit can be changed through the [Ulimit settings](https://docs.aws.amazon.com/AmazonECS/latest/APIReference/API_Ulimit.html) on the task definition's [resource limits settings](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definition_parameters.html#container_definition_limits)\.

If your issue is still not resolved, then consider opening a [GitHub issue](https://github.com/aws/aws-app-mesh-roadmap/issues/new?assignees=&labels=Bug&template=issue--bug-report.md&title=Bug%3A+describe+bug+here) or contact [AWS Support](http://aws.amazon.com/premiumsupport/)\.
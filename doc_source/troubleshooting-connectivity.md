# App Mesh connectivity troubleshooting<a name="troubleshooting-connectivity"></a>

This topic details common issues that you may experience with App Mesh connectivity\.

## Unable to resolve DNS name for a virtual service<a name="ts-connectivity-dns-resolution-virtual-service"></a>

**Symptoms**  
Your application is unable to resolve the DNS name of a virtual service that it is attempting to connect to\.

**Resolution**  
This is a known issue\. For more information, see the [Name VirtualServices by any hostname or FQDN](https://github.com/aws/aws-app-mesh-roadmap/issues/65) GitHub issue\. Virtual services in App Mesh can be named anything\. As long as there is a DNS `A` record for the virtual service name and the application can resolve the virtual service name, the request will be proxied by Envoy and routed to its appropriate destination\. To resolve the issue, add a DNS `A` record to any non\-loopback IP address, such as `10.10.10.10`, for the virtual service name\. The DNS `A` record can be added under the following conditions:
+ In Amazon Route 53, if the name is suffixed by your private hosted zone name
+ Within the application container's `/etc/hosts` file
+ In a third\-party DNS server that you manage

If your issue is still not resolved, then consider opening a [GitHub issue](https://github.com/aws/aws-app-mesh-roadmap/issues/new?assignees=&labels=Bug&template=issue--bug-report.md&title=Bug%3A+describe+bug+here) or contact [AWS Support](http://aws.amazon.com/premiumsupport/)\.

## Unable to connect to a virtual service backend<a name="ts-connectivity-virtual-service-backend"></a>

**Symptoms**  
Your application is unable to establish a connection to a virtual service defined as a backend on your virtual node\. When attempting to establish a connection, the connection may fail entirely, or the request from the application's perspective may fail with an `HTTP 503` response code\.

**Resolution**  
If the application fails to connect at all \(no `HTTP 503` response code returned\), then do the following:
+ Make sure that your compute environment has been set up to work with App Mesh\.
  + For Amazon ECS, make sure that you have the appropriate [proxy configuration](proxy-authorization.md) enabled\. For an end\-to\-end walkthrough, see [Getting Started with App Mesh and Amazon ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/appmesh-getting-started.html)\.
  + For Kubernetes, including Amazon EKS, make sure that you have the latest App Mesh controller installed via Helm\. For more information, see [App Mesh Controller](https://hub.helm.sh/charts/aws/appmesh-controller) on Helm Hub or [Tutorial: Configure App Mesh integration with Kubernetes](https://docs.aws.amazon.com/app-mesh/latest/userguide/mesh-k8s-integration.html)\.
  + For Amazon EC2, make sure that you have setup your Amazon EC2 instance for proxying App Mesh traffic\. For more information, see [Update services](https://docs.aws.amazon.com/app-mesh/latest/userguide/appmesh-getting-started.html#update-services)\.
+ Make sure that the Envoy container that is running on your compute service has successfully connected to the App Mesh Envoy management service\. You can confirm this by checking Envoy stats for the field `control_plane.connected_state`\. For more information on `control_plane.connected_state`, see [Monitor the Envoy Proxy Connectivity](https://docs.aws.amazon.com/app-mesh/latest/userguide/troubleshooting-best-practices.html#ts-bp-enable-envoy-control-plane-connected-state) in our **Troubleshooting Best Practices**\.

  If the Envoy was able to establish the connection initially, but later was disconnected and never reconnected, see [Envoy disconnected from App Mesh Envoy management service with error text](https://docs.aws.amazon.com/app-mesh/latest/userguide/troubleshooting-setup.html#ts-setup-grpc-error-codes) to troubleshoot why it was disconnected\.

If the application connects but the request fails with an `HTTP 503` response code, try the following:
+ Make sure that the virtual service you're connecting to exists in the mesh\.
+ Make sure that the virtual service has a provider \(a virtual router or virtual node\)\.
+ When using Envoy as an HTTP Proxy, if you're seeing egress traffic coming into `cluster.cds_egress_*_mesh-allow-all` instead of the correct destination through Envoy stats, most likely Envoy isn't routing requests properly through `filter_chains`\. This can be a result of using an unqualified virtual service name\. We recommend that you use the service discovery name of the actual service as the virtual service name, because Envoy proxy communicates with other virtual services through their names\.

  For more information, see [virtual services](https://docs.aws.amazon.com/app-mesh/latest/userguide/virtual_services.html)\.
+ Inspect the Envoy proxy logs for any of the following error messages:
  + `No healthy upstream` – The virtual node that the Envoy proxy is attempting to route to does not have any resolved endpoints, or it does not have any healthy endpoints\. Make sure that the target virtual node has the correct service discovery and health check settings\.

    If requests to the service are failing during a deployment or scaling of the backend virtual service, follow the guidance in [Some requests fail with HTTP status code `503` when a virtual service has a virtual node provider](#ts-connectivity-virtual-node-provider)\.
  + `No cluster match for URL` – This is most likely caused when a request is sent to a virtual service that does not match the criteria defined by any of the routes defined under a virtual router provider\. Make sure that the requests from the application are sent to a supported route by ensuring the path and HTTP request headers are correct\.
  + `No matching filter chain found` – This is most likely caused when a request is sent to a virtual service on an invalid port\. Make sure that the requests from the application are using the same port specified on the virtual router\.

If your issue is still not resolved, then consider opening a [GitHub issue](https://github.com/aws/aws-app-mesh-roadmap/issues/new?assignees=&labels=Bug&template=issue--bug-report.md&title=Bug%3A+describe+bug+here) or contact [AWS Support](http://aws.amazon.com/premiumsupport/)\.

## Unable to connect to an external service<a name="ts-connectivity-external-service"></a>

**Symptoms**  
Your application is unable to connect to a service outside of the mesh, such as `amazon.com`\.

**Resolution**  
By default, App Mesh does not allow outbound traffic from applications within the mesh to any destination outside of the mesh\. To enable communication with an external service, there are two options:
+ Set the [outbound filter](https://docs.aws.amazon.com/app-mesh/latest/APIReference/API_EgressFilter.html) on the mesh resource to `ALLOW_ALL`\. This setting will allow any application within the mesh to communicate with any destination IP address inside or outside of the mesh\.
+ Model the external service in the mesh using a virtual service, virtual router, route, and virtual node\. For example, to model the external service `example.com`, you can create a virtual service named `example.com` with a virtual router and route that sends all traffic to a virtual node with a DNS service discovery hostname of `example.com`\.

If your issue is still not resolved, then consider opening a [GitHub issue](https://github.com/aws/aws-app-mesh-roadmap/issues/new?assignees=&labels=Bug&template=issue--bug-report.md&title=Bug%3A+describe+bug+here) or contact [AWS Support](http://aws.amazon.com/premiumsupport/)\.

## Unable to connect to a MySQL or SMTP server<a name="ts-connectivity-troubleshooting-mysql-and-smtp"></a>

**Symptoms**  
When allowing outbound traffic to all destinations \(Mesh `EgressFilter type`=`ALLOW_ALL`\), such as an SMTP server or a MySQL database using a virtual node definition, the connection from your application fails\. As an example, the following is an error message from attempting to connect to a MySQL server\.

```
ERROR 2013 (HY000): Lost connection to MySQL server at 'reading initial communication packet', system error: 0
```

**Resolution**  
This is a known issue that is resolved by using App Mesh image version 1\.15\.0 or later\. For more information, see the [Unable to connect to MySQL with App Mesh](https://github.com/aws/aws-app-mesh-roadmap/issues/62) GitHub issue\. This error occurs because the outbound listener in Envoy configured by App Mesh adds the Envoy TLS Inspector listener filter\. For more information, see [TLS Inspector](https://www.envoyproxy.io/docs/envoy/latest/configuration/listeners/listener_filters/tls_inspector#config-listener-filters-tls-inspector) in the Envoy documentation\. This filter evaluates whether or not a connection is using TLS by inspecting the first packet sent from the client\. With MySQL and SMTP, however, the server sends the first packet after connection\. For more information about MySQL, see [Initial Handshake](https://dev.mysql.com/doc/internals/en/initial-handshake.html) in the MySQL documentation\. Because the server sends the first packet, inspection at the filter fails\.

**To work around this issue depending on your version of Envoy:**
+ If your App Mesh image Envoy version is 1\.15\.0 or later, do not model external services such as **MySQL**, **SMTP**, **MSSQL**, etc\. as a backend for your application's virtual node\.
+ If your App Mesh image Envoy version is prior to 1\.15\.0, add port `3306` to the list of values for the `APPMESH_EGRESS_IGNORED_PORTS` in your services for **MySQL** and as the port you are using for **STMP**\.

**Important**  
While the standard SMTP ports are `25`, `587`, and `465`, you should only add the port you are using to `APPMESH_EGRESS_IGNORED_PORTS` and not all three\.

For more information, see [Update services](https://docs.aws.amazon.com/app-mesh/latest/userguide/getting-started-kubernetes.html#create-update-services) for Kubernetes , [Update services](https://docs.aws.amazon.com/app-mesh/latest/userguide/getting-started-ecs.html#update-services) for Amazon ECS, or [Update services](https://docs.aws.amazon.com/app-mesh/latest/userguide/getting-started-ec2.html#update-services) for Amazon EC2\. 

If your issue is still not resolved, then you can provide us with details on what you're experiencing using the existing [GitHub issue](https://github.com/aws/aws-app-mesh-roadmap/issues/62) or contact [AWS Support](http://aws.amazon.com/premiumsupport/)\.

## Unable to connect to a service modeled as a TCP virtual node or virtual router in App Mesh<a name="ts-connectivity-virtual-node-router"></a>

**Symptoms**  
Your application is unable to connect to a backend that uses the TCP protocol setting in the App Mesh [PortMapping](https://docs.aws.amazon.com/app-mesh/latest/APIReference/API_PortMapping.html) definition\.

**Resolution**  
This is a known issue\. For more information, see [Routing to multiple TCP destinations on the same port](https://github.com/aws/aws-app-mesh-roadmap/issues/195) on GitHub\. App Mesh does not currently allow multiple backend destinations modeled as TCP to share the same port due to restrictions in the information provided to the Envoy proxy at OSI Layer 4\. To make sure that TCP traffic can be routed appropriately for all backend destinations, do the following: 
+ Make sure that all destinations are using a unique port\. If you are using a virtual router provider for the backend virtual service, you can change the virtual router port without changing the port on the virtual nodes that it routes to\. This allows the applications to open connections on the virtual router port while the Envoy proxy continues to use the port defined in the virtual node\.
+ If the destination modeled as TCP is a MySQL server, or any other TCP\-based protocol in which the server sends the first packets after connection, see [Unable to connect to a MySQL or SMTP server](#ts-connectivity-troubleshooting-mysql-and-smtp)\.

If your issue is still not resolved, then you can provide us with details on what you're experiencing using the existing [GitHub issue](https://github.com/aws/aws-app-mesh-roadmap/issues/195) or contact [AWS Support](http://aws.amazon.com/premiumsupport/)\.

## Connectivity succeeds to service not listed as a virtual service backend for a virtual node<a name="ts-connectivity-not-virtual-service"></a>

**Symptoms**  
Your application is able to connect and send traffic to a destination that is not specified as a virtual service backend on your virtual node\.

**Resolution**  
If requests are succeeding to a destination that has not been modeled in the App Mesh APIs, then the most likely cause is that the mesh's [outbound filter](https://docs.aws.amazon.com/app-mesh/latest/APIReference/API_EgressFilter.html) type has been set to `ALLOW_ALL`\. When the outbound filter is set to `ALLOW_ALL`, an outbound request from your application that does not match a modeled destination \(backend\) will be sent to the destination IP address set by the application\. 

If you want to disallow traffic to destinations not modeled in the mesh, consider setting the outbound filter value to `DROP_ALL`\.

**Note**  
Setting the mesh outbound filter value affects all virtual nodes within the mesh\.

If your issue is still not resolved, then consider opening a [GitHub issue](https://github.com/aws/aws-app-mesh-roadmap/issues/new?assignees=&labels=Bug&template=issue--bug-report.md&title=Bug%3A+describe+bug+here) or contact [AWS Support](http://aws.amazon.com/premiumsupport/)\.

## Some requests fail with HTTP status code `503` when a virtual service has a virtual node provider<a name="ts-connectivity-virtual-node-provider"></a>

**Symptoms**  
A portion of your application's requests fail to a virtual service backend that is using a virtual node provider instead of a virtual router provider\. When using a virtual router provider for the virtual service, requests do not fail\.

**Resolution**  
This is a known issue\. For more information, see [Retry policy on Virtual Node provider for a Virtual Service](https://github.com/aws/aws-app-mesh-roadmap/issues/194) on GitHub\. When using a virtual node as a provider for a virtual service, you cannot specify the default retry policy that you want the clients of your virtual service to use\. By comparison, virtual router providers allow retry policies to be specified because they are a property of the child route resources\.

To reduce request failures to virtual node providers, use a virtual router provider instead, and specify a retry policy on its routes\. For other ways to reduce request failures to your applications, see [App Mesh best practices](best-practices.md)\. 

If your issue is still not resolved, then consider opening a [GitHub issue](https://github.com/aws/aws-app-mesh-roadmap/issues/new?assignees=&labels=Bug&template=issue--bug-report.md&title=Bug%3A+describe+bug+here) or contact [AWS Support](http://aws.amazon.com/premiumsupport/)\.

## Unable to connect to an Amazon EFS filesystem<a name="ts-connectivity-efs"></a>

**Symptoms**  
When configuring an Amazon ECS task with an Amazon EFS filesystem as a volume, the task fails to start with the following error\.

```
ResourceInitializationError: failed to invoke EFS utils commands to set up EFS volumes: stderr: mount.nfs4: Connection refused : unsuccessful EFS utils command execution; code: 32
```

**Resolution**  
This is a known issue\. This error occurs because the NFS connection to Amazon EFS occurs before any containers in your task are started\. This traffic is routed by the proxy configuration to Envoy, which will not be running at this point\. Because of the ordering of startup, the NFS client fails to connecting to the Amazon EFS filesystem and the task fails to launch\. To resolve the issue, add port `2049` to the list of values for the `EgressIgnoredPorts` setting in the proxy configuration of your Amazon ECS task definition\. For more information, see [Proxy configuration](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definition_parameters.html#proxyConfiguration)\.

If your issue is still not resolved, then consider opening a [GitHub issue](https://github.com/aws/aws-app-mesh-roadmap/issues/new?assignees=&labels=Bug&template=issue--bug-report.md&title=Bug%3A+describe+bug+here) or contact [AWS Support](http://aws.amazon.com/premiumsupport/)\.

## Connectivity succeeds to service, but the incoming request does not appear in access logs, traces, or metrics for Envoy<a name="ts-connectivity-iptables"></a>

**Symptoms**  
 Even though your application can connect and send requests to another application, you either can not see incoming requests in the access logs or in tracing information for the Envoy proxy\.

**Resolution**  
This is a known issue\. From more information, see [iptables rules setup](https://github.com/aws/aws-app-mesh-roadmap/issues/166) issue on Github\. The Envoy proxy only intercepts inbound traffic to the port of which its corresponding virtual node is listening on\. Requests to any other port will bypass the Envoy proxy and reach to the service behind it directly\. In order to let the Envoy proxy intercept the inbound traffic for your service you need to set your virtual node and service to listen on the same port\.

If your issue is still not resolved, then consider opening a [GitHub issue](https://github.com/aws/aws-app-mesh-roadmap/issues/new?assignees=&labels=Bug&template=issue--bug-report.md&title=Bug%3A+describe+bug+here) or contact [AWS Support](http://aws.amazon.com/premiumsupport/)\.

## Setting the `HTTP_PROXY`/`HTTPS_PROXY` environment variables at container level doesn't work as expected\.<a name="http-https-proxy"></a>

**Symptoms**  
When HTTP\_PROXY/HTTPS\_PROXY is set as an environment variable at the:
+ App container in the task definition with App Mesh enabled, requests being sent to the namespace of the App Mesh services will get `HTTP 500` error responses from the Envoy sidecar\.
+ Envoy container in task definition with App Mesh enabled, requests coming out of Envoy sidecar will not go through the `HTTP`/`HTTPS` proxy server, and the environment variable will not work\.

**Resolution**  
For the app container:

App Mesh functions by having traffic within your task go through the Envoy proxy\. `HTTP_PROXY`/`HTTPS_PROXY` configuration overrides this behavior by configuring container traffic to go through a different external proxy\. The traffic will still be intercepted by Envoy, but it doesn't support proxying the mesh traffic using an external proxy\.

If you want to proxy all non\-mesh traffic, please set `NO_PROXY` to include your mesh's CIDR/namespace, localhost, and the credential's endpoints like in the following example\.

```
NO_PROXY=localhost,127.0.0.1,169.254.169.254,169.254.170.2,10.0.0.0/16
```

For the Envoy container:

Envoy doesn't support a generic proxy\. We do not recommend setting these variables\.

If your issue is still not resolved, then consider opening a [GitHub issue](https://github.com/aws/aws-app-mesh-roadmap/issues/new?assignees=&labels=Bug&template=issue--bug-report.md&title=Bug%3A+describe+bug+here) or contact [AWS Support](http://aws.amazon.com/premiumsupport/)\.

## Upstream request timeouts even after setting the timeout for routes\.<a name="upstream-timeout-request"></a>

**Symptoms**  
You defined the timeout for:
+ The routes, but you are still getting an upstream request timeout error\.
+ The virtual node listener and the timeout and the retry timeout for the routes, but you are still getting an upstream request timeout error\.

**Resolution**  
For the high latency requests greater than 15 seconds to complete successfully, you need to specify a timeout at both the route and virtual node listener level\.

If you specify a route timeout that is greater than the default 15 seconds, make sure that the timeout is also specified for the listener for all participating virtual nodes\. However, if you decrease the timeout to a value that is lower than the default, it's optional to update the timeouts at virtual nodes\. For more information about options when setting up virtual nodes and routes, see [virtual nodes](https://docs.aws.amazon.com/app-mesh/latest/userguide/virtual_nodes.html) and [routes](https://docs.aws.amazon.com/app-mesh/latest/userguide/routes.html)\.

If you specified a **retry policy**, the duration that you specify for the request timeout should always be greater than or equal to the *retry timeout* multiplied by the *max retries* that you defined in the **retry policy**\. This allows your request with all the retries to complete successfully\. For more information, see [routes](https://docs.aws.amazon.com/app-mesh/latest/userguide/routes.html)\.

If your issue is still not resolved, then consider opening a [GitHub issue](https://github.com/aws/aws-app-mesh-roadmap/issues/new?assignees=&labels=Bug&template=issue--bug-report.md&title=Bug%3A+describe+bug+here) or contact [AWS Support](http://aws.amazon.com/premiumsupport/)\.

## Envoy responds with HTTP Bad request\.<a name="ts-http-bad-request"></a>

**Symptoms**  
Envoy responds with **HTTP 400 Bad request** for all requests sent through the Network Load Balancer \(NLB\)\. When we check the Envoy logs, we see:
+ Debug logs:

  ```
  dispatch error: http/1.1 protocol error: HPE_INVALID_METHOD
  ```
+ Access logs:

  ```
  "- - HTTP/1.1" 400 DPE 0 11 0 - "-" "-" "-" "-" "-"
  ```

**Resolution**  
The resolution is to disable the proxy protocol version 2 \(PPv2\) on your NLB's [target group attributes](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/load-balancer-target-groups.html#target-group-attributes)\.

As of today the PPv2 is not supported by virtual gateway and virtual node Envoy that are run using the App Mesh control plane\. If you deploy NLB using AWS load balancer controller on Kubernetes, then disable PPv2 by setting the following attribute to `false`:

```
service.beta.kubernetes.io/aws-load-balancer-target-group-attributes: proxy_protocol_v2.enabled
```

See [AWS Load Balancer Controller Annotations](https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.4/guide/service/annotations/#resource-attributestrue) for more details about NLB resource attributes\.

If your issue is still not resolved, then consider opening a [GitHub issue](https://github.com/aws/aws-app-mesh-roadmap/issues/new?assignees=&labels=Bug&template=issue--bug-report.md&title=Bug%3A+describe+bug+here) or contact [AWS Support](http://aws.amazon.com/premiumsupport/)\.
# App Mesh connectivity troubleshooting<a name="troubleshoot-connectivity"></a>

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
+ Make sure that the Envoy container running on your compute service has successfully connected to the App Mesh Envoy management service\. For more information, see [Envoy disconnected from App Mesh Envoy management service with error text](troubleshooting-setup.md#ts-setup-grpc-error-codes)\. If the application connects but the request fails with an `HTTP 503` response code, try the following:
  + Make sure that the virtual service you're connecting to exists in the mesh\.
  + Make sure that the virtual service has a provider \(a virtual router or virtual node\)\.
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
By default, App Mesh does not allow egress traffic from applications within the mesh to any destination outside of the mesh\. To enable communication with an external service, there are two options:
+ Set the [egress filter](https://docs.aws.amazon.com/app-mesh/latest/APIReference/API_EgressFilter.html) on the mesh resource to `ALLOW_ALL`\. This setting will allow any application within the mesh to communicate with any destination IP address inside or outside of the mesh\.
+ Model the external service in the mesh using a virtual service, virtual router, route, and virtual node\. For example, to model the external service `example.com`, you can create a virtual service named `example.com` with a virtual router and route that sends all traffic to a virtual node with a DNS service discovery hostname of `example.com`\.

If your issue is still not resolved, then consider opening a [GitHub issue](https://github.com/aws/aws-app-mesh-roadmap/issues/new?assignees=&labels=Bug&template=issue--bug-report.md&title=Bug%3A+describe+bug+here) or contact [AWS Support](http://aws.amazon.com/premiumsupport/)\.

## Unable to connect to a MySQL server<a name="ts-connectivity-troubleshooting-mysql"></a>

**Symptoms**  
When allowing egress traffic to all destinations \(Mesh `EgressFilter type`=`ALLOW_ALL`\) or specifically to a MySQL database using a virtual node definition, the connection from your application fails to MySQL with the following error message\.

```
ERROR 2013 (HY000): Lost connection to MySQL server at 'reading initial communication packet', system error: 0
```

**Resolution**  
This is a known issue\. For more information, see the [Unable to connect to MySQL with App Mesh](https://github.com/aws/aws-app-mesh-roadmap/issues/62) GitHub issue\. This error occurs because the egress listener in Envoy configured by App Mesh adds the Envoy TLS Inspector listener filter\. For more information, see [TLS Inspector](https://www.envoyproxy.io/docs/envoy/latest/configuration/listeners/listener_filters/tls_inspector#config-listener-filters-tls-inspector) in the Envoy documentation\. This filter evaluates whether or not a connection is using TLS by inspecting the first packet sent from the client\. With MySQL however, the server sends the first packet after connection\. For more information, see [Initial Handshake](https://dev.mysql.com/doc/internals/en/initial-handshake.html) in the MySQL documentation\. Because the server sends the first packet, inspection at the filter fails\.

To work around this issue, add port `3306` to the list of values for the `APPMESH_EGRESS_IGNORED_PORTS` in your services\. For more information, see [Update services](https://docs.aws.amazon.com/eks/latest/userguide/appmesh-getting-started.html#update-services) for Kubernetes , [Update services](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/appmesh-getting-started.html#update-services) for Amazon ECS, or [Update services](https://docs.aws.amazon.com/app-mesh/latest/userguide/appmesh-getting-started.html#update-services) for Amazon EC2\. 

If your issue is still not resolved, then you can provide us with details on what you're experiencing using the existing [GitHub issue](https://github.com/aws/aws-app-mesh-roadmap/issues/62) or contact [AWS Support](http://aws.amazon.com/premiumsupport/)\.

## Unable to connect to a service modeled as a TCP virtual node or virtual router in App Mesh<a name="ts-connectivity-virtual-node-router"></a>

**Symptoms**  
Your application is unable to connect to a backend that uses the TCP protocol setting in the App Mesh [PortMapping](https://docs.aws.amazon.com/app-mesh/latest/APIReference/API_PortMapping.html) definition\.

**Resolution**  
This is a known issue\. For more information, see [Routing to multiple TCP destinations on the same port](https://github.com/aws/aws-app-mesh-roadmap/issues/195) on GitHub\. App Mesh does not currently allow multiple backend destinations modeled as TCP to share the same port due to restrictions in the information provided to the Envoy proxy at OSI Layer 4\. To make sure that TCP traffic can be routed appropriately for all backend destinations, do the following: 
+ Make sure that all destinations are using a unique port\. If you are using a virtual router provider for the backend virtual service, you can change the virtual router port without changing the port on the virtual nodes that it routes to\. This allows the applications to open connections on the virtual router port while the Envoy proxy continues to use the port defined in the virtual node\.
+ If the destination modeled as TCP is a MySQL server, or any other TCP\-based protocol in which the server sends the first packets after connection, see [Unable to connect to a MySQL server](#ts-connectivity-troubleshooting-mysql)\.

If your issue is still not resolved, then you can provide us with details on what you're experiencing using the existing [GitHub issue](https://github.com/aws/aws-app-mesh-roadmap/issues/195) or contact [AWS Support](http://aws.amazon.com/premiumsupport/)\.

## Connectivity succeeds to service not listed as a virtual service backend for a virtual node<a name="ts-connectivity-not-virtual-service"></a>

**Symptoms**  
Your application is able to connect and send traffic to a destination that is not specified as a virtual service backend on your virtual node\.

**Resolution**  
If requests are succeeding to a destination that has not been modeled in the App Mesh APIs, then the most likely cause is that the mesh's [egress filter](https://docs.aws.amazon.com/app-mesh/latest/APIReference/API_EgressFilter.html) type has been set to `ALLOW_ALL`\. When the egress filter is set to `ALLOW_ALL`, an outbound request from your application that does not match a modeled destination \(backend\) will be sent to the destination IP address set by the application\. 

If you want to disallow traffic to destinations not modeled in the mesh, consider setting the egress filter value to `DROP_ALL`\.

**Note**  
Setting the mesh egress filter value affects all virtual nodes within the mesh\.

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
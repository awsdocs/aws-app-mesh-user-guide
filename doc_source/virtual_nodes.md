# Virtual nodes<a name="virtual_nodes"></a>

A virtual node acts as a logical pointer to a particular task group, such as an Amazon ECS service or a Kubernetes deployment\. When you create a virtual node, you must specify a service discovery method for your task group\. Any inbound traffic that your virtual node expects is specified as a *listener*\. Any virtual service that a virtual node sends outbound traffic to is specified as a *backend*\.

The response metadata for your new virtual node contains the Amazon Resource Name \(ARN\) that is associated with the virtual node\. Set this value \(either the full ARN or the truncated resource name\) as the `APPMESH_VIRTUAL_NODE_NAME` environment variable for your task group's Envoy proxy container in your Amazon ECS task definition or Kubernetes pod spec\. For example, the value could be `mesh/default/virtualNode/simpleapp`\. This is then mapped to the `node.id` and `node.cluster` Envoy parameters\.

**Note**  
If you require your Envoy stats or tracing to use a different name, you can override the `node.cluster` value that is set by `APPMESH_VIRTUAL_NODE_NAME` with the `APPMESH_VIRTUAL_NODE_CLUSTER` environment variable\.

## Creating a virtual node<a name="vn-create-virtual-node"></a>

 To create a virtual node using the AWS CLI version 1\.18\.116 or higher, see the example in the AWS CLI reference for the [create\-virtual\-node](https://docs.aws.amazon.com/cli/latest/reference/appmesh/create-virtual-node.html) command\.

**To create a virtual node using the AWS Management Console**

1. Open the App Mesh console at [https://console\.aws\.amazon\.com/appmesh/](https://console.aws.amazon.com/appmesh/)\. 

1. Choose the mesh that you want to create the virtual node in\. All of the meshes that you own and that have been [shared](sharing.md) with you are listed\.

1. Choose **Virtual nodes** in the left navigation\.

1. Choose **Create virtual node**\.

1. Specify settings for your virtual node\.

   1. **Virtual node configuration**
      + For **Virtual node name**, enter a name for your virtual node\.
      + For **Service discovery method**, choose one of the following options:
        + **DNS** – Specify the **DNS hostname** of the actual service that the virtual node represents\. The Envoy proxy is deployed in an Amazon VPC\. The proxy sends name resolution requests to the DNS server that is configured for the VPC\. If the hostname resolves, the DNS server returns one or more IP addresses\. For more information about VPC DNS settings, see [Using DNS with your VPC](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-dns.html)\. If the DNS server returns multiple IP addresses, then the Envoy proxy chooses one of the addresses using the [Logical DNS](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/upstream/service_discovery#arch-overview-service-discovery-types-logical-dns) service discovery type\.
        + **AWS Cloud Map** – Specify an existing **Service name** and **Namespace**\. Optionally, you can also specify attributes that App Mesh can query AWS Cloud Map for by selecting **Add row** and specifying a **Key** and **Value**\. Only instances that match all of the specified key/value pairs will be returned\. To use AWS Cloud Map, your account must have the `AWSServiceRoleForAppMesh` [service\-linked role](using-service-linked-roles.md)\. For more information about AWS Cloud Map, see the [AWS Cloud Map Developer Guide](https://docs.aws.amazon.com/cloud-map/latest/dg/)\.
        + **None** – Select if your virtual node doesn't expect any inbound traffic\.
      + \(Optional\) **Client policy defaults** – Configure default requirements when communicating to backend virtual services\.
**Note**  
If you want to enable Transport Layer Security \(TLS\) for an existing virtual node, then we recommend that you create a new virtual node, which represents the same service as the existing virtual node, on which to enable TLS\. Then gradually shift traffic to the new virtual node using a virtual router and route\. For more information about creating a route and adjusting weights for the transition, see [Routes](routes.md)\. If you update an existing, traffic\-serving virtual node with TLS, there is a chance that the downstream client Envoy proxies will receive TLS validation context before the Envoy proxy for the virtual node that you have updated receives the certificate\. This can cause TLS negotiation errors on the downstream Envoy proxies\.
[Proxy authorization](proxy-authorization.md) must be enabled for the Envoy proxy deployed with the application represented by the backend service's virtual nodes\. We recommend that when you enable proxy authorization, you restrict access to only the virtual nodes that this virtual node is communicating with\.
        + \(Optional\) Select **Enforce TLS** if you want to require the virtual node to communicate with all backends using Transport Layer Security \(TLS\)\.
        + \(Optional\) If you only want to require the use of TLS for one or more specific ports, then enter a number in **Ports**\. To add additional ports, select **Add port**\. If you don't specify any ports, TLS is enforced for all ports\.
        + For **Certificate discovery method**, select one of the following options\. The certificate that you specify must already exist and meet specific requirements\. For more information, see [Certificate requirements](tls.md#virtual-node-tls-prerequisites)\.
          + **AWS Certificate Manager Private Certificate Authority** hosting – Select one or more existing **Certificates**\. For a complete, end\-to\-end walk through of deploying a mesh with a sample application using encryption with an ACM certificate, see [Configuring TLS with AWS Certificate Manager](https://github.com/aws/aws-app-mesh-examples/tree/master/walkthroughs/tls-with-acm) on GitHub\.
          + **Local file hosting** – Specify the path to the **Certificate chain** file on the file system where the Envoy is deployed\. For a complete, end\-to\-end walk through of deploying a mesh with a sample application using encryption with local files, see [Configuring TLS with File Provided TLS Certificates](https://github.com/aws/aws-app-mesh-examples/tree/master/walkthroughs/howto-tls-file-provided) on GitHub\.
      + \(Optional\) **Service backends** – Specify the App Mesh virtual service that the virtual node will communicate with\.
        + Enter an App Mesh virtual service name or full Amazon Resource Name \(ARN\) for the virtual service that your virtual node communicates with\.
        + \(Optional\) If you set **Client policy defaults**, but want to override them with unique TLS settings for a backend, select **TLS settings** and then select **Override defaults**\.
          + \(Optional\) Select **Enforce TLS** if you want to require the virtual node to communicate with all backends using TLS\.
          + \(Optional\) If you only want to require the use of TLS for one or more specific ports, then enter a number in **Ports**\. To add additional ports, select **Add port**\. If you don't specify any ports, TLS is enforced for all ports\.
          + For **Certificate discovery method**, select one of the following options\. The certificate that you specify must already exist and meet specific requirements\. For more information, see [Certificate requirements](tls.md#virtual-node-tls-prerequisites)\.
            + **AWS Certificate Manager Private Certificate Authority** hosting – Select one or more existing **Certificates**\.
            + **Local file hosting** – Specify the path to the **Certificate chain** file on the file system where the Envoy is deployed\.

        To add additional backends, select **Add backend**\.
      + \(Optional\) **Logging**

        To configure logging, enter the HTTP access logs path that you want Envoy to use\. We recommend the `/dev/stdout` path so that you can use Docker log drivers to export your Envoy logs to a service such as Amazon CloudWatch Logs\.
**Note**  
Logs must still be ingested by an agent in your application and sent to a destination\. This file path only instructs Envoy where to send the logs\. 

   1. **Listener configuration**
      + If your virtual node expects ingress traffic, specify a **Port** and **Protocol** for the **Listener**\. The **http** listener permits connection transition to websockets\.
      + \(Optional\) **Enable health check** – Configure settings for a health check policy\.

        A health check policy is optional, but if you specify any values for a health policy, then you must specify values for **Healthy threshold**, **Health check interval**, **Health check protocol**, **Timeout period**, and **Unhealthy threshold**\.
        + For **Health check protocol**, choose a protocol\. If you select **grpc**, then your service must conform to the [GRPC Health Checking Protocol](https://github.com/grpc/grpc/blob/master/doc/health-checking.md)\.
        + For **Health check port**, specify the port that the health check should run on\.
        + For **Healthy threshold**, specify the number of consecutive successful health checks that must occur before declaring the listener healthy\.
        + For **Health check interval**, specify the time period in milliseconds between each health check execution\.
        + For **Path**, specify the destination path for the health check request\. This value is only used if the **Health check protocol** is `http` or `http2`\. The value is ignored for other protocols\.
        + For **Timeout period**, specify the amount of time to wait when receiving a response from the health check, in milliseconds\.
        + For **Unhealthy threshold**, specify the number of consecutive failed health checks that must occur before declaring the listener unhealthy\.
      + \(Optional\) **Enable TLS termination** – Configure how other virtual nodes communicate with this virtual node using TLS\.
        + For **Mode**, select the mode you want TLS to be configured for on the listener\.
        + For **Certificate method**, select one of the following options\. The certificate must meet specific requirements\. For more information, see [Certificate requirements](tls.md#virtual-node-tls-prerequisites)\.
          + **AWS Certificate Manager hosting** – Select an existing **Certificate**\.
          + **Local file hosting** – Specify the path to the **Certificate chain** and **Private key** files on the file system where the Envoy proxy is deployed\.
      + \(Optional\) **Timeouts**\.
        + **Request timeout** – You can specify a request timeout if you selected **grpc**, **http**, or **http2** for the listener's **Protocol**\. The default is 15 seconds\. If increasing the timeout, make sure that the timeout specified for any route that is used for a **Service backend** is also greater than 15 seconds\. For more information, see [Routes](routes.md)\.
        + **Idle duration** – You can specify an idle duration for any listener protocol\.

1. Choose **Create virtual node** to finish\.

## App Mesh Preview Channel only – Connection pooling<a name="virtual-node-connection-pooling"></a>

Connection pooling limits the number of connections that an Envoy can concurrently establish with all the hosts in the upstream cluster\. Connection pooling is supported at the virtual node listener\. It is intended to protect your local application from being overwhelmed with connections and lets you adjust traffic shaping for the needs of your applications\.

You can configure destination\-side connection pool settings for a virtual node listener\. App Mesh by sets the client\-side connection pool settings to infinite by default, simplifying mesh configuration\.

**Prerequisites**
+ An existing App Mesh [service mesh](meshes.md#create-mesh)\.
+ An existing App Mesh [virtual service](virtual_services.md#create-virtual-service) that has an existing App Mesh [virtual node](#vn-create-virtual-node) or App Mesh [virtual router](virtual_routers.md#create-virtual-router) provider\.

**To create a virtual node with connection pooling**

1. Add the Preview Channel service model to the AWS CLI version 1 with the following command\.

   ```
   aws configure add-model \
       --service-name appmesh-preview \
       --service-model https://raw.githubusercontent.com/aws/aws-app-mesh-roadmap/master/appmesh-preview/service-model.json
   ```

   If you're using the AWS CLI version 2, add the service model with the following commands:

   ```
   curl -o service-model.json https://raw.githubusercontent.com/aws/aws-app-mesh-roadmap/master/appmesh-preview/service-model.json
   aws configure add-model --service-name appmesh-preview --service-model file://service-model.json
   ```

1. Create a JSON file named `virtual-node.json` with a virtual node configuration\. In the following configuration, the `http` listener has custom values for `maxPendingRequests` and `maxConnections`\.

   ```
   {
      "meshName" : "my-mesh",
      "spec" : {
         "backends" : [
            {
               "virtualService" : {
                  "virtualServiceName" : "my-virtual-service-b.svc.cluster.local"
               }
            }
         ],
         "listeners" : [
            {
               "connectionPool" : {
                  "http" : {
                     "maxPendingRequests" : 1500,
                     "maxConnections" : 2000
                  }
               },
               "portMapping" : {
                  "port" : 80,
                  "protocol" : "http"
               }
            }
         ],
         "serviceDiscovery" : {
            "dns" : {
               "hostname" : "my-virtual-service-a.svc.cluster.local"
            }
         }
      },
      "virtualNodeName" : "my-virtual-node-a"
   }
   ```
**Note**  
If your listener is `tcp`, specify `maxConnections` only\. If your listener protocol is `grpc` or `http2`, specify `maxRequests` only\. The `connectionPool` and `portMapping` protocols must be the same\.

   You can specify values for the following settings in the previous example:
   + `maxConnections` – Represents the maximum number of outbound TCP connections the Envoy can establish concurrently with all the hosts in the upstream cluster\. This parameter is used for HTTP/1\.1 connections\.
   + `maxPendingRequests` – Represents the number of overflowing requests after `max_connections` that an Envoy will queue to an upstream cluster\. This parameter is used for HTTP/1\.1 connections\.
   + `maxRequests` – Represents the maximum number of inflight requests that an Envoy can concurrently support across all the hosts in the upstream cluster\. This parameter is used for controlling HTTP/2\.0 connections\.

1. Create the virtual node with the following command\.

   ```
   aws appmesh-preview create-virtual-node --cli-input-json file://virtual-node.json
   ```

## App Mesh Preview Channel only – Outlier detection<a name="create-virtual-node-outlier-detection"></a>

While existing active health checks protect your applications from sending traffic to hosts that are not deemed ready, they are not a complete solution\. Specifically, active health checks:
+ Quickly hit cardinality limits as you scale up\. Every client checks the health of every known destination\. In a fully connected mesh, this can lead to a significant number of health checks in a mesh\.
+ Will be a lagging indicator of emergent failure in a mesh, especially because you must balance health check call volume against health check frequency\.

Adding outlier detection that is applied at the client Envoy allows clients to take near\-immediate action on connections where it has observed known bad failures\. Outlier detection also mitigates cardinality concerns, as Envoy is passively monitoring request traffic, rather than injecting additional health check traffic\.

**Outlier detection:**
+ At client Envoys is a form of a circuit breaker implementation that tracks the health status of each individual host in the upstream service\. It automatically ejects endpoints that demonstrate abnormal behavior from the connection pool for a specified period of time, so that traffic is not routed to ill\-behaving endpoints\. Circuit breaking is a pattern for creating resilient applications that allow you to write applications that limit the impact of failures, latency spikes, and network irregularities\. Using a circuit breaker pattern enables fast failure, rather than clients trying to connect to an overloaded or failing host\.
+ Is a form of passive health checking\. It dynamically determines whether endpoints in an upstream cluster are performing unlike the others and removes them from the healthy load balancing set\. Envoy also supports active health checking, that probes upstream endpoints via HTTP requests, Layer 3 and 4 echo capabilities or the Redis `PING` command\. 

Passive and active health checking can be enabled together or independently, and form the basis for an upstream health checking solution\. Tcp and gRPC errors are internally mapped to http `5xx` codes\.

**Prerequisites**
+ An existing App Mesh [service mesh](meshes.md#create-mesh)\.
+ An existing App Mesh [virtual service](virtual_services.md#create-virtual-service) that has an existing App Mesh [virtual node](#vn-create-virtual-node) or App Mesh [virtual router](virtual_routers.md#create-virtual-router) provider\.

**To create a virtual node with outlier detection**

1. Add the Preview Channel service model to the AWS CLI version 1 with the following command\.

   ```
   aws configure add-model \
       --service-name appmesh-preview \
       --service-model https://raw.githubusercontent.com/aws/aws-app-mesh-roadmap/master/appmesh-preview/service-model.json
   ```

   If you're using the AWS CLI version 2, add the service model with the following commands:

   ```
   curl -o service-model.json https://raw.githubusercontent.com/aws/aws-app-mesh-roadmap/master/appmesh-preview/service-model.json
   aws configure add-model --service-name appmesh-preview --service-model file://service-model.json
   ```

1. Create a JSON file named `virtual-node.json` with a virtual node configuration\.

   ```
   {
      "meshName" : "my-mesh",
      "spec" : {
         "backends" : [
            {
               "virtualService" : {
                  "virtualServiceName" : "my-virtual-service-b.svc.cluster.local"
               }
            }
         ],
         "listeners" : [
            {
               "outlierDetection" : {
                  "baseEjectionDuration" : {
                     "unit" : "s",
                     "value" : 20
                  },
                  "interval" : {
                     "unit" : "s",
                     "value" : 10
                  },
                  "maxEjectionPercent" : 25,
                  "maxServerErrors" : 7
               },
               "portMapping" : {
                  "port" : 80,
                  "protocol" : "http"
               }
            }
         ],
         "serviceDiscovery" : {
            "cloudmap" : {
               "namespaceName": "svc.cluster.local"
               "serviceName" : "my-virtual-service-a"
            }
         }
      },
      "virtualNodeName" : "my-virtual-node-a"
   }
   ```

   App Mesh supports the following parameters:
   + `maxServerErrors` – The number of consecutive `5xx` errors required for ejection\.  This detection type takes into account both locally externally originated type of errors\. Errors generated by tcp proxy are internally mapped to HTTP `5xx` codes and treated as such\. If an upstream host returns some number of errors which are treated as consecutive `5xx` type errors, it will be ejected\.  
   + `interval` – The time interval between ejection sweep analysis\.
   + `baseEjectionDuration` – The base time that a host is ejected for\. The real time is equal to the base time multiplied by the number of times the host has been ejected, so that Envoys automatically increase the ejection period for unhealthy upstream servers\. 

     Once a host is ejected, it will be returned to the pool after the ejection duration has elapsed\. Envoys automatically increase the ejection period for unhealthy upstream servers\.
   + `maxEjectionPercent` – The maximum percentage of hosts in the load balancing pool for the upstream service that can be ejected\. Will eject at least one host regardless of the value\.

1. Create the virtual node with the following command\.

   ```
   aws appmesh-preview create-virtual-node --cli-input-json file://virtual-node.json
   ```

## Deleting a virtual node<a name="delete-virtual-node"></a>

To delete a virtual node using the AWS CLI, use the `aws appmesh delete-virtual-node` command\. For an example of deleting a virtual node using the AWS CLI, see [delete\-virtual\-node](https://docs.aws.amazon.com/cli/latest/reference/appmesh/delete-virtual-node.html)\.

**Note**  
You can't delete a virtual node if it is specified as a target in any [route](routes.md) or as a provider in any [virtual service](virtual_services.md)\.

**To delete a virtual node using the AWS Management Console**

1. Open the App Mesh console at [https://console\.aws\.amazon\.com/appmesh/](https://console.aws.amazon.com/appmesh/)\. 

1. Choose the mesh that you want to delete a virtual node from\. All of the meshes that you own and that have been [shared](sharing.md) with you are listed\.

1. Choose **Virtual nodes** in the left navigation\.

1. In the **Virtual Nodes** table, choose the virtual node that you want to delete and select **Delete**\. To delete a virtual node, your account ID must be listed in either the **Mesh owner** or the **Resource owner** columns of the virtual node\.

1. In the confirmation box, type **delete** and then select **Delete**\.
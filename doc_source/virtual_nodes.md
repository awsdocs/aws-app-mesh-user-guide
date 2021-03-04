# Virtual nodes<a name="virtual_nodes"></a>

A virtual node acts as a logical pointer to a particular task group, such as an Amazon ECS service or a Kubernetes deployment\. When you create a virtual node, you must specify a service discovery method for your task group\. Any inbound traffic that your virtual node expects is specified as a *listener*\. Any virtual service that a virtual node sends outbound traffic to is specified as a *backend*\.

The response metadata for your new virtual node contains the Amazon Resource Name \(ARN\) that is associated with the virtual node\. Set this value as the `APPMESH_RESOURCE_ARN` environment variable for your task group's Envoy proxy container in your Amazon ECS task definition or Kubernetes pod spec\. For example, the value could be `arn:aws:appmesh:us-west-2:111122223333:mesh/myMesh/virtualNode/myVirtualNode`\. This is then mapped to the `node.id` and `node.cluster` Envoy parameters\. You must be using `1.15.0` or later of the Envoy image when setting this variable\. For more information about App Mesh Envoy variables, see [Envoy image](envoy.md)\.

**Note**  
By default, App Mesh uses the name of the resource you specified in `APPMESH_RESOURCE_ARN` when Envoy is referring to itself in metrics and traces\. You can override this behavior by setting the `APPMESH_RESOURCE_CLUSTER` environment variable with your own name\.

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
        + For **Validation method**, select one of the following options\. The certificate that you specify must already exist and meet specific requirements\. For more information, see [Certificate requirements](tls.md#virtual-node-tls-prerequisites)\.
          + **AWS Certificate Manager Private Certificate Authority** hosting – Select one or more existing **Certificates**\. For a complete, end\-to\-end walk through of deploying a mesh with a sample application using encryption with an ACM certificate, see [Configuring TLS with AWS Certificate Manager](https://github.com/aws/aws-app-mesh-examples/tree/master/walkthroughs/tls-with-acm) on GitHub\.
          + **Envoy Secret Discovery Service \(SDS\)** hosting – Enter the name of the secret Envoy will fetch using the Secret Discovery Service\.
          + **Local file hosting** – Specify the path to the **Certificate chain** file on the file system where Envoy is deployed\. For a complete, end\-to\-end walk through of deploying a mesh with a sample application using encryption with local files, see [Configuring TLS with File Provided TLS Certificates](https://github.com/aws/aws-app-mesh-examples/tree/master/walkthroughs/howto-tls-file-provided) on GitHub\.
        + \(Optional\) Enter a **Subject Alternative Name**\. To add additional SANs, select **Add SAN**\. SANs must be FQDN or URI formatted\.
        + \(Optional\) Select **Provide client certificate** and one of the options below to provide a client certificate when a server requests it and enable mutual TLS authentication \. To learn more about mutual TLS, see the App Mesh [Mutual TLS Authentication](https://docs.aws.amazon.com/app-mesh/latest/userguide/mutual-tls.html) docs\.
          + **Envoy Secret Discovery Service \(SDS\)** hosting – Enter the name of the secret Envoy will fetch using the Secret Discovery Service\.
          + **Local file hosting** – Specify the path to the **Certificate chain** file, as well as the **Private key**, on the file system where Envoy is deployed\.
      + \(Optional\) **Service backends** – Specify the App Mesh virtual service that the virtual node will communicate with\.
        + Enter an App Mesh virtual service name or full Amazon Resource Name \(ARN\) for the virtual service that your virtual node communicates with\.
        + \(Optional\) If you want to set unique TLS settings for a backend, select **TLS settings** and then select **Override defaults**\.
          + \(Optional\) Select **Enforce TLS** if you want to require the virtual node to communicate with all backends using TLS\.
          + \(Optional\) If you only want to require the use of TLS for one or more specific ports, then enter a number in **Ports**\. To add additional ports, select **Add port**\. If you don't specify any ports, TLS is enforced for all ports\.
          + For **Validation method**, select one of the following options\. The certificate that you specify must already exist and meet specific requirements\. For more information, see [Certificate requirements](tls.md#virtual-node-tls-prerequisites)\.
            + **AWS Certificate Manager Private Certificate Authority** hosting – Select one or more existing **Certificates**\.
            + **Envoy Secret Discovery Service \(SDS\)** hosting – Enter the name of the secret Envoy will fetch using the Secret Discovery Service\.
            + **Local file hosting** – Specify the path to the **Certificate chain** file on the file system where Envoy is deployed\.
          + \(Optional\) Enter a **Subject Alternative Name**\. To add additional SANs, select **Add SAN**\. SANs must be FQDN or URI formatted\.
          + \(Optional\) Select **Provide client certificate** and one of the options below to provide a client certificate when a server requests it and enable mutual TLS authentication\. To learn more about mutual TLS, see the App Mesh [Mutual TLS Authentication](https://docs.aws.amazon.com/app-mesh/latest/userguide/mutual-tls.html) docs\.
            + **Envoy Secret Discovery Service \(SDS\)** hosting – Enter the name of the secret Envoy will fetch using the Secret Discovery Service\.
            + **Local file hosting** – Specify the path to the **Certificate chain** file, as well as the **Private key**, on the file system where Envoy is deployed\.

        To add additional backends, select **Add backend**\.
      + \(Optional\) **Logging**

        To configure logging, enter the HTTP access logs path that you want Envoy to use\. We recommend the `/dev/stdout` path so that you can use Docker log drivers to export your Envoy logs to a service such as Amazon CloudWatch Logs\.
**Note**  
Logs must still be ingested by an agent in your application and sent to a destination\. This file path only instructs Envoy where to send the logs\. 

   1. **Listener configuration**
      + If your virtual node expects inbound traffic, specify a **Port** and **Protocol** for the **Listener**\. The **http** listener permits connection transition to websockets\.
      + \(Optional\) **Enable connection pool** 

        Connection pooling limits the number of connections that an Envoy can concurrently establish with all the hosts in the upstream cluster\. It is intended to protect your local application from being overwhelmed with connections and lets you adjust traffic shaping for the needs of your applications\.

        You can configure destination\-side connection pool settings for a virtual node listener\. App Mesh sets the client\-side connection pool settings to infinite by default, simplifying mesh configuration\.
**Note**  
The connectionPool and portMapping protocols must be the same\. If your listener protocol is tcp, specify maxConnections only\. If your listener protocol is grpc or http2, specify maxRequests only\. If your listener protocol is http, you can specify both maxConnections and maxPendingRequests\. 
        + For **Maximum connections**, specify the maximum number of outbound connections\.
        + For **Maximum requests**, specify maximum number of parallel requests that can occur to the upstream cluster\.
        + \(Optional\) For **Maximum pending requests**, specify the number of overflowing requests after **Maximum connections** that an Envoy will queue\. The default value is `2147483647`\.
      + \(Optional\) **Enable outlier detection **

        Outlier detection applied at the client Envoy allows clients to take near\-immediate action on connections with observed known bad failures\. It is a form of a circuit breaker implementation that tracks the health status of individual hosts in the upstream service\.

        Outlier detection dynamically determines whether endpoints in an upstream cluster are performing unlike the others and removes them from the healthy load balancing set\.
**Note**  
To effectively configure outlier detection for a server Virtual Node, the service discovery method of that Virtual Node should use AWS Cloud Map\. Otherwise if using DNS, the Envoy proxy would only elect a single IP address for routing to the upstream service, nullifying the outlier detection behavior of ejecting an unhealthy host from a set of hosts\. Refer to the Service discovery method section for more details on the Envoy proxy's behavior in relation to the service discovery type\. 
        + For **Server errors**, specify the number of consecutive 5xx errors required for ejection\.
        + For **Outlier detection interval**, specify the time interval and unit between ejection sweep analysis\.
        + For **Base ejection duration**, specify the base amount of time and unit for which a host is ejected\.
        + For **Ejection percentage**, specify the maximum percentage of hosts in the load balancing pool that can be ejected\.
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
          + **Envoy Secret Discovery Service \(SDS\)** hosting – Enter the name of the secret Envoy will fetch using the Secret Discovery Service\.
          + **Local file hosting** – Specify the path to the **Certificate chain** file, as well as the **Private key**, on the file system where the Envoy proxy is deployed\.
        + \(Optional\) Select **Require client certificates** and one of the options below to enable mutual TLS authentication when a client provides a certificate\. To learn more about mutual TLS, see the App Mesh [Mutual TLS Authentication](https://docs.aws.amazon.com/app-mesh/latest/userguide/mutual-tls.html) docs\.
          + **Envoy Secret Discovery Service \(SDS\)** hosting – Enter the name of the secret Envoy will fetch using the Secret Discovery Service\.
          + **Local file hosting** – Specify the path to the **Certificate chain** file on the file system where Envoy is deployed\.
        + \(Optional\) Enter a **Subject Alternative Name**\. To add additional SANs, select **Add SAN**\. SANs must be FQDN or URI formatted\.
      + \(Optional\) **Timeouts**\.
        + **Request timeout** – You can specify a request timeout if you selected **grpc**, **http**, or **http2** for the listener's **Protocol**\. The default is 15 seconds\.
        + **Idle duration** – You can specify an idle duration for any listener protocol\. The default is 300 seconds\.
**Note**  
 If you specify a timeout greater than the default, make sure to set up a virtual router and a route with a timeout greater than the default\. However, if you decrease the timeout to a value that is lower than the default, it's optional to update the timeouts at Route\. For more information, see [Routes](https://docs.aws.amazon.com/app-mesh/latest/userguide/routes.html)\.

1. Choose **Create virtual node** to finish\.

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
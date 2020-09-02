# Virtual gateways<a name="virtual_gateways"></a>

A virtual gateway allows resources that are outside of your mesh to communicate to resources that are inside of your mesh\. The virtual gateway represents an Envoy proxy running in an Amazon ECS service, in a Kubernetes service, or on an Amazon EC2 instance\. Unlike a virtual node, which represents Envoy running with an application, a virtual gateway represents Envoy deployed by itself\. 

External resources must be able to resolve a DNS name to an IP address assigned to the service or instance that runs Envoy\. Envoy can then access all of the App Mesh configuration for resources that are inside of the mesh\. To complete an end\-to\-end walkthrough, see [Configuring Ingress Gateway](https://github.com/aws/aws-app-mesh-examples/tree/master/walkthroughs/howto-ingress-gateway)\.

## Creating a virtual gateway<a name="create-virtual-gateway"></a>

**To create a virtual gateway using the AWS Management Console**

 To create a virtual gateway using the AWS CLI version 1\.18\.116 or higher, see the AWS CLI reference for the [create\-virtual\-gateway](https://docs.aws.amazon.com/cli/latest/reference/appmesh/create-virtual-gateway.html) command\.

1. Open the App Mesh console at [https://console\.aws\.amazon\.com/appmesh/](https://console.aws.amazon.com/appmesh/)\. 

1. Choose the mesh that you want to create the virtual gateway in\. All of the meshes that you own and that have been [shared](sharing.md) with you are listed\.

1. Choose **Virtual gateways** in the left navigation\.

1. Choose **Create virtual gateway**\.

1. For **Virtual gateway name**, enter a name for your virtual gateway\.

1. \(Optional, but recommended\) Configure **Client policy defaults**\.

   1. \(Optional\) Select **Enforce TLS** if you want the gateway to only communicate with virtual services using Transport Layer Security \(TLS\)\.

   1. \(Optional\) For **Ports**, specify one or more ports that you want to enforce TLS communication with virtual services on\.

   1. For **Certificate discovery method**, select one of the following options\. The certificate that you specify must already exist and meet specific requirements\. For more information, see [Certificate requirements](tls.md#virtual-node-tls-prerequisites)\.
      + **AWS Certificate Manager Private Certificate Authority** hosting – Select one or more existing **Certificates**\.
      + **Local file hosting** – Specify the path to the **Certificate chain** file on the file system where the Envoy is deployed\.

   1. Specify a **Port** and **Protocol** for the **Listener**\. Each virtual gateway can have only one port and protocol specified\. If you need the virtual gateway to route traffic over multiple ports and protocols, then you must create a virtual gateway for each port or prototol\.

1. \(Optional\) To configure logging, selected **Logging**\. Enter the **HTTP access logs path** that you want Envoy to use\. We recommend the `/dev/stdout` path so that you can use Docker log drivers to export your Envoy logs to a service such as Amazon CloudWatch Logs\.
**Note**  
Logs must still be ingested by an agent in your application and sent to a destination\. This file path only instructs Envoy where to send the logs\. 

1. Configure the **Listener**\.

   1. Select a **Protocol** and specify the **Port** that Envoy will listen for traffic on\. The **http** listener permits connection transition to websockets\.

   1. \(Optional\) If you want to configure a health check for your listener, then select **Enable health check**\.

      A health check policy is optional, but if you specify any values for a health policy, then you must specify values for **Healthy threshold**, **Health check interval**, **Health check protocol**, **Timeout period**, and **Unhealthy threshold**\.
      + For **Health check protocol**, choose a protocol\. If you select **grpc**, then your service must conform to the [GRPC Health Checking Protocol](https://github.com/grpc/grpc/blob/master/doc/health-checking.md)\.
      + For **Health check port**, specify the port that the health check should run on\.
      + For **Healthy threshold**, specify the number of consecutive successful health checks that must occur before declaring the listener healthy\.
      + For **Health check interval**, specify the time period in milliseconds between each health check execution\.
      + For **Path**, specify the destination path for the health check request\. This value is only used if the **Health check protocol** is `http` or `http2`\. The value is ignored for other protocols\.
      + For **Timeout period**, specify the amount of time to wait when receiving a response from the health check, in milliseconds\.
      + For **Unhealthy threshold**, specify the number of consecutive failed health checks that must occur before declaring the listener unhealthy\.

   1. \(Optional\) If you want to specify whether virtual nodes communicate with this virtual gateway using TLS, then select **Enable TLS termination**\.
      + For **Mode**, select the mode you want TLS to be configured for on the listener\.
      + For **Certificate method**, select one of the following options\. The certificate must meet specific requirements\. For more information, see [Certificate requirements](tls.md#virtual-node-tls-prerequisites)\.
        + **AWS Certificate Manager hosting** – Select an existing **Certificate**\.
        + **Local file hosting** – Specify the path to the **Certificate chain** and **Private key** files on the file system where Envoy is deployed\.

1. Choose **Create virtual gateway** to finish\.

## Deploy virtual gateway<a name="deploy-virtual-gateway"></a>

Deploy an Amazon ECS or Kubernetes service that contains only the [Envoy container](envoy.md)\. You can also deploy the Envoy container on an [Amazon EC2](https://docs.aws.amazon.com/app-mesh/latest/userguide/appmesh-getting-started.html#update-services) instance\. For more information see [Getting started with App Mesh and Amazon ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/appmesh-getting-started.html#update-services) or [Tutorial: Configure App Mesh integration with Kubernetes](https://docs.aws.amazon.com/app-mesh/latest/userguide/mesh-k8s-integration.html)\. You need to set the `APPMESH_VIRTUAL_NODE_NAME` environment variable to `mesh/mesh-name/virtualGateway/virtual-gateway-name` and you must not specify proxy configuration so that the proxy's traffic doesn't get redirected to itself\.

We recommend that you deploy multiple instances of the container and set up a Network Load Balancer to load balance traffic to the instances\. The service discovery name of the load balancer is the name that you want external services to use to access resources that are in the mesh, such as *myapp\.example\.com*\. For more information see [Creating a Network Load Balancer](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/create-network-load-balancer.html) \(Amazon ECS\), [Creating an External Load Balancer](https://kubernetes.io/docs/tasks/access-application-cluster/create-external-load-balancer/) \(Kubernetes\), or [Tutorial: Increase the availability of your application on Amazon EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-increase-availability.html)\.

Enable proxy authorization for Envoy\. For more information, see [Proxy authorization](proxy-authorization.md)\.

## Deleting a virtual gateway<a name="delete-virtual-gateway"></a>

To delete a virtual gateway using the AWS CLI version 1\.18\.116 or higher, see the AWS CLI reference for the [https://docs.aws.amazon.com/cli/latest/reference/appmesh/delete-virtual-gateway.html](https://docs.aws.amazon.com/cli/latest/reference/appmesh/delete-virtual-gateway.html) command\.

**To delete a virtual gateway using the AWS Management Console**

1. Open the App Mesh console at [https://console\.aws\.amazon\.com/appmesh/](https://console.aws.amazon.com/appmesh/)\. 

1. Choose the mesh that you want to delete a virtual gateway from\. All of the meshes that you own and that have been [shared](sharing.md) with you are listed\.

1. Choose **Virtual gateways** in the left navigation\.

1. Choose the virtual gateway that you want to delete and select **Delete**\. You cannot delete a virtual gateway if it has any associated gateway routes\. You must delete any associated gateway routes first\. You can only delete a virtual gateway where your account is listed as **Resource owner**\.

1. In the confirmation box, type **delete** and then select **Delete**\.
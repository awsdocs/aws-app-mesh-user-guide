# Virtual gateways<a name="virtual_gateways"></a>

A virtual gateway allows resources that are outside of your mesh to communicate to resources that are inside of your mesh\. The virtual gateway represents an Envoy proxy running in an Amazon ECS service, in a Kubernetes service, or on an Amazon EC2 instance\. Unlike a virtual node, which represents Envoy running with an application, a virtual gateway represents Envoy deployed by itself\. 

External resources must be able to resolve a DNS name to an IP address assigned to the service or instance that runs Envoy\. Envoy can then access all of the App Mesh configuration for resources that are inside of the mesh\. The configuration for handling the incoming requests at the Virtual Gateway are specified using [Gateway Routes](https://docs.aws.amazon.com/app-mesh/latest/userguide/gateway-routes.html)\.

**Important**  
A virtual gateway with a HTTP or HTTP2 listener rewrites the incoming request's hostname to the Gateway Route target Virtual Service's name, and the matched prefix from the Gateway Route is rewritten to `/`, by default\. For example, if you have configured the Gateway route match prefix to `/chapter`, and, if the incoming request is `/chapter/1`, the request would be rewritten to `/1`\. To configure rewrites, refer to the [Creating a gateway route](https://docs.aws.amazon.com/app-mesh/latest/userguide/gateway-routes.html#create-gateway-route) section from Gateway Routes\.  
When creating a virtual gateway, `proxyConfiguration` and `user` should not be configured\.

To complete an end\-to\-end walkthrough, see [Configuring Inbound Gateway](https://github.com/aws/aws-app-mesh-examples/tree/main/walkthroughs/howto-ingress-gateway)\.

## Creating a virtual gateway<a name="create-virtual-gateway"></a>

**Note**  
When creating a Virtual Gateway, you must add a namespace selector with a label to identify the list of namespaces with which to associate Gateway Routes to the created Virtual Gateway\.

------
#### [ AWS Management Console ]

**To create a virtual gateway using the AWS Management Console**

1. Open the App Mesh console at [https://console\.aws\.amazon\.com/appmesh/](https://console.aws.amazon.com/appmesh/)\. 

1. Choose the mesh in which you want to create the virtual gateway\. All of the meshes that you own and that have been [shared](sharing.md) with you are listed\.

1. Choose **Virtual gateways** in the left navigation\.

1. Choose **Create virtual gateway**\.

1. For **Virtual gateway name**, enter a name for your virtual gateway\.

1. \(Optional, but recommended\) Configure **Client policy defaults**\.

   1. \(Optional\) Select **Enforce TLS** if you want the gateway to only communicate with virtual services using Transport Layer Security \(TLS\)\.

   1. \(Optional\) For **Ports**, specify one or more ports on which you want to enforce TLS communication with virtual services\.

   1. For **Validation method**, select one of the following options\. The certificate that you specify must already exist and meet specific requirements\. For more information, see [Certificate requirements](tls.md#virtual-node-tls-prerequisites)\.
      + **AWS Certificate Manager Private Certificate Authority** hosting – Select one or more existing **Certificates**\.
      + **Envoy Secret Discovery Service \(SDS\)** hosting – Enter the name of the secret that Envoy fetches using the Secret Discovery Service\.
      + **Local file hosting** – Specify the path to the **Certificate chain** file on the file system where Envoy is deployed\.

   1. \(Optional\) Enter a **Subject Alternative Name**\. To add additional SANs, select **Add SAN**\. SANs must be FQDN or URI formatted\.

   1. \(Optional\) Select **Provide client certificate** and one of the options below to provide a client certificate when a server requests it and enable mutual TLS authentication\. To learn more about mutual TLS, see the App Mesh [Mutual TLS Authentication](https://docs.aws.amazon.com/app-mesh/latest/userguide/mutual-tls.html) docs\.
      + **Envoy Secret Discovery Service \(SDS\)** hosting – Enter the name of the secret that Envoy fetches using the Secret Discovery Service\.
      + **Local file hosting** – Specify the path to the **Certificate chain** file, as well as the **Private key**, on the file system where Envoy is deployed\. For a complete, end\-to\-end walk through of deploying a mesh with a sample application using encryption with local files, see [Configuring TLS with File Provided TLS Certificates](https://github.com/aws/aws-app-mesh-examples/tree/main/walkthroughs/howto-tls-file-provided) on GitHub\.

   1. Specify a **Port** and **Protocol** for the **Listener**\. Each virtual gateway can have only one port and protocol specified\. If you need the virtual gateway to route traffic over multiple ports and protocols, then you must create a virtual gateway for each port or prototol\.

1. \(Optional\) To configure logging, selected **Logging**\. Enter the **HTTP access logs path** that you want Envoy to use\. We recommend the `/dev/stdout` path so that you can use Docker log drivers to export your Envoy logs to a service such as Amazon CloudWatch Logs\.
**Note**  
Logs must still be ingested by an agent in your application and sent to a destination\. This file path only instructs Envoy where to send the logs\. 

1. Configure the **Listener**\.

   1. Select a **Protocol** and specify the **Port** on which Envoy listens for traffic\. The **http** listener permits connection transition to websockets\.

   1. \(Optional\) **Enable connection pool** 

      Connection pooling limits the number of connections that the Virtual Gateway Envoy can concurrently establish\. It is intended to protect your Envoy instance from being overwhelmed with connections and lets you adjust traffic shaping for the needs of your applications\.

      You can configure destination\-side connection pool settings for a virtual gateway listener\. App Mesh sets the client\-side connection pool settings to infinite by default, simplifying mesh configuration\.
**Note**  
The `connectionPool` and `connectionPool`portMapping protocols must be the same\. If your listener protocol is `grpc` or `http2`, specify `maxRequests` only\. If your listener protocol is `http`, you can specify both `maxConnections` and `maxPendingRequests`\. 
      + For **Maximum connections**, specify the maximum number of outbound connections\.
      + For **Maximum requests**, specify maximum number of parallel requests that can be established with the Virtual Gateway Envoy\.
      + \(Optional\) For **Maximum pending requests**, specify the number of overflowing requests after **Maximum connections** that an Envoy queues\. The default value is `2147483647`\.

   1. \(Optional\) If you want to configure a health check for your listener, then select **Enable health check**\.

      A health check policy is optional, but if you specify any values for a health policy, then you must specify values for **Healthy threshold**, **Health check interval**, **Health check protocol**, **Timeout period**, and **Unhealthy threshold**\.
      + For **Health check protocol**, choose a protocol\. If you select **grpc**, then your service must conform to the [GRPC Health Checking Protocol](https://github.com/grpc/grpc/blob/master/doc/health-checking.md)\.
      + For **Health check port**, specify the port that the health check should run on\.
      + For **Healthy threshold**, specify the number of consecutive successful health checks that must occur before declaring the listener healthy\.
      + For **Health check interval**, specify the time period in milliseconds between each health check execution\.
      + For **Path**, specify the destination path for the health check request\. This value is only used if the **Health check protocol** is `http` or `http2`\. The value is ignored for other protocols\.
      + For **Timeout period**, specify the amount of time to wait when receiving a response from the health check in milliseconds\.
      + For **Unhealthy threshold**, specify the number of consecutive failed health checks that must occur before declaring the listener unhealthy\.

   1. \(Optional\) If you want to specify whether virtual nodes communicate with this virtual gateway using TLS, then select **Enable TLS termination**\.
      + For **Mode**, select the mode that you want TLS to be configured for on the listener\.
      + For **Certificate method**, select one of the following options\. The certificate must meet specific requirements\. For more information, see [Certificate requirements](tls.md#virtual-node-tls-prerequisites)\.
        + **AWS Certificate Manager hosting** – Select an existing **Certificate**\.
        + **Envoy Secret Discovery Service \(SDS\)** hosting – Enter the name of the secret that Envoy fetches using the Secret Discovery Service\.
        + **Local file hosting** – Specify the path to the **Certificate chain** and **Private key** files on the file system where Envoy is deployed\.
      + \(Optional\) Select **Require client certificate** and one of the options below to enable mutual TLS authentication if the client provides a certificate\. To learn more about mutual TLS, see the App Mesh [Mutual TLS Authentication](https://docs.aws.amazon.com/app-mesh/latest/userguide/mutual-tls.html) docs\.
        + **Envoy Secret Discovery Service \(SDS\)** hosting – Enter the name of the secret that Envoy fetches using the Secret Discovery Service\.
        + **Local file hosting** – Specify the path to the **Certificate chain** file on the file system where Envoy is deployed\.
      + \(Optional\) Enter a **Subject Alternative Name**\. To add additional SANs, select **Add SAN**\. SANs must be FQDN or URI formatted\.

1. Choose **Create virtual gateway** to finish\.

------
#### [ AWS CLI ]

**To create a virtual gateway using the AWS CLI\.**

Create a virtual gateway using the following command and input JSON \(replace the *red* values with your own\):

1. 

   ```
   aws appmesh create-virtual-gateway \ 
   --mesh-name meshName \ 
   --virtual-gateway-name virtualGatewayName \ 
   --cli-input-json file://create-virtual-gateway.json
   ```

1. Contents of **example** create\-virtual\-gateway\.json:

   ```
   {
       "spec": {
         "listeners": [
           {
             "portMapping": {
               "port": 9080,
               "protocol": "http"
             }
           }
         ]
       }
   }
   ```

1. Example output:

   ```
   {
       "virtualGateway": {
           "meshName": "meshName",
           "metadata": {
               "arn": "arn:aws:appmesh:us-west-2:123456789012:mesh/meshName/virtualGateway/virtualGatewayName",
               "createdAt": "2022-04-06T10:42:42.015000-05:00",
               "lastUpdatedAt": "2022-04-06T10:42:42.015000-05:00",
               "meshOwner": "123456789012",
               "resourceOwner": "123456789012",
               "uid": "a1b2c3d4-5678-90ab-cdef-11111EXAMPLE",
               "version": 1
           },
           "spec": {
               "listeners": [
                   {
                       "portMapping": {
                           "port": 9080,
                           "protocol": "http"
                       }
                   }
               ]
           },
           "status": {
               "status": "ACTIVE"
           },
           "virtualGatewayName": "virtualGatewayName"
       }
   }
   ```

For more information on creating a virtual gateway with the AWS CLI for App Mesh, see the [create\-virtual\-gateway](https://docs.aws.amazon.com/cli/latest/reference/appmesh/create-virtual-gateway.html) command in the AWS CLI reference\.

------

## Deploy virtual gateway<a name="deploy-virtual-gateway"></a>

Deploy an Amazon ECS or Kubernetes service that contains only the [Envoy container](envoy.md)\. You can also deploy the Envoy container on an Amazon EC2 instance\. For more information, see [Getting started with App Mesh and Amazon EC2](https://docs.aws.amazon.com/app-mesh/latest/userguide/getting-started-ec2.html)\. For more information on how to deploy on Amazon ECS see [Getting started with App Mesh and Amazon ECS](https://docs.aws.amazon.com/app-mesh/latest/userguide/getting-started-ecs.html) or [Getting started with AWS App Mesh and Kubernetes](https://docs.aws.amazon.com/app-mesh/latest/userguide/getting-started-kubernetes.html) to deploy to Kubernetes\. You need to set the `APPMESH_RESOURCE_ARN` environment variable to `mesh/mesh-name/virtualGateway/virtual-gateway-name` and you must not specify proxy configuration so that the proxy's traffic doesn't get redirected to itself\. By default, App Mesh uses the name of the resource you specified in `APPMESH_RESOURCE_ARN` when Envoy is referring to itself in metrics and traces\. You can override this behavior by setting the `APPMESH_RESOURCE_CLUSTER `environment variable with your own name\.

We recommend that you deploy multiple instances of the container and set up a Network Load Balancer to load balance traffic to the instances\. The service discovery name of the load balancer is the name that you want external services to use to access resources that are in the mesh, such as *myapp\.example\.com*\. For more information see [Creating a Network Load Balancer](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/create-network-load-balancer.html) \(Amazon ECS\), [Creating an External Load Balancer](https://kubernetes.io/docs/tasks/access-application-cluster/create-external-load-balancer/) \(Kubernetes\), or [Tutorial: Increase the availability of your application on Amazon EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-increase-availability.html)\. You can also find more examples and walkthroughs in our [App Mesh examples](https://docs.aws.amazon.com/app-mesh/latest/userguide/examples.html)\.

Enable proxy authorization for Envoy\. For more information, see [Envoy Proxy authorization](proxy-authorization.md)\.

## Deleting a virtual gateway<a name="delete-virtual-gateway"></a>

------
#### [ AWS Management Console ]

**To delete a virtual gateway using the AWS Management Console**

1. Open the App Mesh console at [https://console\.aws\.amazon\.com/appmesh/](https://console.aws.amazon.com/appmesh/)\. 

1. Choose the mesh from which you want to delete a virtual gateway\. All of the meshes that you own and that have been [shared](sharing.md) with you are listed\.

1. Choose **Virtual gateways** in the left navigation\.

1. Choose the virtual gateway that you want to delete and select **Delete**\. You cannot delete a virtual gateway if it has any associated gateway routes\. You must delete any associated gateway routes first\. You can only delete a virtual gateway where your account is listed as **Resource owner**\.

1. In the confirmation box, type **delete** and then select **Delete**\.

------
#### [ AWS CLI ]

**To delete a virtual gateway using the AWS CLI**

1. Use the following command to delete your virtual gateway \(replace the *red* values with your own\):

   ```
   aws appmesh delete-virtual-gateway \
        --mesh-name meshName \
        --virtual-gateway-name virtualGatewayName
   ```

1. Example output:

   ```
   {
       "virtualGateway": {
           "meshName": "meshName",
           "metadata": {
               "arn": "arn:aws:appmesh:us-west-2:123456789012:mesh/meshName/virtualGateway/virtualGatewayName",
               "createdAt": "2022-04-06T10:42:42.015000-05:00",
               "lastUpdatedAt": "2022-04-07T10:57:22.638000-05:00",
               "meshOwner": "123456789012",
               "resourceOwner": "123456789012",
               "uid": "a1b2c3d4-5678-90ab-cdef-11111EXAMPLE",
               "version": 2
           },
           "spec": {
               "listeners": [
                   {
                       "portMapping": {
                           "port": 9080,
                           "protocol": "http"
                       }
                   }
               ]
           },
           "status": {
               "status": "DELETED"
           },
           "virtualGatewayName": "virtualGatewayName"
       }
   }
   ```

For more information on deleting a virtual gateway with the AWS CLI for App Mesh, see the [delete\-virtual\-gateway](https://docs.aws.amazon.com/cli/latest/reference/appmesh/delete-virtual-gateway.html) command in the AWS CLI reference\.

------
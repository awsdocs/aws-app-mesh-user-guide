# Gateway routes<a name="gateway-routes"></a>

A gateway route is attached to a virtual gateway and routes traffic to an existing virtual service\. If a route matches a request, it can distribute traffic to a target virtual service\. This topic helps you work with gateway routes in a service mesh\.

## Creating a gateway route<a name="create-gateway-route"></a>

To create a gateway route using the AWS CLI version 1\.18\.116 or later, see the examples in the AWS CLI reference for the [create\-gateway\-route](https://docs.aws.amazon.com/cli/latest/reference/appmesh/create-gateway-route.html) command\.

**To create a gateway route using the AWS Management Console**

1. Open the App Mesh console at [https://console\.aws\.amazon\.com/appmesh/](https://console.aws.amazon.com/appmesh/)\. 

1. Choose the mesh that you want to create the gateway route in\. All of the meshes that you own and that have been [shared](sharing.md) with you are listed\.

1. Choose **Virtual gateways** in the left navigation\.

1. Choose the virtual gateway that you want to associate a new gateway route with\. If none are listed, then you need to [create a virtual gateway](virtual_gateways.md#create-virtual-gateway) first\. You can only create a gateway route for a virtual gateway that your account is listed as the **Resource owner** of\.

1. In the **Gateway routes** table, choose **Create gateway route**\.

1. For **Gateway route name**, specify the name to use for your gateway route\.

1. For **Gateway route type**, enter a name for the gateway route\.

1. For **Gateway route type** choose one of the following protocols\.
   + **http** or **http2** – Specify a **Match prefix**\. A matched request by a gateway route is rewritten to the target virtual service's name and the matched prefix is rewritten to `/`, by default\. For example, if you have configured the **Match prefix** to be `/`, when a request is received for `www.example.com/chapter/page`, it would match the prefix, because it has a `/` after the host name\. The request would be rewritten to the name of a virtual service, such as `myservicea.svc.cluster.local/chapter/page`\. 

     Depending on how you configure your virtual service, it could use a virtual router to route the request to different virtual nodes, based on specific prefixes or headers\. If the **Match prefix** in the gateway route example were `/chapter`, rather than `/`, then:
     + A request to `www.example.com/chapter/page` would be re\-written to `www.example.com/page`\. Since `chapter` was the matched prefix, it would be removed in the re\-written request\. 
     + A request to `www.example.com/chapter1` would be rewritten to `www.example.com/1`\. Again, `chapter` was removed, since it was the matched prefix, but `/1` would be added to the rewritten request from the original request\. 

     Provided the virtual service’s virtual router has a route defined with a prefix of `/1` or `/page`, the request would be routed to a virtual node\. If the virtual service's virtual router doesn't have such a route however, the request would be dropped\. This behavior enables you to minimize the number of gateway routes that are needed, but does require you to ensure that you have routes for a virtual service’s virtual router for each of the expected prefixes that may be requested\. Learn more about [Virtual services](virtual_services.md), [Virtual routers](virtual_routers.md), and [Routes](routes.md)\.
**Important**  
You can't specify either `/aws-appmesh*` or `/aws-app-mesh*` for **Match prefix**\. These prefixes are reserved for future App Mesh internal use\.
If multiple gateway routes are defined, then a request is matched to the route with the longest prefix\. For example, if two gateway routes existed, with one having a prefix of `/chapter` and one having a prefix of `/`, then a request for `www.example.com/chapter/` would be matched to the gateway route with the `/chapter` prefix\.
   + **grpc** – Specify a **Service name**\. Depending on how you configure your virtual service, the virtual service could route the request to different virtual nodes based on method name or specific metadata, for example\. To learn more about virtual services, see [Virtual services](virtual_services.md)\. 
**Important**  
You can't specify `/aws.app-mesh*` or `aws.appmesh` for the **Service name**\. These service names are reserved for future App Mesh internal use\.

1. Select an existing **Virtual service name**\. If none are listed, then you need to create a [virtual service](virtual_services.md#create-virtual-service) first\.

1. Choose **Create gateway route** to finish\.

## Deleting a gateway route<a name="delete-gateway-route"></a>

To delete a route using the AWS CLI version 1\.18\.116 or higher, see the AWS CLI reference for the [https://docs.aws.amazon.com/cli/latest/reference/appmesh/delete-gateway-route.html](https://docs.aws.amazon.com/cli/latest/reference/appmesh/delete-gateway-route.html) command\.

**To delete a gateway route using the AWS Management Console**

1. Open the App Mesh console at [https://console\.aws\.amazon\.com/appmesh/](https://console.aws.amazon.com/appmesh/)\. 

1. Choose the mesh that you want to delete a gateway route from\. All of the meshes that you own and that have been [shared](sharing.md) with you are listed\.

1. Choose **Virtual gateways** in the left navigation\.

1. Choose the virtual gateway that you want to delete a gateway route from\.

1. In the **Gateway routes** table, choose the gateway route that you want to delete and select **Delete**\. You can only delete a gateway route if your account is listed for **Resource owner**\.

1. In the confirmation box, type **delete** and then select **Delete**\.
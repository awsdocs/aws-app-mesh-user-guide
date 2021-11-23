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

1. For **Gateway route type** choose either **http**, **http2**, or **grcp**\.

1. Select an existing **Virtual service name**\. If none are listed, then you need to create a [virtual service](virtual_services.md#create-virtual-service) first\.

1. \(Optional\) For **Priority**, specify the priority for this gateway route\.

1. For **Match** configuration, specify:
   + 

     If **http/http2** is the selected type:
     + \(Optional\) **Method** ‐ specifies the method header to be matched in the incoming **http**/**http2** requests\.
     + \(Optional\) **Exact/Suffix hostname** ‐ specifies the hostname that should be matched on the incoming request to route to the target virtual service\.
     + \(Optional\) **Prefix/Exact/Regex path** ‐ method of matching the path of the URL\.
       + **Prefix match** ‐ a matched request by a gateway route is rewritten to the target virtual service's name and the matched prefix is rewritten to `/`, by default\. Depending on how you configure your virtual service, it could use a virtual router to route the request to different virtual nodes, based on specific prefixes or headers\. 
**Important**  
You can't specify either `/aws-appmesh*` or `/aws-app-mesh*` for **Prefix match**\. These prefixes are reserved for future App Mesh internal use\.
If multiple gateway routes are defined, then a request is matched to the route with the longest prefix\. For example, if two gateway routes existed, with one having a prefix of `/chapter` and one having a prefix of `/`, then a request for `www.example.com/chapter/` would be matched to the gateway route with the `/chapter` prefix\.
**Note**  
If you enable **Path**/**Prefix** based matching, App Mesh enables path normalization \([normalize\_path](https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/filters/network/http_connection_manager/v3/http_connection_manager.proto#envoy-v3-api-field-extensions-filters-network-http-connection-manager-v3-httpconnectionmanager-normalize-path) and [merge\_slashes](https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/filters/network/http_connection_manager/v3/http_connection_manager.proto#envoy-v3-api-field-extensions-filters-network-http-connection-manager-v3-httpconnectionmanager-merge-slashes)\) to minimize the probability of path confusion vulnerabilities\.  
Path confusion vulnerabilities occur when parties participating in the request use different path representations\.
       + **Exact match** ‐ the exact param disables the partial matching for a route and makes sure that it only returns the route if the path is an EXACT match to the current url\.
       + **Regex match** ‐ used to describe patterns where multiple URLs may actually identify a single page on the website\.
     + \(Optional\) **Query parameters** ‐ this field allows you to match on the query parameters\.
     + \(Optional\) **Headers** ‐ specifies the headers for **http** and **http2**\. It should match the incoming request to route to the target virtual service\.\.
   + 

     If **grpc** is the selected type:
     + **Hostname match type** and \(optional\) **Exact/Suffix match** ‐ specifies the hostname that should be matched on the incoming request to route to the target virtual service\. 
     + **grpc service name** ‐ the gRPC service acts as an API for your application and is defined with ProtoBuf\.
**Important**  
You can't specify `/aws.app-mesh*` or `aws.appmesh` for the **Service name**\. These service names are reserved for future App Mesh internal use\.
     + \(Optional\) **Metadata** ‐ specifies the metadata for **grpc**\. It should match the incoming request to route to the target virtual service\.

1. \(Optional\) For **Rewrite** configuration:
   + 

     If **http/http2** is the selected type:
     + 

       If **Prefix** is the selected match type:
       + **Override automatic rewrite of hostname** ‐ by default the hostname will be rewritten to the target virtual service's name\.
       + **Override automatic rewrite of prefix** ‐ When toggled on, **Prefix rewrite** specifies the value of the rewritten prefix\.
     + 

       If **Exact** is the selected match type:
       + **Override automatic rewrite of hostname** ‐ by default the hostname will be rewritten to the target virtual service's name\.
       + **Path rewrite** specifies the value of the rewritten path\. No default path\.
     + 

       If **Regex** is the selected match type:
       + **Override automatic rewrite of hostname** ‐ by default the hostname will be rewritten to the target virtual service's name\.
       + **Path rewrite** specifies the value of the rewritten path\. No default path\.
   + 

     If **grpc** is the selected type:
     + **Override automatic rewrite of hostname** ‐ by default the hostname will be rewritten to the target virtual service's name\.

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
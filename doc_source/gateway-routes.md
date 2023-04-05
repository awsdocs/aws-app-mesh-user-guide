# Gateway routes<a name="gateway-routes"></a>

A gateway route is attached to a virtual gateway and routes traffic to an existing virtual service\. If a route matches a request, it can distribute traffic to a target virtual service\. This topic helps you work with gateway routes in a service mesh\.

## Creating a gateway route<a name="create-gateway-route"></a>

------
#### [ AWS Management Console ]

**To create a gateway route using the AWS Management Console**

1. Open the App Mesh console at [https://console\.aws\.amazon\.com/appmesh/](https://console.aws.amazon.com/appmesh/)\. 

1. Choose the mesh in which you want to create the gateway route\. All of the meshes that you own and that have been [shared](sharing.md) with you are listed\.

1. Choose **Virtual gateways** in the left navigation\.

1. Choose the virtual gateway with which you want to associate a new gateway route\. If none are listed, then you need to [create a virtual gateway](virtual_gateways.md#create-virtual-gateway) first\. You can only create a gateway route for a virtual gateway of which your account is listed as the **Resource owner**\.

1. In the **Gateway routes** table, choose **Create gateway route**\.

1. For **Gateway route name**, specify the name to use for your gateway route\.

1. For **Gateway route type** choose either **http**, **http2**, or **grpc**\.

1. Select an existing **Virtual service name**\. If none are listed, then you need to create a [virtual service](virtual_services.md#create-virtual-service) first\.

1. Choose the port that corresponds to the target for **Virtual service provider port**\. Virtual service provider port is **required** when the provider \(router or node\) of the selected virtual service has multiple listeners\.

1. \(Optional\) For **Priority**, specify the priority for this gateway route\.

1. For **Match** configuration, specify:
   + 

     If **http/http2** is the selected type:
     + \(Optional\) **Method** ‐ Specifies the method header to be matched in the incoming **http**/**http2** requests\.
     + \(Optional\) **Port match** ‐ Match the port for incoming traffic\. Port match is **required** if this virtual gateway has multiple listeners\.
     + \(Optional\) **Exact/Suffix hostname** ‐ Specifies the hostname that should be matched on the incoming request to route to the target virtual service\.
     + \(Optional\) **Prefix/Exact/Regex path** ‐ The method of matching the path of the URL\.
       + **Prefix match** ‐ A matched request by a gateway route is rewritten to the target virtual service's name and the matched prefix is rewritten to `/`, by default\. Depending on how you configure your virtual service, it could use a virtual router to route the request to different virtual nodes, based on specific prefixes or headers\. 
**Important**  
You can't specify either `/aws-appmesh*` or `/aws-app-mesh*` for **Prefix match**\. These prefixes are reserved for future App Mesh internal use\.
If multiple gateway routes are defined, then a request is matched to the route with the longest prefix\. For example, if two gateway routes existed, with one having a prefix of `/chapter` and one having a prefix of `/`, then a request for `www.example.com/chapter/` would be matched to the gateway route with the `/chapter` prefix\.
**Note**  
If you enable **Path**/**Prefix** based matching, App Mesh enables path normalization \([normalize\_path](https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/filters/network/http_connection_manager/v3/http_connection_manager.proto#envoy-v3-api-field-extensions-filters-network-http-connection-manager-v3-httpconnectionmanager-normalize-path) and [merge\_slashes](https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/filters/network/http_connection_manager/v3/http_connection_manager.proto#envoy-v3-api-field-extensions-filters-network-http-connection-manager-v3-httpconnectionmanager-merge-slashes)\) to minimize the probability of path confusion vulnerabilities\.  
Path confusion vulnerabilities occur when parties participating in the request use different path representations\.
       + **Exact match** ‐ The exact parameter disables the partial matching for a route and makes sure that it only returns the route if the path is an **EXACT** match to the current URL\.
       + **Regex match** ‐ Used to describe patterns where multiple URLs may actually identify a single page on the website\.
     + \(Optional\) **Query parameters** ‐ This field allows you to match on the query parameters\.
     + \(Optional\) **Headers** ‐ Specifies the headers for **http** and **http2**\. It should match the incoming request to route to the target virtual service\.\.
   + 

     If **grpc** is the selected type:
     + **Hostname match type** and \(optional\) **Exact/Suffix match** ‐ Specifies the hostname that should be matched on the incoming request to route to the target virtual service\. 
     + **grpc service name** ‐ The **grpc** service acts as an API for your application and is defined with ProtoBuf\.
**Important**  
You can't specify `/aws.app-mesh*` or `aws.appmesh` for the **Service name**\. These service names are reserved for future App Mesh internal use\.
     + \(Optional\) **Metadata** ‐ Specifies the metadata for **grpc**\. It should match the incoming request to route to the target virtual service\.

1. \(Optional\) For **Rewrite** configuration:
   + 

     If **http/http2** is the selected type:
     + 

       If **Prefix** is the selected match type:
       + **Override automatic rewrite of hostname** ‐ By default the hostname is rewritten to the target virtual service's name\.
       + **Override automatic rewrite of prefix** ‐ When toggled on, **Prefix rewrite** specifies the value of the rewritten prefix\.
     + 

       If **Exact Path** is the selected match type:
       + **Override automatic rewrite of hostname** ‐ by default the hostname is rewritten to the target virtual service's name\.
       + **Path rewrite** ‐ Specifies the value of the rewritten path\. No default path\.
     + 

       If **Regex Path** is the selected match type:
       + **Override automatic rewrite of hostname** ‐ by default the hostname is rewritten to the target virtual service's name\.
       + **Path rewrite** ‐ Specifies the value of the rewritten path\. No default path\.
   + 

     If **grpc** is the selected type:
     + **Override automatic rewrite of hostname** ‐ By default the hostname is rewritten to the target virtual service's name\.

1. Choose **Create gateway route** to finish\.

------
#### [ AWS CLI ]

**To create a gateway route using the AWS CLI\.**

Create a gateway route using the following command and input JSON \(replace the *red* values with your own\):

1. 

   ```
   aws appmesh create-virtual-gateway \ 
   --mesh-name meshName \ 
   --virtual-gateway-name virtualGatewayName \
   --gateway-route-name gatewayRouteName \ 
   --cli-input-json file://create-gateway-route.json
   ```

1. Contents of **example** create\-gateway\-route\.json:

   ```
   {
       "spec": {
           "httpRoute" : {
               "match" : {
                   "prefix" : "/"
               },
               "action" : {
                   "target" : {
                       "virtualService": {
                           "virtualServiceName": "serviceA.svc.cluster.local"
                       }
                   }
               }
           }
       }
   }
   ```

1. Example output:

   ```
   {
       "gatewayRoute": {
           "gatewayRouteName": "gatewayRouteName",
           "meshName": "meshName",
           "metadata": {
               "arn": "arn:aws:appmesh:us-west-2:210987654321:mesh/meshName/virtualGateway/virtualGatewayName/gatewayRoute/gatewayRouteName",
               "createdAt": "2022-04-06T11:05:32.100000-05:00",
               "lastUpdatedAt": "2022-04-06T11:05:32.100000-05:00",
               "meshOwner": "123456789012",
               "resourceOwner": "210987654321",
               "uid": "a1b2c3d4-5678-90ab-cdef-11111EXAMPLE",
               "version": 1
           },
           "spec": {
               "httpRoute": {
                   "action": {
                       "target": {
                           "virtualService": {
                               "virtualServiceName": "serviceA.svc.cluster.local"
                           }
                       }
                   },
                   "match": {
                       "prefix": "/"
                   }
               }
           },
           "status": {
               "status": "ACTIVE"
           },
           "virtualGatewayName": "gatewayName"
       }
   }
   ```

For more information on creating a gateway route with the AWS CLI for App Mesh, see the [create\-gateway\-route](https://docs.aws.amazon.com/cli/latest/reference/appmesh/create-gateway-route.html) command in the AWS CLI reference\.

------

## Deleting a gateway route<a name="delete-gateway-route"></a>

------
#### [ AWS Management Console ]

**To delete a gateway route using the AWS Management Console**

1. Open the App Mesh console at [https://console\.aws\.amazon\.com/appmesh/](https://console.aws.amazon.com/appmesh/)\. 

1. Choose the mesh from which you want to delete a gateway route\. All of the meshes that you own and that have been [shared](sharing.md) with you are listed\.

1. Choose **Virtual gateways** in the left navigation\.

1. Choose the virtual gateway from which you want to delete a gateway route\.

1. In the **Gateway routes** table, choose the gateway route that you want to delete and select **Delete**\. You can only delete a gateway route if your account is listed as **Resource owner**\.

1. In the confirmation box, type **delete** and then click on **Delete**\.

------
#### [ AWS CLI ]

**To delete a gateway route using the AWS CLI**

1. Use the following command to delete your gateway route \(replace the *red* values with your own\):

   ```
   aws appmesh delete-gateway-route \
        --mesh-name meshName \
        --virtual-gateway-name virtualGatewayName \
        --gateway-route-name gatewayRouteName
   ```

1. Example output:

   ```
   {
       "gatewayRoute": {
           "gatewayRouteName": "gatewayRouteName",
           "meshName": "meshName",
           "metadata": {
               "arn": "arn:aws:appmesh:us-west-2:210987654321:mesh/meshName/virtualGateway/virtualGatewayName/gatewayRoute/gatewayRouteName",
               "createdAt": "2022-04-06T11:05:32.100000-05:00",
               "lastUpdatedAt": "2022-04-07T10:36:33.191000-05:00",
               "meshOwner": "123456789012",
               "resourceOwner": "210987654321",
               "uid": "a1b2c3d4-5678-90ab-cdef-11111EXAMPLE",
               "version": 2
           },
           "spec": {
               "httpRoute": {
                   "action": {
                       "target": {
                           "virtualService": {
                               "virtualServiceName": "serviceA.svc.cluster.local"
                           }
                       }
                   },
                   "match": {
                       "prefix": "/"
                   }
               }
           },
           "status": {
               "status": "DELETED"
           },
           "virtualGatewayName": "virtualGatewayName"
       }
   }
   ```

For more information on deleting a gateway route with the AWS CLI for App Mesh, see the [delete\-gateway\-route](https://docs.aws.amazon.com/cli/latest/reference/appmesh/delete-gateway-route.html) command in the AWS CLI reference\.

------
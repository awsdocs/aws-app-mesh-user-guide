# HTTP Headers<a name="route-http-headers"></a>

You can route traffic based on the presence and values of headers in a request\. You can create a route with HTTP\-header\-based routing using the AWS Management Console or the AWS CLI\. Select the name of the tool that you want to use to create a route\.

------
#### [ AWS Management Console ]

1. Open the App Mesh console at [https://console\.aws\.amazon\.com/appmesh/](https://console.aws.amazon.com/appmesh/)\.

1. Choose the mesh that you want to create the route in\.

1. Choose **Virtual routers** in the left navigation\.

1. Choose the virtual router that you want to associate a new route with\. 

1. In the **Routes** table, choose **Create route**\.

1. For **Route name**, specify the name to use for your route\.

1. For **Route type**, choose the protocol for your route\.

1. \(Optional\) For **Route priority**, specify a priority from 0\-1000 to use for your route\. Routes are matched based on the specified value, where 0 is the highest priority\. If no value is specified, then the route with the longest prefix is selected\.

1. For **Virtual node name**, choose the virtual node that this route will serve traffic to\. If none are listed, then you need to [create a virtual node](https://docs.aws.amazon.com//app-mesh/latest/userguide/virtual_nodes.html) first\.

1. For **Weight**, choose a relative weight for the route\. Select **Add target** to add additional virtual nodes\. The total weight for all targets combined must be less than or equal to 100\.

1. \(Optional\) To use HTTP path\-based routing, choose **Additional configuration** and then specify the **Prefix** that the route should match\. For additional information about path\-based routing, see [Path\-based Routing](route-path.md)\. 

1. \(Optional\) Select a **Method** to use header\-based routing for your route\. 

1. \(Optional\) Select a **Scheme** to use header\-based routing for your route\. 

1. \(Optional\) Select **Add header**\. Enter the **Header name** that you want to route based on, select a **Match type**, and enter a **Match value**\. Selecting **Invert** will match the opposite\. For example, if you specified a header named `clientRequestId` with a **Prefix** of `123`, and selected **Invert**, then the route would be matched for any request that had a header that started with anything other than `123`\. Select **Add header** to add up to ten headers\. 

1. Choose **Create route** to finish\.

------
#### [ AWS CLI ]

1. Create a JSON file named *`create-route-http-header.json`* with a route configuration\. The following example JSON file routes all requests to *`serviceB`* that have any path prefix in an *HTTPS** post* where the `clientRequestId` header has a `prefix` of `123`\.

   ```
   {
      "meshName" : "app1",
      "routeName" : "route-headers1",
      "spec" : {
         "httpRoute" : {
            "action" : {
               "weightedTargets" : [
                  {
                     "virtualNode" : "serviceB",
                     "weight" : 100
                  }
               ]
            },
            "match" : {
               "headers" : [
                  {
                     "invert" : false,
                     "match" : {
                        "prefix" : "123"
                     },
                     "name" : "clientRequestId"
                  }
               ],
               "method" : "POST",
               "prefix" : "/",
               "scheme" : "https"
            }
         }
      },
      "virtualRouterName" : "virtual-router1"
   }
   ```

   For a list of all values for all properties, see the AWS Management Console\.

1. Run the following command to create the route\.

   ```
   aws appmesh create-route --cli-input-json file://create-route-http-header.json
   ```
**Note**  
You must use at least version 1\.16\.178 of the AWS CLI with App Mesh\. To install the latest version of the AWS CLI, see [Installing the AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/installing.html) in the *AWS Command Line Interface User Guide*\.

------
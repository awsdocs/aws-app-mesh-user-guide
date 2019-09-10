# Retry policy<a name="route-retry-policy"></a>

A retry policy enables clients to protect themselves from intermittent network failures or intermittent server\-side failures\. You can add retry logic to a route\. You can create a route with a retry policy using the AWS Management Console or the AWS CLI\. Select the name of the tool that you want to create a route with\. 

------
#### [ AWS Management Console ]

1. Open the App Mesh console at [https://console\.aws\.amazon\.com/appmesh/](https://console.aws.amazon.com/appmesh/)\.

1. Choose the mesh that you want to create the route in\.

1. Choose **Virtual routers** in the left navigation\.

1. Choose the virtual router that you want to associate a new route with\. 

1. In the **Routes** table, choose **Create route**\.

1. For **Route name**, specify the name to use for your route\.

1. For **Route type**, choose the protocol for your route\.

1. \(Optional\) For **Route priority**, specify a priority from 0\-1000 to use for your route\. Routes are matched based on the specified value, where 0 is the highest priority\.

1. For **Virtual node name**, choose the virtual node that this route will serve traffic to\. If none are listed, then you need to [create a virtual node](https://docs.aws.amazon.com//app-mesh/latest/userguide/virtual_nodes.html) first\.

1. For **Weight**, choose a relative weight for the route\. Select **Add target** to add additional virtual nodes\. The total weight for all targets combined must be less than or equal to 100\.

1. \(Optional\) To use HTTP path and header\-based routing, choose **Additional configuration**\. 

1. \(Optional\) To use HTTP path\-based routing, specify the **Prefix** that the route should match\. For additional information about path\-based routing, see [Path\-based Routing](https://docs.aws.amazon.com//app-mesh/latest/userguide/route-path.html)\. 

1. \(Optional\) Select a **Method** to use header\-based routing for your route\. 

1. \(Optional\) Select a **Scheme** to use header\-based routing for your route\. 

1. \(Optional\) Select **Add header**\. Enter the **Header name** that you want to route based on, select a **Match type**, and enter a **Match value**\. Selecting **Invert** will match the opposite\. For additional information about HTTP header\-based routing, see [HTTP Headers](https://docs.aws.amazon.com//app-mesh/latest/userguide/route-http-headers.html)\. 

1. \(Optional\) Select **Add header** to add up to ten headers\. 

1. \(Optional\) For **Retry timeout**, enter the number of units for the timeout duration\. For example, you can can define a retry policy that attempts to route traffic three times when the routing attempt receives a TCP connection error or an HTTP server\-error or gateway\-error\. You might specify a duration of 15 seconds between retry attempts\.

1. \(Optional\) For **Retry timeout unit**, select a unit\.

1. \(Optional\) For **Max retries**, enter the number of times to retry the route when an attempt fails\.

1. \(Optional\) Select one or more **HTTP retry events**\.

1. \(Optional\) Select a **TCP retry event**\.

1. Choose **Create route** to finish\.

------
#### [ AWS CLI ]

1. Create a JSON file named *create\-route\-retry\-policy\.json* with a route configuration\. In the following JSON example, a route with a retry policy is created\. The retry policy attempts to route traffic *3* times when it receives a TCP `connection error` or an HTTP `server-error` or `gateway-error`\. Each retry attempt waits *15* *seconds* before timing out\. For the route to create successfully, a mesh and virtual node with the names specified must exist \.

   ```
   {
      "meshName" : "App1",
      "routeName" : "Route-retries1",
      "spec" : {
         "httpRoute" : {
            "action" : {
               "weightedTargets" : [
                  {
                     "virtualNode" : "ServiceB",
                     "weight" : 100
                  }
               ]
            },
            "match" : {
               "prefix" : "/"
            },
            "retryPolicy" : {
               "perRetryTimeout" : {
                  "value": 15,
                  "unit": "s"
               },
               "maxRetries" : 3,
               "httpRetryEvents" : [ "server-error", "gateway-error" ],
               "tcpRetryEvents" : [ "connection-error" ]
            }
         }
      },
      "virtualRouterName" : "Virtual-router1"
   }
   ```

   You must specify at least one value for `httpRetryEvents`, or at least one value for `tcpRetryEvents`\. You can also specify at least one value for each setting\. The only valid value for `tcpRetryEvents` is `connection-error`\. Valid values for `httpRetryEvents` include the following: 
   + `server-error` – HTTP status codes `500`, `501`, `502`, `503`, `504`, `505`, `506`, `507`, `508`, `510`, and `511`
   + `gateway-error` – HTTP status codes `502`, `503`, and `504`
   + `client-error` – HTTP status code `409`
   + `stream-error` – Retry on refused stream

   The `maxRetries` value is optional, and the default is `1`\. The `perRetryTimeout` settings are optional, and the default is `15s`\. Valid values for `unit` include the following:
   + `ms` – milliseconds
   + `s` – seconds

1. Create the route using the following command:

   ```
   aws appmesh create-route --cli-input-json file://create-route-retry-policy.json
   ```

------
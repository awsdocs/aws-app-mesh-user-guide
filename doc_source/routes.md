# Routes<a name="routes"></a>

A route is associated with a virtual router\. It is used to match requests for the virtual router and distribute traffic to its associated virtual nodes\. If a route matches a request, it can distribute traffic to one or more target virtual nodes\. You can specify relative weighting for each virtual node\. This topic helps you work with routes in a service mesh\.

## Creating a Route<a name="create-route"></a>

To create a virtual node, select the tool that you want to use to create it\. To create a route using the AWS CLI version 1\.16\.235 or higher, see the example in the AWS CLI reference for the [https://docs.aws.amazon.com/cli/latest/reference/appmesh/create-route.html](https://docs.aws.amazon.com/cli/latest/reference/appmesh/create-route.html) command\.

### AWS Management Console<a name="console"></a>

1. Open the App Mesh console at [https://console\.aws\.amazon\.com/appmesh/](https://console.aws.amazon.com/appmesh/)\.

1. Choose the mesh that you want to create the route in\.

1. Choose **Virtual routers** in the left navigation\.

1. Choose the virtual router that you want to associate a new route with\. 

1. In the **Routes** table, choose **Create route**\.

1. For **Route name**, specify the name to use for your route\.

1. For **Route type**, choose the protocol for your route\. The protocol that you select must match the listener protocol that you selected for your virtual router and the virtual node that you're routing traffic to\.

1. \(Optional\) For **Route priority**, specify a priority from 0\-1000 to use for your route\. Routes are matched based on the specified value, where 0 is the highest priority\.

1. For **Virtual node name**, choose the virtual node that this route will serve traffic to\. If none are listed, then you need to [create a virtual node](https://docs.aws.amazon.com//app-mesh/latest/userguide/virtual_nodes.html) first\.

1. For **Weight**, choose a relative weight for the route\. Select **Add target** to add additional virtual nodes\. The total weight for all targets combined must be less than or equal to 100\.

1. \(Optional\) To use HTTP path and header\-based routing, choose **Additional configuration**\. 

1. \(Optional\) To use HTTP path\-based routing, specify the **Prefix** that the route should match\. For additional information about path\-based routing, see [Path\-based Routing](https://docs.aws.amazon.com//app-mesh/latest/userguide/route-path.html)\. 

1. \(Optional\) Select a **Method** to use header\-based routing for your route\. For additional information about HTTP header\-based routing, see [HTTP Headers](https://docs.aws.amazon.com//app-mesh/latest/userguide/route-http-headers.html)\. 

1. \(Optional\) Select a **Scheme** to use header\-based routing for your route\. 

1. \(Optional\) Select **Add header**\. Enter the **Header name** that you want to route based on, select a **Match type**, and enter a **Match value**\. Selecting **Invert** will match the opposite\. 

1. \(Optional\) Select **Add header** to add up to ten headers\. 

1. \(Optional\) For **Retry timeout**, enter the number of units for the timeout duration\. For additional information about retry policy, see [Retry Policy](https://docs.aws.amazon.com//app-mesh/latest/userguide/route-retry-policy.html)\. 

1. \(Optional\) For **Retry timeout unit**, select a unit\.

1. \(Optional\) For **Max retries**, enter the number of times to retry the route when an attempt fails\.

1. \(Optional\) Select one or more **HTTP retry events**\.

1. \(Optional\) Select a **TCP retry event**\.

1. Choose **Create route** to finish\.

### AWS CLI<a name="cli"></a>

To create a route with released features using the AWS CLI version 1\.16\.235 or higher, see the examples in the AWS CLI reference for the [create\-route](https://docs.aws.amazon.com/cli/latest/reference/appmesh/create-route.html) command\.

\([App Mesh Preview Channel](https://docs.aws.amazon.com//app-mesh/latest/userguide/preview.html) only\) To create an HTTP2 or GRPC route, complete the following steps\.

1. Download the Preview Channel service model with the following command\.

   ```
   curl -o appmesh-preview-channel-service-model.json https://raw.githubusercontent.com/aws/aws-app-mesh-roadmap/master/appmesh-preview/service-model.json
   ```

1. Add the Preview Channel service model to the AWS CLI with the following command\.

   ```
   aws configure add-model \
       --service-name appmesh-preview \
       --service-model file://appmesh-preview-channel-service-model.json
   ```

1. Create a JSON file named `create-route.json` using one of the following examples\. You can create an HTTP2 or GRPC route\. Each example includes a retry policy\. To learn more about retry policies, see [Retry policy](route-retry-policy.md)\.  
**Example HTTP2 route**  

   Other than using a different protocol, an HTTP2 route is identical to an HTTP route\.

   ```
   {
      "meshName" : "app1",
      "routeName" : "http2Route",
      "spec" : {
         "http2Route" : {
            "action" : {
               "weightedTargets" : [
                  {
                     "virtualNode" : "serviceB-http2",
                     "weight" : 100
                  }
               ]
            },
            "match" : {
               "prefix" : "/"
            },
            "retryPolicy" : {
               "httpRetryEvents" : [ "server-error" ],
               "maxRetries" : 3,
               "perRetryTimeout" : {
                  "unit" : "s",
                  "value" : 8
               }
            }
         }
      },
      "virtualRouterName" : "serviceB-http2"
   }
   ```  
**Example GRPC route**  

   A GRPC route requires you to specify the destination `serviceName` and `methodName` that the request must mach\. If no `serviceName` or `methodName` are specified, than all requests are routed\. The following example routes traffic destined only for the *GetColor* method of the *com\.amazonaws\.services\.ColorService* service\.

   ```
   {
      "meshName" : "app1",
      "routeName" : "grpcRoute",
      "spec" : {
         "grpcRoute" : {
            "action" : {
               "weightedTargets" : [
                  {
                     "virtualNode" : "serviceB-grpc",
                     "weight" : 100
                  }
               ]
            },
            "match" : {
               "methodName" : "GetColor",
               "serviceName" : "com.amazonaws.services.ColorService"
            },
            "retryPolicy" : {
               "grpcRetryEvents" : [ "deadline-exceeded" ],
               "maxRetries" : 3,
               "perRetryTimeout" : {
                  "unit" : "s",
                  "value" : 8
               }
            }
         }
      },
      "virtualRouterName" : "serviceB-grpc"
   }
   ```

   Valid values for `grpcRetryEvents` are: `cancelled`, `deadline-exceeded`, `internal`, `resource-exhausted`, and `unavailable`\. 

1. Create the route with the following command\.

   ```
   aws appmesh-preview create-route --cli-input-json file://create-route.json
   ```

## Deleting a Route<a name="delete-route"></a>

To delete a route using the AWS Management Console, complete the following steps\. To delete a route using the AWS CLI version 1\.16\.235 or higher, see the example in the AWS CLI reference for the [https://docs.aws.amazon.com/cli/latest/reference/appmesh/delete-route.html](https://docs.aws.amazon.com/cli/latest/reference/appmesh/delete-route.html) command\.

1. Open the App Mesh console at [https://console\.aws\.amazon\.com/appmesh/](https://console.aws.amazon.com/appmesh/)\.

1. Choose the mesh that you want to delete a route from\.

1. Choose **Virtual routers** in the left navigation\.

1. Choose the router that you want to delete a route from\.

1. In the **Routes** table, choose the route that you want to delete and select **Delete**\.

1. In the confirmation box, type **delete** and then select **Delete**\.
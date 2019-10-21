# Path\-based Routing<a name="route-path"></a>

 To create a route with path\-based routing using the AWS Management Console, complete the following steps\. To create a route using the AWS CLI version 1\.16\.235 or higher, see the example in the AWS CLI reference for the [https://docs.aws.amazon.com/cli/latest/reference/appmesh/create-route.html](https://docs.aws.amazon.com/cli/latest/reference/appmesh/create-route.html) command\.

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

1. \(Optional\) To use HTTP path\-based routing, specify the **Prefix** that the route should match\. For example, if your virtual service name is `service-b.local` and you want the route to match requests to `service-b.local/metrics`, your prefix should be `/metrics`\.

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
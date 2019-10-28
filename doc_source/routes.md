# Routes<a name="routes"></a>

A route is associated with a virtual router\. The route is used to match requests for the virtual router and to distribute traffic to its associated virtual nodes\. If a route matches a request, it can distribute traffic to one or more target virtual nodes\. You can specify relative weighting for each virtual node\. This topic helps you work with routes in a service mesh\.

## Creating a Route<a name="create-route"></a>

To create a route using the AWS Management Console, complete the following steps\. To create a route using the AWS CLI version 1\.16\.266 or later, see the examples in the AWS CLI reference for the [create\-route](https://docs.aws.amazon.com/cli/latest/reference/appmesh/create-route.html) command\.

1. Open the App Mesh console at [https://console\.aws\.amazon\.com/appmesh/](https://console.aws.amazon.com/appmesh/)\.

1. Choose the mesh that you want to create the route in\.

1. Choose **Virtual routers** in the left navigation\.

1. Choose the virtual router that you want to associate a new route with\. If none are listed, then you need to [create a virtual router](https://docs.aws.amazon.com//app-mesh/latest/userguide/virtual_routers.html) first\.

1. In the **Routes** table, choose **Create route**\.

1. For **Route name**, specify the name to use for your route\.

1. For **Route type**, choose the protocol that you want to route\. The protocol that you select must match the listener protocol that you selected for your virtual router and the virtual node that you're routing traffic to\.

1. Select the protocol that you want to route, enter or select the values that appear, and then select **Create route**\.

### gRPC<a name="grpc"></a>

### Route configuration
+ \(Optional\) For **Route priority**, specify a priority from 0\-1000 to use for your route\. Routes are matched based on the specified value, where 0 is the highest priority\.

### Targets
+ For **Virtual node name**, choose the virtual node that this route will serve traffic to\. If none are listed, then you need to [create a virtual node](https://docs.aws.amazon.com//app-mesh/latest/userguide/virtual_nodes.html) first\.
+ For **Weight**, choose a relative weight for the route\. Select **Add target** to add additional virtual nodes\. The total weight for all targets combined must be less than or equal to 100\.
+ Choose **Additional configuration**\.

### Match
+ \(Optional\) Enter the **Service name** of the destination service to match the request for\. If you don't specify a name, requests to any service are matched\. 
+ \(Optional\) Enter the **Method name** of the destination method to match the request for\. If you don't specify a name, requests to any method are matched\. If you specify a method name, you must specify a service name\.

### Metadata
+ \(Optional\) Enter the **Metadata name** that you want to route based on, select a **Match type**, and enter a **Match value**\. Selecting **Invert** will match the opposite\. For example, if you specify a **Metadata name** of `myMetadata`, a **Match type** of **Exact**, a **Match value** of `123`, and select **Invert**, then the route is matched for any request that has a metadata name that starts with anything other than `123`\.
+ \(Optional\) Select **Add metadata** to add up to ten metadata items\. 

### **Retry policy**

A retry policy enables clients to protect themselves from intermittent network failures or intermittent server\-side failures\. A retry policy is optional\. The retry timeout values define the duration of time between retry attempts\.
+ For **Retry timeout**, enter the number of units for the timeout duration\. A value is required if you select any protocol retry event\.
+ For **Retry timeout unit**, select a unit\. A value is required if you select any protocol retry event\.
+ For **Max retries**, enter the maximum number of retry attempts when the request fails\. A value is required if you select any protocol retry event\.
+ Select one or more **HTTP retry events**\.
+ Select a **TCP retry event**\.
+ Select one or more **gRPC retry events**\.

### HTTP and HTTP/2<a name="http-http2"></a>

### Route configuration
+ \(Optional\) For **Route priority**, specify a priority from 0\-1000 to use for your route\. Routes are matched based on the specified value, where 0 is the highest priority\.

### Targets
+ For **Virtual node name**, choose the virtual node that this route will serve traffic to\. If none are listed, then you need to [create a virtual node](https://docs.aws.amazon.com//app-mesh/latest/userguide/virtual_nodes.html) first\.
+ For **Weight**, choose a relative weight for the route\. Select **Add target** to add additional virtual nodes\. The total weight for all targets combined must be less than or equal to 100\.
+ Choose **Additional configuration**\.

### Match
+ Specify the **Prefix** that the route should match\. For example, if your virtual service name is `service-b.local` and you want the route to match requests to `service-b.local/metrics`, your prefix should be `/metrics`\. Specifying `/` routes all traffic\.
+ \(Optional\) Select a **Method**\. 
+ \(Optional\) Select a **Scheme**\. 

### Headers
+ \(Optional\) Select **Add header**\. Enter the **Header name** that you want to route based on, select a **Match type**, and enter a **Match value**\. Selecting **Invert** will match the opposite\. For example, if you specify a header named `clientRequestId` with a **Prefix** of `123`, and select **Invert**, then the route is matched for any request that has a header that starts with anything other than `123`\.
+ \(Optional\) Select **Add header**\. You can add up to ten headers\. 

### **Retry policy**

A retry policy enables clients to protect themselves from intermittent network failures or intermittent server\-side failures\. A retry policy is optional\. The retry timeout values define the duration of time between retry attempts\.
+ For **Retry timeout**, enter the number of units for the timeout duration\. A value is required if you select any protocol retry event\.
+ For **Retry timeout unit**, select a unit\. A value is required if you select any protocol retry event\.
+ For **Max retries**, enter the maximum number of retry attempts when the request fails\. A value is required if you select any protocol retry event\.
+ Select one or more **HTTP retry events**\.
+ Select a **TCP retry event**\.

### TCP<a name="tcp"></a>

### Route configuration
+ \(Optional\) For **Route priority**, specify a priority from 0\-1000 to use for your route\. Routes are matched based on the specified value, where 0 is the highest priority\.

### Targets
+ For **Virtual node name**, choose the virtual node that this route will serve traffic to\. If none are listed, then you need to [create a virtual node](https://docs.aws.amazon.com//app-mesh/latest/userguide/virtual_nodes.html) first\.
+ For **Weight**, choose a relative weight for the route\. Select **Add target** to add additional virtual nodes\. The total weight for all targets combined must be less than or equal to 100\.

## Deleting a Route<a name="delete-route"></a>

To delete a route using the AWS Management Console, complete the following steps\. To delete a route using the AWS CLI version 1\.16\.266 or higher, see the example in the AWS CLI reference for the [https://docs.aws.amazon.com/cli/latest/reference/appmesh/delete-route.html](https://docs.aws.amazon.com/cli/latest/reference/appmesh/delete-route.html) command\.

1. Open the App Mesh console at [https://console\.aws\.amazon\.com/appmesh/](https://console.aws.amazon.com/appmesh/)\.

1. Choose the mesh that you want to delete a route from\.

1. Choose **Virtual routers** in the left navigation\.

1. Choose the router that you want to delete a route from\.

1. In the **Routes** table, choose the route that you want to delete and select **Delete**\.

1. In the confirmation box, type **delete** and then select **Delete**\.
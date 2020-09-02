# Routes<a name="routes"></a>

A route is associated with a virtual router\. The route is used to match requests for the virtual router and to distribute traffic to its associated virtual nodes\. If a route matches a request, it can distribute traffic to one or more target virtual nodes\. You can specify relative weighting for each virtual node\. This topic helps you work with routes in a service mesh\.

## Creating a route<a name="create-route"></a>

To create a route using the AWS CLI version 1\.18\.116 or later, see the examples in the AWS CLI reference for the [create\-route](https://docs.aws.amazon.com/cli/latest/reference/appmesh/create-route.html) command\.

**To create a route using the AWS Management Console**

1. Open the App Mesh console at [https://console\.aws\.amazon\.com/appmesh/](https://console.aws.amazon.com/appmesh/)\. 

1. Choose the mesh that you want to create the route in\. All of the meshes that you own and that have been [shared](sharing.md) with you are listed\.

1. Choose **Virtual routers** in the left navigation\.

1. Choose the virtual router that you want to associate a new route with\. If none are listed, then you need to [create a virtual router](virtual_routers.md#create-virtual-router) first\.

1. In the **Routes** table, choose **Create route**\. To create a route, your account ID must be listed as the **Resource owner** of the route\.

1. For **Route name**, specify the name to use for your route\.

1. For **Route type**, choose the protocol that you want to route\. The protocol that you select must match the listener protocol that you selected for your virtual router and the virtual node that you're routing traffic to\.

1. \(Optional\) For **Route priority**, specify a priority from 0\-1000 to use for your route\. Routes are matched based on the specified value, where 0 is the highest priority\.

1. For **Virtual node name**, select the existing App Mesh virtual node to route traffic to and specify a **Weight**\. You can choose **Add target** to add additional targets\. The percentage for all targets must add up to 100\. If no virtual nodes are listed, then you must [create](virtual_nodes.md#vn-create-virtual-node) one first\.

1. \(Optional\) Choose **Additional configuration**\. From the following protocols, choose the protocol that you selected for **Route type**, specify settings in the console as desired, and then select **Create route**\.

### gRPC<a name="grpc"></a>

### \(Optional\) **Match**
+ \(Optional\) Enter the **Service name** of the destination service to match the request for\. If you don't specify a name, requests to any service are matched\.
+ \(Optional\) Enter the **Method name** of the destination method to match the request for\. If you don't specify a name, requests to any method are matched\. If you specify a method name, you must specify a service name\.

### \(Optional\) **Metadata**

Choose **Add metadata**\.
+ \(Optional\) Enter the **Metadata name** that you want to route based on, select a **Match type**, and enter a **Match value**\. Selecting **Invert** will match the opposite\. For example, if you specify a **Metadata name** of `myMetadata`, a **Match type** of **Exact**, a **Match value** of `123`, and select **Invert**, then the route is matched for any request that has a metadata name that starts with anything other than `123`\.
+ \(Optional\) Select **Add metadata** to add up to ten metadata items\. 

### \(Optional\) **Retry policy**

A retry policy enables clients to protect themselves from intermittent network failures or intermittent server\-side failures\. A retry policy is optional, but recommended\. The retry timeout values define the duration of time between retry attempts\. If you don't define a retry policy, then App Mesh may automatically create a default policy for each of your routes\. For more information, see [Default route retry policy](envoy.md#default-retry-policy)\.
+ For **Retry timeout**, enter the number of units for the timeout duration\. A value is required if you select any protocol retry event\.
+ For **Retry timeout unit**, select a unit\. A value is required if you select any protocol retry event\.
+ For **Max retries**, enter the maximum number of retry attempts when the request fails\. A value is required if you select any protocol retry event\. We recommend a value of at least two\.
+ Select one or more **HTTP retry events**\. We recommend selecting at least **stream\-error** and **gateway\-error**\.
+ Select a **TCP retry event**\.
+ Select one or more **gRPC retry events**\. We recommend selecting at least **cancelled** and **unavailable**\.

### **\(Optional\) Timeouts**
+ **Request timeout** – The default is 15 seconds\. If you specified a **Retry policy**, then the duration that you specify here should always be greater than or equal to the retry duration multiplied by the **Max retries** that you defined in the **Retry policy** so that your retry policy can complete\. If you specify a duration greater than 15 seconds, then make sure that the timeout specified for the listener of any virtual node **Target** is also greater than 15 seconds\. For more information, see [Virtual nodes](virtual_nodes.md)\.
+ **Idle duration** – The maximum amount of time that the route can be idle\.

### HTTP and HTTP/2<a name="http-http2"></a>

### \(Optional\) Match
+ Specify the **Prefix** that the route should match\. For example, if your virtual service name is `service-b.local` and you want the route to match requests to `service-b.local/metrics`, your prefix should be `/metrics`\. Specifying `/` routes all traffic\.
+ \(Optional\) Select a **Method**\. 
+ \(Optional\) Select a **Scheme**\. 

### \(Optional\) Headers
+ \(Optional\) Select **Add header**\. Enter the **Header name** that you want to route based on, select a **Match type**, and enter a **Match value**\. Selecting **Invert** will match the opposite\. For example, if you specify a header named `clientRequestId` with a **Prefix** of `123`, and select **Invert**, then the route is matched for any request that has a header that starts with anything other than `123`\.
+ \(Optional\) Select **Add header**\. You can add up to ten headers\. 

### **\(Optional\) Retry policy**

A retry policy enables clients to protect themselves from intermittent network failures or intermittent server\-side failures\. A retry policy is optional, but recommended\. The retry timeout values define the duration of time between retry attempts\. If you don't define a retry policy, then App Mesh may automatically create a default policy for each of your routes\. For more information, see [Default route retry policy](envoy.md#default-retry-policy)\.
+ For **Retry timeout**, enter the number of units for the timeout duration\. A value is required if you select any protocol retry event\.
+ For **Retry timeout unit**, select a unit\. A value is required if you select any protocol retry event\.
+ For **Max retries**, enter the maximum number of retry attempts when the request fails\. A value is required if you select any protocol retry event\. We recommend a value of at least two\.
+ Select one or more **HTTP retry events**\. We recommend selecting at least **stream\-error** and **gateway\-error**\.
+ Select a **TCP retry event**\.

### **\(Optional\) Timeouts**
+ **Request timeout** – The default is 15 seconds\. If you specified a **Retry policy**, then the duration that you specify here should always be greater than or equal to the retry duration multiplied by the **Max retries** that you defined in the **Retry policy** so that your retry policy can complete\. If you specify a duration greater than 15 seconds, then make sure that the timeout specified for the listener of any virtual node **Target** is also greater than 15 seconds\. For more information, see [Virtual nodes](virtual_nodes.md)\.
+ **Idle duration** – The maximum amount of time that the route can be idle\.

### TCP<a name="tcp"></a>

### **\(Optional\) Timeouts**
+ **Idle duration** – The maximum amount of time that the route can be idle\.

## Deleting a route<a name="delete-route"></a>

To delete a route using the AWS CLI version 1\.18\.116 or higher, see the example in the AWS CLI reference for the [https://docs.aws.amazon.com/cli/latest/reference/appmesh/delete-route.html](https://docs.aws.amazon.com/cli/latest/reference/appmesh/delete-route.html) command\.

**To delete a route using the AWS Management Console**

1. Open the App Mesh console at [https://console\.aws\.amazon\.com/appmesh/](https://console.aws.amazon.com/appmesh/)\. 

1. Choose the mesh that you want to delete a route from\. All of the meshes that you own and that have been [shared](sharing.md) with you are listed\.

1. Choose **Virtual routers** in the left navigation\.

1. Choose the router that you want to delete a route from\.

1. In the **Routes** table, choose the route that you want to delete and select **Delete**\.

1. In the confirmation box, type **delete** and then select **Delete**\.
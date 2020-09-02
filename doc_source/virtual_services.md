# Virtual services<a name="virtual_services"></a>

A virtual service is an abstraction of a real service that is provided by a virtual node directly or indirectly by means of a virtual router\. Dependent services call your virtual service by its `virtualServiceName`, and those requests are routed to the virtual node or virtual router that is specified as the provider for the virtual service\.

## Creating a virtual service<a name="create-virtual-service"></a>

To create a virtual service using the AWS CLI version 1\.18\.116 or higher, see the example in the AWS CLI reference for the [create\-virtual\-service](https://docs.aws.amazon.com/cli/latest/reference/appmesh/create-virtual-service.html) command\. 

**To create a virtual service using the AWS Management Console**

1. Open the App Mesh console at [https://console\.aws\.amazon\.com/appmesh/](https://console.aws.amazon.com/appmesh/)\. 

1. Choose the mesh that you want to create the virtual service in\. All of the meshes that you own and that have been [shared](sharing.md) with you are listed\.

1. Choose **Virtual services** in the left navigation\.

1. Choose **Create virtual service**\.

1. For **Virtual service name**, choose a name for your virtual service\. You can choose any name, but the service discovery name of the real service that you're targeting, such as `my-service.default.svc.cluster.local`, is recommended to make it easier to correlate your virtual services to real services and so that you don't need to change your code to reference a different name than your code currently references\. The name that you specify must resolve to a non\-loopback IP address because the app container must be able to successfully resolve the name before the request is sent to the Envoy proxy\. You can use any non\-loopback IP address because neither the app or proxy containers communicate with this IP address\. The proxy communicates with other virtual services through the names you’ve configured for them in App Mesh, not through IP addresses that the names resolve to\.

1. For **Provider**, choose the provider type for your virtual service:
   + If you want the virtual service to spread traffic across multiple virtual nodes, select **Virtual router** and then choose the virtual router to use from the drop\-down menu\.
   + If you want the virtual service to reach a virtual node directly, without a virtual router, select **Virtual node** and then choose the virtual node to use from the drop\-down menu\.
**Note**  
App Mesh may automatically create a default Envoy route retry policy for each virtual node provider that you define on or after July 29, 2020, even though you can't define such a policy through the App Mesh API\. For more information, see [Default route retry policy](envoy.md#default-retry-policy)\.
   + If you don't want the virtual service to route traffic at this time \(for example, if your virtual nodes or virtual router doesn't exist yet\), choose **None**\. You can update the provider for this virtual service later\.

1. Choose **Create virtual service** to finish\.
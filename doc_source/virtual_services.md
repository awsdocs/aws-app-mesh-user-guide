# Virtual Services<a name="virtual_services"></a>

A virtual service is an abstraction of a real service that is provided by a virtual node directly or indirectly by means of a virtual router\. Dependent services call your virtual service by its `virtualServiceName`, and those requests are routed to the virtual node or virtual router that is specified as the provider for the virtual service\.

## Creating a Virtual Service<a name="create-virtual-service"></a>

This topic helps you to create a virtual service in your service mesh\.

**Creating a virtual service in the AWS Management Console\.**

1. Open the App Mesh console at [https://console\.aws\.amazon\.com/appmesh/](https://console.aws.amazon.com/appmesh/)\.

1. Choose the mesh that you want to create the virtual service in\.

1. Choose **Virtual services** in the left navigation\.

1. Choose **Create virtual service**\.

1. For **Virtual service name**, choose a name for your virtual service\. You can choose any name, but the service discovery name of the real service that you're targeting, such as `my-service.default.svc.cluster.local`, is recommended to make it easier to correlate your virtual services to real services\. The name that you specify must resolve to a non\-loopback IP address because the app container must be able to successfully resolve the name before the request is sent to the Envoy proxy\. You can use any non\-loopback IP address because neither the app or proxy containers communicate with this IP address\. The proxy communicates with other virtual services through the names you’ve configured for them in App Mesh, not through IP addresses that the names resolve to\.

1. For **Provider**, choose the provider type for your virtual service:
   + If you want the virtual service to spread traffic across multiple virtual nodes, select **Virtual router** and then choose the virtual router to use from the drop\-down menu\.
   + If you want the virtual service to reach a virtual node directly, without a virtual router, select **Virtual node** and then choose the virtual node to use from the drop\-down menu\.
   + If you don't want the virtual service to route traffic at this time \(for example, if your virtual nodes or virtual router doesn't exist yet\), choose **None**\. You can update the provider for this virtual service later\.

1. Choose **Create virtual service** to finish\.

**Creating a virtual service in the AWS CLI\.**
+ The following JSON represents a virtual service named `serviceB` that is provided by the virtual router named `serviceB`\.

  ```
  {
      "meshName": "simpleapp",
      "spec": {
          "provider": {
              "virtualRouter": {
                  "virtualRouterName": "serviceB"
              }
          }
      },
      "virtualServiceName": "serviceB"
  }
  ```

  If you save the preceding JSON as a file, you can create the virtual service with the following AWS CLI command\.

  ```
  aws appmesh create-virtual-service --cli-input-json file://serviceB-virtual-service.json
  ```
**Note**  
You must use at least version 1\.16\.235 of the AWS CLI with App Mesh\. To install the latest version of the AWS CLI, see [Installing the AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/installing.html) in the *AWS Command Line Interface User Guide*\.

  For more information about creating virtual services with the AWS CLI, see [create\-virtual\-service](https://docs.aws.amazon.com/cli/latest/reference/appmesh/create-virtual-service.html) in the *AWS Command Line Interface User Guide*\.
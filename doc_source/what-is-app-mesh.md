# What Is AWS App Mesh?<a name="what-is-app-mesh"></a>

AWS App Mesh is a service mesh that makes it easy to monitor and control services\. App Mesh standardizes how your services communicate, giving you end\-to\-end visibility and helping to ensure high availability for your applications\. App Mesh gives you consistent visibility and network traffic controls for every service in an application\. 

## Adding App Mesh to an example application<a name="example-application"></a>

Consider the following simple example application, that doesn’t use App Mesh\. The two services can be running on AWS Fargate, Amazon Elastic Container Service \(Amazon ECS\), Amazon Elastic Kubernetes Service \(Amazon EKS\), Kubernetes on Amazon Elastic Compute Cloud \(Amazon EC2\) instances, or on Amazon EC2 instances with Docker\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/app-mesh/latest/userguide/images/simple-app-diagram.png)

In this illustration, both `serviceA` and `serviceB` are discoverable through the `apps.local` namespace\. Let's say, for example, you decide to deploy a new version of `serviceb.apps.local` named `servicebv2.apps.local`\. Next, you want to direct a percentage of the traffic from `servicea.apps.local` to `serviceb.apps.local` and a percentage to `servicebv2.apps.local`\. When you're sure that `servicebv2` is performing well, you want to send 100 percent of the traffic to it\.

 App Mesh can help you do this without changing any application code or registered service names\. If you use App Mesh with this example application, then your mesh might look like the following illustration\. 

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/app-mesh/latest/userguide/images/simple-app-with-mesh-diagram.png)

In this configuration, the services no longer communicate with each other directly\. Instead, they communicate with each other through a proxy\. The proxy deployed with the `servicea.apps.local` service reads the App Mesh configuration and sends traffic to `serviceb.apps.local` or `servicebv2.apps.local`, based on the configuration\.

## Components of App Mesh<a name="app_mesh_components"></a>

App Mesh is made up of the following components, illustrated in the previous example:
+ **Service mesh** – A service mesh is a logical boundary for network traffic between the services that reside within it\. In the example, the mesh is named `apps`, and it contains all other resources for the mesh\. For more information, see [Service meshes](meshes.md)\.
+ **Virtual services** – A virtual service is an abstraction of an actual service that is provided by a virtual node, directly or indirectly, by means of a virtual router\. In the illustration, two virtual services represent the two actual services\. The names of the virtual services are the discoverable names of the actual services\. When a virtual service and an actual service have the same name, multiple services can communicate with each other using the same names that they used before App Mesh was implemented\. For more information, see [Virtual services](virtual_services.md)\.
+ **Virtual nodes** – A virtual node acts as a logical pointer to a discoverable service, such as an Amazon ECS or Kubernetes service\. For each virtual service, you will have at least one virtual node\. In the illustration, the `servicea.apps.local` virtual service gets configuration information for the virtual node named `serviceA`\. The `serviceA` virtual node is configured with the `servicea.apps.local` name for service discovery\. The `serviceb.apps.local` virtual service is configured to route traffic to the `serviceB` and `serviceBv2` virtual nodes through a virtual router named `serviceB`\. For more information, see [Virtual nodes](virtual_nodes.md)\.
+ **Virtual routers and routes** – Virtual routers handle traffic for one or more virtual services within your mesh\. A route is associated to a virtual router\. The route is used to match requests for the virtual router and to distribute traffic to its associated virtual nodes\. In the previous illustration, the `serviceB` virtual router has a route that directs a percentage of traffic to the `serviceB` virtual node, and a percentage of traffic to the `serviceBv2` virtual node\. You can set the percentage of traffic routed to a particular virtual node and change it over time\. You can route traffic based on criteria such as HTTP headers, URL paths, or gRPC service and method names\. You can configure retry policies to retry a connection if there is an error in the response\. For example, in the illustration, the retry policy for the route can specify that a connection to `serviceb.apps.local` is retried five times, with ten seconds between retry attempts, if `serviceb.apps.local` returns specific types of errors\. For more information, see [Virtual routers](virtual_routers.md) and [Routes](routes.md)\.
+ **Proxy** – You configure your services to use the proxy after you create your mesh and its resources\. The proxy reads the App Mesh configuration and directs traffic appropriately\. In the illusration, all communication from `servicea.apps.local` to `serviceb.apps.local` goes through the proxy deployed with each service\. The services communicate with each other using the same service discovery names that they used before introducing App Mesh\. Because the proxy reads the App Mesh configuration, you can control how the two services communicate with each other\. When you want change the App Mesh configuration, you don’t need to change or redeploy the services themselves or the proxies\. For more information, see [Envoy image](envoy.md)\.

## How to get started<a name="how_to_get_started"></a>

To use App Mesh you must have an existing service running on AWS Fargate, Amazon ECS, Amazon EKS, Kubernetes on Amazon EC2, or Amazon EC2 with Docker\.

To get started with App Mesh, see one of the following guides:
+ [Getting Started with App Mesh and Amazon ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/appmesh-getting-started.html)
+ [Getting Started with App Mesh and Kubernetes](https://docs.aws.amazon.com/eks/latest/userguide/appmesh-getting-started.html)
+ [Getting Started with App Mesh and Amazon EC2](https://docs.aws.amazon.com/app-mesh/latest/userguide/appmesh-getting-started.html)

## Accessing App Mesh<a name="accessing_app_mesh"></a>

You can work with App Mesh in the following ways:

**AWS Management Console**  
The console is a browser\-based interface that you can use to manage App Mesh resources\. You can open the App Mesh console at [https://console\.aws\.amazon\.com/appmesh/](https://console.aws.amazon.com/appmesh)\.

**AWS CLI**  
Provides commands for a broad set of AWS products, and is supported on Windows, Mac, and Linux\. To get started, see [AWS Command Line Interface User Guide](https://docs.aws.amazon.com/cli/latest/userguide/)\. For more information about the commands for App Mesh, see [appmesh](https://docs.aws.amazon.com/cli/latest/reference/appmesh/index.html#) in the [AWS CLI Command Reference](https://docs.aws.amazon.com/cli/latest/reference/)\.

**AWS Tools for Windows PowerShell**  
Provides commands for a broad set of AWS products for those who script in the PowerShell environment\. To get started, see the [AWS Tools for Windows PowerShell User Guide](https://docs.aws.amazon.com/powershell/latest/userguide/pstools-welcome.html)\. For more information about the cmdlets for App Mesh, see [App Mesh](https://docs.aws.amazon.com/powershell/latest/reference/items/AppMesh_cmdlets.html) in the [AWS Tools for PowerShell Cmdlet Reference](https://docs.aws.amazon.com/powershell/latest/reference/index.html)\.

**AWS CloudFormation**  
Enables you to create a template that describes all of the AWS resources that you want\. Using the template, AWS CloudFormation provisions and configures the resources for you\. To get started, see [AWS CloudFormation User Guide](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/)\. For more information about the App Mesh resource types, see [App Mesh Resource Type Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/AWS_AppMesh.html) in the [AWS CloudFormation Template Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-reference.html)\.

**AWS SDKs**  
We also provide SDKs that enable you to access App Mesh from a variety of programming languages\. The SDKs automatically take care of tasks such as:  
+ Cryptographically signing your service requests
+ Retrying requests
+ Handling error responses
For more information about available SDKs, see [Tools for Amazon Web Services](https://aws.amazon.com/tools/)\.  
For more information about the App Mesh APIs, see the [AWS App Mesh API Reference](https://docs.aws.amazon.com/app-mesh/latest/APIReference/Welcome.html)\.
# What Is AWS App Mesh?<a name="what-is-app-mesh"></a>

AWS App Mesh is a service mesh based on the [Envoy](https://www.envoyproxy.io/) proxy that makes it easy to monitor and control microservices\. App Mesh standardizes how your microservices communicate, giving you end\-to\-end visibility and helping to ensure high\-availability for your applications\.

App Mesh gives you consistent visibility and network traffic controls for every microservice in an application\. 

App Mesh supports microservice applications that use service discovery naming for their components\. To use App Mesh, you must have an existing application running on AWS Fargate, Amazon ECS, Amazon EKS, Kubernetes on AWS, or Amazon EC2\.

## Components of App Mesh<a name="app_mesh_components"></a>

App Mesh is made up of the following components:
+ **Service mesh** – A service mesh is a logical boundary for network traffic between the services that reside within it\. For more information, see [Service Meshes](meshes.md)\.
+ **Virtual services** – A virtual service is an abstraction of a real service that is provided by a virtual node directly or indirectly by means of a virtual router\. For more information, see [Virtual Services](virtual_services.md)\.
+ **Virtual nodes** – A virtual node acts as a logical pointer to a particular task group, such as an ECS service or a Kubernetes deployment\. When you create a virtual node, you must specify the service discovery name for your task group\. For more information, see [Virtual Nodes](virtual_nodes.md)\.
+ **Envoy proxy** – The Envoy proxy configures your microservice task group to use the App Mesh service mesh traffic rules that you set up for your virtual routers and virtual nodes\. You add the Envoy container to your task group after you have created your virtual nodes, virtual routers, routes, and virtual services\. For more information, see [Envoy Image](envoy.md)\.
+ **Virtual routers** – The virtual router handles traffic for one or more virtual services within your mesh\. For more information, see [Virtual Routers](virtual_routers.md)\.
+ **Routes** – A route is associated with a virtual router, and it directs traffic that matches a service name prefix to one or more virtual nodes\. For more information, see [Routes](routes.md)\.

## How to Get Started<a name="how_to_get_started"></a>

App Mesh supports microservice applications that use service discovery naming for their components\. To use App Mesh, you must have an existing application running on AWS Fargate, Amazon ECS, Amazon EKS, Kubernetes on AWS, or Amazon EC2\.

For more information about service discovery on Amazon ECS, see [Service Discovery](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-discovery.html) in the *Amazon Elastic Container Service Developer Guide*\. Kubernetes `kube-dns` is supported\. For more information, see [DNS for Services and Pods](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/) in the Kubernetes documentation\.

To get started using the App Mesh console first\-run wizard, see [Getting Started](https://docs.aws.amazon.com//app-mesh/latest/userguide/appmesh-getting-started.html)\.

## Accessing App Mesh<a name="accessing_app_mesh"></a>

You can work with App Mesh in the following ways:

**AWS Management Console**  
The console is a browser\-based interface to manage App Mesh resources\. For a tutorial that guides you through the console, see [Getting Started](file://AWSShared/ec2-container-shared/mesh-gs.xml)\.

**AWS command line tools**  
You can use the AWS command line tools to issue commands at your system's command line to perform App Mesh and AWS tasks\. This can be faster and more convenient than using the console\. The command line tools are also useful for building scripts that perform AWS tasks\.  
AWS provides two sets of command line tools: the [AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/) \(AWS CLI\) and the [AWS Tools for Windows PowerShell](https://docs.aws.amazon.com/powershell/latest/userguide/)\. For more information, see the [AWS Command Line Interface User Guide](https://docs.aws.amazon.com/cli/latest/userguide/) and the [AWS Tools for Windows PowerShell User Guide](https://docs.aws.amazon.com/powershell/latest/userguide/)\.

**AWS SDKs**  
We also provide SDKs that enable you to access App Mesh from a variety of programming languages\. The SDKs automatically take care of tasks such as:  
+ Cryptographically signing your service requests
+ Retrying requests
+ Handling error responses
For more information about available SDKs, see [Tools for Amazon Web Services](https://aws.amazon.com/tools/)\.
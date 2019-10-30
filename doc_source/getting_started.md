# Getting Started with AWS App Mesh<a name="getting_started"></a>

This topic helps you to get started using AWS App Mesh with the AWS Management Console\. This getting\-started guide helps you to create one of each component that is available in App Mesh\. When you're finished with the wizard, you can create additional resources in the AWS Management Console to represent the other services that you want to contain in the mesh\.

**Topics**
+ [Prerequisites](#gs_prerequisites)
+ [Step 1: Create a Mesh and Virtual Service](#gs_create_mesh_service)
+ [Step 2: Configure a Virtual Node](#gs_create_virtual_node)
+ [Step 3: Configure a Virtual Router and Route](#gs_configure_virtual_router)
+ [Step 4: Review and Create](#gs_review_and_create)
+ [Step 5: Create Your Remaining App Mesh Resources](#gs_create_remaining_resources)
+ [Step 6: Update Your Microservices](#gs_update_microservices)

## Prerequisites<a name="gs_prerequisites"></a>

App Mesh supports microservice applications that use service discovery naming for their components\. To use App Mesh, you must have an existing application running on AWS Fargate, Amazon ECS, Amazon EKS, Kubernetes on AWS, or Amazon EC2\.

For more information about service discovery on Amazon ECS, see [Service Discovery](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-discovery.html) in the *Amazon Elastic Container Service Developer Guide*\. Kubernetes `kube-dns` and `coredns` are supported\. For more information, see [DNS for Services and Pods](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/) in the Kubernetes documentation\.

## Step 1: Create a Mesh and Virtual Service<a name="gs_create_mesh_service"></a>

To begin, you must first create a service mesh and a virtual service for one of the microservices in your application\.

A service mesh is a logical boundary for network traffic between the services that reside within it\. A virtual service represents a service that can be used by virtual nodes in the mesh\. Dependent nodes call your virtual service by its virtual service name, and those requests are routed to the virtual node or virtual router that is specified as the provider for the virtual service\.

**To create a new service mesh and virtual service**

1. Open the App Mesh console first\-run wizard at [https://console\.aws\.amazon\.com/appmesh/get\-started](https://console.aws.amazon.com/appmesh/get-started)\.

1. For **Mesh name**, specify a name for your service mesh\. 

1. For **Virtual service name**, choose a name for your virtual service\. We recommend that you use the service discovery name of the real service that you're targeting \(such as `service-a.default.svc.cluster.local`\)\. The name that you specify must resolve to a non\-loopback IP address\.

1. Choose **Next** to proceed\.

## Step 2: Configure a Virtual Node<a name="gs_create_virtual_node"></a>

In this step, you configure a virtual node to be the end destination for requests that are made to the virtual service that you created earlier\.

A virtual node acts as a logical pointer to a particular task group, such as an Amazon ECS service or a Kubernetes deployment\. When you create a virtual node, you must specify a service discovery method for your task group\.

Any inbound traffic that your virtual node expects should be specified as a *listener*; you can optionally configure health checks for your virtual node listeners\. Any outbound traffic that your virtual node expects to reach should be specified as a *backend*\.

**To configure a virtual node**

1. For **Virtual node name**, enter a name for your virtual node\. 

1. For **Service discovery method**, choose one of the following options:
   + **DNS** – Specify the DNS\-registered hostname of the actual service that the virtual node represents\. For additional information about using DNS as a service discovery method, see [Virtual Nodes](https://docs.aws.amazon.com//app-mesh/latest/userguide/virtual_nodes.html)\. 
   + **AWS Cloud Map** – Specify the service name and namespace\. Optionally, you can also specify attributes that App Mesh can query AWS Cloud Map for\. Only instances that match all of the specified key/value pairs will be returned\. To use AWS Cloud Map, your account must have the `AWSServiceRoleForAppMesh` [service\-linked role](using-service-linked-roles.md)\. 
   + **None** – Select if your virtual node doesn't expect any inbound traffic\. 

1. To specify any backends \(for egress traffic\) for your virtual node, or to configure inbound and outbound access logging information, choose **Additional configuration**\.

   1. To specify a backend, choose **Add backend** and enter a virtual service name or full Amazon Resource Name \(ARN\) for the virtual service that your virtual node communicates with\. Repeat this step until all of your virtual node backends are accounted for\. 

   1. To configure logging, enter the HTTP access logs path that you want Envoy to use\. We recommend the `/dev/stdout` path so that you can use Docker log drivers to export your Envoy logs to a service such as Amazon CloudWatch Logs\. 
**Note**  
Logs must still be ingested by an agent in your application and sent to a destination\. This file path only instructs Envoy where to send the logs\. 

1. If your virtual node expects ingress traffic, specify a **Port** and **Protocol** for the **Listener**\. 

1. If you want to configure a health check for your listener, ensure that **Health check enabled** is selected and then complete the following substeps\. If not, clear this check box\. A health check policy is optional, but if you specify any values for a health policy, then you must specify values for **Healthy threshold**, **Health check interval**, **Health check protocol**, **Timeout period**, and **Unhealthy threshold**\.

   1. For **Health check protocol**, choose a protocol\. If you select **grpc**, then your service must conform to the [GRPC Health Checking Protocol](https://github.com/grpc/grpc/blob/master/doc/health-checking.md)\. 

   1. For **Health check port**, specify the port that the health check should run on\. 

   1. For **Healthy threshold**, specify the number of consecutive successful health checks that must occur before declaring the listener healthy\. 

   1. For **Health check interval**, specify the time period in milliseconds between each health check execution\. 

   1. For **Path**, specify the destination path for the health check request\. This value is only used if the **Health check protocol** is `http` or `http2`\. The value is ignored for other protocols\. 

   1. For **Timeout period**, specify the amount of time to wait when receiving a response from the health check, in milliseconds\. 

   1. For **Unhealthy threshold**, specify the number of consecutive failed health checks that must occur before declaring the listener unhealthy\. 

1. Chose **Next** to continue\.

## Step 3: Configure a Virtual Router and Route<a name="gs_configure_virtual_router"></a>

In this step, you configure a virtual router to handle requests that are made to the virtual service that you created earlier\. You also create a route that is used to match requests and distribute traffic accordingly to its associated virtual node\.

**To configure a virtual router and route**

1. For **Virtual router name**, specify a name for your virtual router\. Up to 255 letters, numbers, hyphens, and underscores are allowed\. 

1. For **Listener**, specify a **Port** and **Protocol** for your virtual router\. 

1. For **Route name**, specify the name to use for your route\.

1. For **Route type**, choose the protocol that you want to route\.

1. \(Optional\) For **Route priority**, specify a priority from 0\-1000 to use for your route\. Routes are matched based on the specified value, where 0 is the highest priority\.

1. For **Virtual node name**, choose the virtual node that this route will serve traffic to\. 

1. For **Weight**, choose a relative weight for the route\. Select **Add target** to add additional virtual nodes\. The total weight for all targets combined must be less than or equal to 100\.

1. \(Optional\) To use HTTP path and header\-based routing, choose **Additional configuration**\. 

1. \(Optional\) To use HTTP path\-based routing, specify the **Prefix** that the route should match\. For additional information about path\-based routing, see [Path\-based Routing](https://docs.aws.amazon.com//app-mesh/latest/userguide/route-path.html)\. 

1. \(Optional\) Select a **Method**\. For additional information about HTTP header\-based routing, see [HTTP Headers](https://docs.aws.amazon.com//app-mesh/latest/userguide/route-http-headers.html)\. 

1. \(Optional\) Select a **Scheme**\. 

1. \(Optional\) Select **Add header**\. Enter the **Header name** that you want to route based on, select a **Match type**, and enter a **Match value**\. Selecting **Invert** will match the opposite\.

1. \(Optional\) Select **Add header**\. You can add up to ten headers\. 

1. Choose **Next** to proceed\.

## Step 4: Review and Create<a name="gs_review_and_create"></a>

Review your information and choose **Create mesh service** to finish\. You can now view each of your App Mesh resources in the console\.

## Step 5: Create Your Remaining App Mesh Resources<a name="gs_create_remaining_resources"></a>

Currently, you should have one of each App Mesh resource for your application\. Now you must create the remaining resources for your application\.

**To create your remaining App Mesh resources**

1. Create virtual nodes for each remaining microservice in your application\. For more information, see [Creating a Virtual Node](virtual_nodes.md#create-virtual-node)\.

1. Create virtual routers for each remaining microservice in your application\. For more information, see [Creating a Virtual Router](virtual_routers.md#create-virtual-router)\.

1. Create routes for each remaining microservice in your application\. For more information, see [Creating a Route](routes.md#create-route)\.

1. Create virtual services for each remaining microservice in your application\. For more information, see [Creating a Virtual Service](virtual_services.md#create-virtual-service)\.

## Step 6: Update Your Microservices<a name="gs_update_microservices"></a>

App Mesh is a service mesh based on the [Envoy](https://www.envoyproxy.io/) proxy\. After you create your service mesh, virtual nodes, virtual routers, routes, and virtual services, you must update your microservices to be compatible with App Mesh\.

To update your existing microservice application that is running on Amazon ECS, see [Getting Started with AWS App Mesh and Amazon ECS](mesh-getting-started-ecs.md)\.

To update your existing microservice application that is running on Kubernetes, see [Getting Started with AWS App Mesh and Kubernetes](mesh-getting-started-k8s.md)\.

To update your existing microservice application that is running on Amazon EC2, see [Getting Started with AWS App Mesh and Amazon EC2](mesh-getting-started-ec2.md)\.
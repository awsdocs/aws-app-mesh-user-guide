# Getting Started with AWS App Mesh<a name="getting_started"></a>

This topic helps you to use AWS App Mesh with an existing containerized microservice application\.

**Topics**
+ [Prerequisites](#gs_prerequisites)
+ [Step 1: Create a Service Mesh](#gs_create_service_mesh)
+ [Step 2: Create Virtual Nodes](#gs_create_virtual_nodes)
+ [Step 3: Create Virtual Routers](#gs_create_virtual_routers)
+ [Step 4: Create Routes](#gs_create_routes)
+ [Step 5: Update your Microservices](#gs_update_microservices)

## Prerequisites<a name="gs_prerequisites"></a>

App Mesh supports containerized microservice applications that use service discovery naming for their components\. To use App Mesh, you must have a containerized application running on Amazon EC2 instances, hosted in either Amazon ECS, Amazon EKS, or Kubernetes on AWS\.

**Note**  
AWS App Mesh is available in the following Regions at this time:  
**US West \(Oregon\)** \(`us-west-2`\)
**US East \(N\. Virginia\)** \(`us-east-1`\)
**US East \(Ohio\)** \(`us-east-2`\)
**EU \(Ireland\)** \(`eu-west-1`\)

For more information about service discovery on Amazon ECS, see [Service Discovery](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-discovery.html) in the *Amazon Elastic Container Service Developer Guide*\. Kubernetes `kube-dns` is supported\. For more information, see [DNS for Services and Pods](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/) in the Kubernetes documentation\.

App Mesh is not supported by the AWS Management Console during the preview\. You must use the AWS CLI or an AWS SDK to interact with App Mesh\. 

**Note**  
You must use at least version 1\.16\.65 of the AWS CLI with App Mesh\. To install the latest version of the AWS CLI, see [Installing the AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/installing.html) in the *AWS Command Line Interface User Guide*\.

## Step 1: Create a Service Mesh<a name="gs_create_service_mesh"></a>

To begin, you must first create a service mesh for your application\. A service mesh is a logical boundary for network traffic between the services that reside within it\.

The following example AWS CLI command creates a service mesh called `simpleapp`:

```
aws appmesh create-mesh --mesh-name simpleapp
```

For more information about creating service meshes with the AWS CLI, see [create\-mesh](https://docs.aws.amazon.com/cli/latest/reference/appmesh/create-mesh.html) in the *AWS Command Line Interface User Guide*\.

## Step 2: Create Virtual Nodes<a name="gs_create_virtual_nodes"></a>

Create virtual nodes for each of the microservices in your application\.

A virtual node acts as a logical pointer to a particular task group, such as an Amazon ECS service or a Kubernetes deployment\. When you create a virtual node, you must specify the DNS service discovery name for your task group\.

Any inbound traffic that your virtual node expects should be specified as a `listener`\. Any outbound traffic that your virtual node expects to reach should be specified as a `backend`\.

Your application will vary, but as an example, the following JSON represents a virtual node called `serviceA`\. This virtual node listens on port 8080, and it expects to send traffic to a service name within the mesh called `serviceB.simpleapp.local` as a backend\. The task group for the virtual node \(the Amazon ECS service with service discovery support, or Kubernetes deployment\) uses the `serviceA.simpleapp.local` hostname for service discovery\.

```
{
  "meshName": "simpleapp",
  "spec": {
    "backends": [
      "serviceB.simpleapp.local"
    ],
    "listeners": [
      {
        "portMapping": {
          "port": 8080,
          "protocol": "http"
        }
      }
    ],
    "serviceDiscovery": {
      "dns": {
        "serviceName": "serviceA.simpleapp.local"
      }
    }
  },
  "virtualNodeName": "serviceA"
}
```

If the above JSON is saved as a file, you can create the virtual node with the following AWS CLI command\.

```
aws appmesh create-virtual-node --cli-input-json file://serviceA.json
```

For more information about creating virtual nodes with the AWS CLI, see [create\-virtual\-node](https://docs.aws.amazon.com/cli/latest/reference/appmesh/create-virtual-node.html) in the *AWS Command Line Interface User Guide*\.

## Step 3: Create Virtual Routers<a name="gs_create_virtual_routers"></a>

Create a virtual router for each microservice's service name in your mesh\. For example, if your application has a frontend service and two backend services, you should create three virtual routers, one for each service\.

Each service name within the mesh must be fronted by a virtual router\. The service name you specify for the virtual router must be a real DNS service name within your VPC\. In most cases, you should just use the same service name that you specified for your virtual nodes\.

Your application will vary, but as an example, the following JSON represents a virtual router called `serviceB`, for the service name `serviceB.simpleapp.local`\. This virtual router represents the backend for the virtual node created earlier\.

```
{
  "meshName": "simpleapp",
  "spec": {
    "serviceNames": [
      "serviceB.simpleapp.local"
    ]
  },
  "virtualRouterName": "serviceB"
}
```

If the above JSON is saved as a file, you can create the virtual node with the following AWS CLI command\.

```
aws appmesh create-virtual-router --cli-input-json file://serviceB-router.json
```

For more information about creating virtual routers with the AWS CLI, see [create\-virtual\-router](https://docs.aws.amazon.com/cli/latest/reference/appmesh/create-virtual-router.html) in the *AWS Command Line Interface User Guide*\.

## Step 4: Create Routes<a name="gs_create_routes"></a>

Now that you have created your virtual nodes and virtual routers, create routes for your virtual routers that direct traffic to one or more virtual nodes\. 

A route is associated with a virtual router, and it is used to match requests for a virtual router and distribute traffic accordingly to its associated virtual nodes\.

You can use the `prefix` parameter in your route specification for path\-based routing of requests\. For example, if your virtual router service name is `my-service.local`, and you want the route to match requests to `my-service.local/metrics`, then your prefix should be `/metrics`\.

If your route matches a request, you can distribute traffic to one or more target virtual nodes with relative weighting\.

Your application will vary, but as an example, the following JSON represents a route called `serviceB-route`, for the virtual router `serviceB`\. This route directs traffic to two virtual nodes; 90% to `serviceBv1`, and 10% to `serviceBv2`\. This canary\-style deployment allows you to test a new version of your service with a small amount of traffic to decrease the blast radius of new service code\.

```
{
  "meshName": "simpleapp",
  "routeName": "serviceB-route",
  "spec": {
    "httpRoute": {
      "action": {
        "weightedTargets": [
          {
            "virtualNode": "serviceBv1",
            "weight": 90
          },
          {
            "virtualNode": "serviceBv2",
            "weight": 10
          }
        ]
      },
      "match": {
        "prefix": "/"
      }
    }
  },
  "virtualRouterName": "serviceB"
}
```

If the above JSON is saved as a file, you can create the route with the following AWS CLI command\.

```
aws appmesh create-route --cli-input-json file://serviceB-routes.json
```

For more information about creating routes with the AWS CLI, see [create\-route](https://docs.aws.amazon.com/cli/latest/reference/appmesh/create-route.html) in the *AWS Command Line Interface User Guide*\.

## Step 5: Update your Microservices<a name="gs_update_microservices"></a>

AWS App Mesh is a service mesh based on the [Envoy](https://www.envoyproxy.io/) proxy\. After you create your service mesh, virtual nodes, virtual routers, and routes, you must update your microservices to be compatible with App Mesh\.

App Mesh vends the following custom container images that you must add to your task groups\.
+ App Mesh Envoy container image: `111345817488.dkr.ecr.us-west-2.amazonaws.com/aws-appmesh-envoy:v1.8.0.2-beta`
+ App Mesh proxy route manager: `111345817488.dkr.ecr.us-west-2.amazonaws.com/aws-appmesh-proxy-route-manager:latest`

The following are example Amazon ECS task definition and Kubernetes pod specification snippets that you can merge with your existing task groups\. Substitute your mesh name and virtual node name for the `APPMESH_VIRTUAL_NODE_NAME` value, and a list of ports that your application listens on for the `APPMESH_APP_PORTS` value\. For the Kubernetes pod spec, substitute the Amazon EC2 instance AWS Region for the `AWS_REGION` value\.

**Example AWS CloudFormation for Amazon ECS task definition**  

```
- Name: "envoy"
  Image: "111345817488.dkr.ecr.us-west-2.amazonaws.com/aws-appmesh-envoy:v1.8.0.2-beta"
  Essential: true
  Environment:
    - Name: APPMESH_VIRTUAL_NODE_NAME
      Value: "mesh/meshName/virtualNode/virtualNodeName"
    - Name: ENVOY_LOG_LEVEL
      Value: info
  User: "1337"
- Name: "proxyinit"
  Image: "111345817488.dkr.ecr.us-west-2.amazonaws.com/aws-appmesh-proxy-route-manager:latest"
  Essential: false
  LinuxParameters:
    Capabilities:
      Add:
        - "NET_ADMIN"
  Environment:
    - Name: "APPMESH_START_ENABLED"
      Value: "1"
    - Name: "APPMESH_IGNORE_UID"
      Value: "1337"
    - Name: "APPMESH_ENVOY_INGRESS_PORT"
      Value: "15000"
    - Name: "APPMESH_ENVOY_EGRESS_PORT"
      Value: "15001"
    - Name: "APPMESH_APP_PORTS"
      Value: "application_port_list"
    - Name: "APPMESH_EGRESS_IGNORED_IP"
      Value: "169.254.169.254,169.254.170.2"
```

**Example JSON for Amazon ECS task definition**  

```
{
  "name": "envoy",
  "image": "111345817488.dkr.ecr.us-west-2.amazonaws.com/aws-appmesh-envoy:v1.8.0.2-beta",
  "essential": true,
  "environment": [
      {
        "name": "APPMESH_VIRTUAL_NODE_NAME",
        "value": "mesh/meshName/virtualNode/virtualNodeName"
      },
      {
        "name": "ENVOY_LOG_LEVEL",
        "value": "info"
      }
  ],
  "user": "1337",
},
{
  "name": "proxyinit",
  "image": "111345817488.dkr.ecr.us-west-2.amazonaws.com/aws-appmesh-proxy-route-manager:latest",
  "essential": false,
  "environment": [
      {
          "name": "APPMESH_START_ENABLED",
          "value": "1"
      },
      {
        "name": "APPMESH_IGNORE_UID",
        "value": "1337"
      },
      {
        "name": "APPMESH_ENVOY_INGRESS_PORT",
        "value": "15000"
      },
      {
        "name": "APPMESH_ENVOY_EGRESS_PORT",
        "value": "15001"
      },
      {
        "name": "APPMESH_APP_PORTS",
        "value": "application_port_list"
      },
      {
        "name": "APPMESH_EGRESS_IGNORED_IP",
        "value": "169.254.169.254,169.254.170.2"
      }
  ],
  "linuxParameters": {
      "capabilities": {
          "add": [
              "NET_ADMIN"
          ]
      },
  }
}
```

**Example Kubernetes pod spec**  

```
spec:
  containers:
    - name: envoy
      image: 111345817488.dkr.ecr.us-west-2.amazonaws.com/aws-appmesh-envoy:v1.8.0.2-beta
      securityContext:
        runAsUser: 1337
      env:
        - name: "APPMESH_VIRTUAL_NODE_NAME"
          value: "mesh/meshName/virtualNode/virtualNodeName"
        - name: "ENVOY_LOG_LEVEL"
          value: "info"
        - name: "AWS_REGION"
          value: "aws_region_name"
  initContainers:
    - name: proxyinit
      image: 111345817488.dkr.ecr.us-west-2.amazonaws.com/aws-appmesh-proxy-route-manager:latest
      securityContext:
        capabilities:
          add: 
            - NET_ADMIN
      env:
        - name: "APPMESH_START_ENABLED"
          value: "1"
        - name: "APPMESH_IGNORE_UID"
          value: "1337"
        - name: "APPMESH_ENVOY_INGRESS_PORT"
          value: "15000"
        - name: "APPMESH_ENVOY_EGRESS_PORT"
          value: "15001"
        - name: "APPMESH_APP_PORTS"
          value: "application_port_list"
        - name: "APPMESH_EGRESS_IGNORED_IP"
          value: "169.254.169.254"
```
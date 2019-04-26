# Getting Started with AWS App Mesh and Kubernetes<a name="mesh-getting-started-k8s"></a>

This topic helps you to use AWS App Mesh with an existing microservice application running on Amazon EKS or Kubernetes on Amazon EC2\.

## Prerequisites<a name="mesh-gs-k8s-prerequisites"></a>

App Mesh supports microservice applications that use service discovery naming for their components\. To use this getting started guide, you must have a microservice application running on Amazon EKS or Kubernetes on AWS\.

Kubernetes `kube-dns` and `coredns` are supported\. For more information, see [DNS for Services and Pods](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/) in the Kubernetes documentation\.

This guide also assumes that you have completed the [Getting Started with AWS App Mesh](getting_started.md) guide and that you have the following App Mesh resources:
+ A service mesh
+ Virtual nodes for each microservice in your application
+ Virtual routers and routes for each microservice in your application \(except for virtual services that are provided by a virtual node directly\)
+ Virtual services for each microservice in your application

## Updating Your Microservice Pod Specifications<a name="mesh-gs-k8s-update-microservices"></a>

App Mesh is a service mesh based on the [Envoy](https://www.envoyproxy.io/) proxy\. After you create your service mesh, virtual nodes, virtual routers, and routes, you must update your microservices to be compatible with App Mesh\.

App Mesh vends the following custom container images that you must add to your Kubernetes pod specifications\.
+ App Mesh Envoy container image: `111345817488.dkr.ecr.us-west-2.amazonaws.com/aws-appmesh-envoy:v1.9.1.0-prod`
+ App Mesh proxy route manager: `111345817488.dkr.ecr.us-west-2.amazonaws.com/aws-appmesh-proxy-route-manager:v2`

The following is an example Kubernetes pod specification that you can merge with your existing application\. Substitute your mesh name and virtual node name for the `APPMESH_VIRTUAL_NODE_NAME` value, and a list of ports that your application listens on for the `APPMESH_APP_PORTS` value\. Substitute the Amazon EC2 instance AWS Region for the `AWS_REGION` value\.

**Example Kubernetes pod spec**  

```
spec:
  containers:
    - name: envoy
      image: 111345817488.dkr.ecr.us-west-2.amazonaws.com/aws-appmesh-envoy:v1.9.1.0-prod
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
      image: 111345817488.dkr.ecr.us-west-2.amazonaws.com/aws-appmesh-proxy-route-manager:v2
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
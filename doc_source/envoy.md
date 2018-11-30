# Envoy and Proxy Route Manager Images<a name="envoy"></a>

AWS App Mesh is a service mesh based on the [Envoy](https://www.envoyproxy.io/) proxy\. 

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/app-mesh/latest/userguide/images/proxy.png)

After you create your service mesh, virtual nodes, virtual routers, and routes, you must update your microservices to be compatible with App Mesh\.

App Mesh vends the following custom container images that you must add to your task groups:
+ App Mesh Envoy container image: `111345817488.dkr.ecr.us-west-2.amazonaws.com/aws-appmesh-envoy:v1.8.0.2-beta`
+ App Mesh proxy route manager: `111345817488.dkr.ecr.us-west-2.amazonaws.com/aws-appmesh-proxy-route-manager:latest`

The following are example Amazon ECS task definition and Kubernetes pod specification snippets that you can merge with your existing task groups, in multiple formats\. Substitute your mesh name and virtual node name for the `APPMESH_VIRTUAL_NODE_NAME` value, and a list of ports that your application listens on for the `APPMESH_APP_PORTS` value\. For the Kubernetes pod spec, substitute the Amazon EC2 instance AWS Region for the `AWS_REGION` value\.

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
# Virtual Nodes<a name="virtual_nodes"></a>

A virtual node acts as a logical pointer to a particular task group, such as an Amazon ECS service or a Kubernetes deployment\. 

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/app-mesh/latest/userguide/images/virtual_node.png)

When you create a virtual node, you must specify the DNS service discovery hostname for your task group\. Any inbound traffic that your virtual node expects should be specified as a *listener*\. Any outbound traffic that your virtual node expects to reach should be specified as a *backend*\.

The response metadata for your new virtual node contains the Amazon Resource Name \(ARN\) that is associated with the virtual node\. Set this value \(either the full ARN or the truncated resource name\) as the `APPMESH_VIRTUAL_NODE_NAME` environment variable for your task group's Envoy proxy container in your task definition or pod spec\. For example, the value could be `mesh/default/virtualNode/simpleapp`\. This is then mapped to the `node.id` and `node.cluster` Envoy parameters\.

**Note**  
If you require your Envoy stats or tracing to use a different name, you can override the `node.cluster` value that is set by `APPMESH_VIRTUAL_NODE_NAME` with the `APPMESH_VIRTUAL_NODE_CLUSTER` environment variable\.

## Creating a Virtual Node<a name="create-virtual-node"></a>

This topic helps you to create a virtual node in your service mesh\.

**To create a virtual node in the AWS Management Console\.**

1. Open the App Mesh console at [https://console\.aws\.amazon\.com/appmesh/](https://console.aws.amazon.com/appmesh/)\.

1. Choose **Virtual nodes** in the left navigation\.

1. Choose **Create virtual node**\.

1. For **Virtual node name**, choose a name for your virtual node\.

1. For **Service discovery method**, choose **DNS** for services that use DNS service discovery and then specify the hostname for **DNS hostname**\. Otherwise, choose **None** if your virtual node doesn't expect any ingress traffic\.

1. To specify any backends \(for egress traffic\) for your virtual node, or to configure inbound and outbound access logging information, choose **Additional configuration**\.

   1. To specify a backend, choose **Add backend** and enter a virtual service name or full Amazon Resource Name \(ARN\) for the virtual service that your virtual node communicates with\. Repeat this step until all of your virtual node backends are accounted for\.

   1. To configure logging, enter the HTTP access logs path that you want Envoy to use\. We recommend the `/dev/stdout` path so that you can use Docker log drivers to export your Envoy logs to a service such as Amazon CloudWatch Logs\.
**Note**  
Logs must still be ingested by an agent in your application and sent to a destination\. This file path only instructs Envoy where to send the logs\.

1. If your virtual node expects ingress traffic, specify a **Port** and **Protocol** for that **Listener**\.

1. If you want to configure health checks for your listener, ensure that **Health check enabled** is selected and then complete the following substeps\. If not, clear this check box\.

   1. For **Health check protocol**, choose to use an HTTP or TCP health check\.

   1. For **Health check port**, specify the port that the health check should run on\.

   1. For **Healthy threshold**, specify the number of consecutive successful health checks that must occur before declaring the listener healthy\.

   1. For **Health check interval**, specify the time period in milliseconds between each health check execution\.

   1. For **Path**, specify the destination path for the health check request\. This is required only if the specified protocol is HTTP\. If the protocol is TCP, this parameter is ignored\.

   1. For **Timeout period**, specify the amount of time to wait when receiving a response from the health check, in milliseconds\.

   1. For **Unhealthy threshold**, specify the number of consecutive failed health checks that must occur before declaring the listener unhealthy\.

1. Chose **Create virtual node** to finish\.

**Creating a virtual node in the AWS CLI\.**
+ The following JSON represents a virtual node named `serviceA`\. This virtual node listens on port 8080, and it expects to send traffic to a virtual service within the mesh named `serviceB.simpleapp.local` as a backend\. The task group for the virtual node \(the Amazon ECS service with service discovery support, or Kubernetes deployment\) uses the `serviceA.simpleapp.local` hostname for service discovery\.

  ```
  {
    "meshName": "simpleapp",
    "spec": {
      "backends": [
              {
                  "virtualService": {
                      "virtualServiceName": "serviceB.simpleapp.local"
                  }
              }
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
          "hostname": "serviceA.simpleapp.local"
        }
      }
    },
    "virtualNodeName": "serviceA"
  }
  ```

  If you save the preceding JSON as a file, you can create the virtual node with the following AWS CLI command\.

  ```
  aws appmesh create-virtual-node --cli-input-json file://serviceA.json
  ```
**Note**  
You must use at least version 1\.16\.133 of the AWS CLI with App Mesh\. To install the latest version of the AWS CLI, see [Installing the AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/installing.html) in the *AWS Command Line Interface User Guide*\.

  For more information about creating virtual nodes with the AWS CLI, see [create\-virtual\-node](https://docs.aws.amazon.com/cli/latest/reference/appmesh/create-virtual-node.html) in the *AWS Command Line Interface User Guide*\.
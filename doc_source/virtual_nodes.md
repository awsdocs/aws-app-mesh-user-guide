# Virtual Nodes<a name="virtual_nodes"></a>

A virtual node acts as a logical pointer to a particular task group, such as an Amazon ECS service or a Kubernetes deployment\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/app-mesh/latest/userguide/images/virtual_node.png)

When you create a virtual node, you must specify a service discovery method for your task group\. Any inbound traffic that your virtual node expects should be specified as a *listener*\. Any outbound traffic that your virtual node expects to reach should be specified as a *backend*\.

The response metadata for your new virtual node contains the Amazon Resource Name \(ARN\) that is associated with the virtual node\. Set this value \(either the full ARN or the truncated resource name\) as the `APPMESH_VIRTUAL_NODE_NAME` environment variable for your task group's Envoy proxy container in your task definition or pod spec\. For example, the value could be `mesh/default/virtualNode/simpleapp`\. This is then mapped to the `node.id` and `node.cluster` Envoy parameters\.

**Note**  
If you require your Envoy stats or tracing to use a different name, you can override the `node.cluster` value that is set by `APPMESH_VIRTUAL_NODE_NAME` with the `APPMESH_VIRTUAL_NODE_CLUSTER` environment variable\.

## Creating a Virtual Node<a name="create-virtual-node"></a>

To create a virtual node using the AWS Management Console complete the following steps\. To create a virtual node using the AWS CLI, use the `aws appmesh create-virtual-node` command\. For CLI examples of creating virtual nodes with different settings, see [create\-virtual\-node](https://docs.aws.amazon.com/cli/latest/reference/appmesh/create-virtual-node.html)\.

1. Open the App Mesh console at [https://console\.aws\.amazon\.com/appmesh/](https://console.aws.amazon.com/appmesh/)\.

1. Choose **Virtual nodes** in the left navigation\.

1. Choose **Create virtual node**\. 

1. For **Virtual node name**, enter a name for your virtual node\. 

1. For **Service discovery method**, choose one of the following options:
   + **DNS** – Specify the DNS hostname\. 
   + **AWS Cloud Map** – Specify the service name and namespace\. Optionally, you can also specify attributes that App Mesh can query AWS Cloud Map for\. Only instances that match all of the specified key/value pairs will be returned\. To use AWS Cloud Map, your account must have the `AWSServiceRoleForAppMesh` [service\-linked role](using-service-linked-roles.md)\. 
   + **None** – Select if your virtual node doesn't expect any inbound traffic\. 

1. To specify any backends \(for egress traffic\) for your virtual node, or to configure inbound and outbound access logging information, choose **Additional configuration** 

   1. To specify a backend, choose **Add backend** and enter a virtual service name or full Amazon Resource Name \(ARN\) for the virtual service that your virtual node communicates with\. Repeat this step until all of your virtual node backends are accounted for\. 

   1. To configure logging, enter the HTTP access logs path that you want Envoy to use\. We recommend the `/dev/stdout` path so that you can use Docker log drivers to export your Envoy logs to a service such as Amazon CloudWatch Logs\. 
**Note**  
Logs must still be ingested by an agent in your application and sent to a destination\. This file path only instructs Envoy where to send the logs\. 

1. If your virtual node expects ingress traffic, specify a **Port** and **Protocol** for the **Listener**\. 

1. If you want to configure health checks for your listener, ensure that **Health check enabled** is selected and then complete the following substeps\. If not, clear this check box\. 

   1. For **Health check protocol**, choose to use an HTTP or TCP health check\. 

   1. For **Health check port**, specify the port that the health check should run on\. 

   1. For **Healthy threshold**, specify the number of consecutive successful health checks that must occur before declaring the listener healthy\. 

   1. For **Health check interval**, specify the time period in milliseconds between each health check execution\. 

   1. For **Path**, specify the destination path for the health check request\. This is required only if the specified protocol is HTTP\. If the protocol is TCP, this parameter is ignored\. 

   1. For **Timeout period**, specify the amount of time to wait when receiving a response from the health check, in milliseconds\. 

   1. For **Unhealthy threshold**, specify the number of consecutive failed health checks that must occur before declaring the listener unhealthy\. 

1. Choose **Create virtual node** to finish\. 

## Deleting a Virtual Node<a name="delete-virtual-node"></a>

To delete a virtual node using the AWS Management Console complete the following steps\. To delete a virtual node using the AWS CLI, use the `aws appmesh delete-virtual-node` command\. For an example of deleting a virtual node using the AWS CLI, see [delete\-virtual\-node](https://docs.aws.amazon.com/cli/latest/reference/appmesh/delete-virtual-node.html)\.

1. Open the App Mesh console at [https://console\.aws\.amazon\.com/appmesh/](https://console.aws.amazon.com/appmesh/)\.

1. Choose the mesh that you want to delete a virtual node from\.

1. Choose **Virtual nodes** in the left navigation\.

1. In the **Virtual Nodes** table, choose the virtual node that you want to delete and select **Delete**\.

1. In the confirmation box, type **delete** and then select **Delete**\.
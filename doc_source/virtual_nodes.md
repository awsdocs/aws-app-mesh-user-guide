# Virtual Nodes<a name="virtual_nodes"></a>

A virtual node acts as a logical pointer to a particular task group, such as an Amazon ECS service or a Kubernetes deployment\. 

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/app-mesh/latest/userguide/images/virtual_node.png)

When you create a virtual node, you must specify the DNS service discovery hostname for your task group\. Any inbound traffic that your virtual node expects should be specified as a `listener`\. Any outbound traffic that your virtual node expects to reach should be specified as a `backend`\.

The response metadata for your new virtual node contains the `arn` that is associated with the virtual node\. Set this value \(either the full ARN or the truncated resource name\) as the `APPMESH_VIRTUAL_NODE_NAME` environment variable for your task group's Envoy proxy container in your task definition or pod spec\. For example, the value could be `mesh/default/virtualNode/simpleapp`\. This is then mapped to the `node.id` and `node.cluster` Envoy parameters\.

**Note**  
If you require your Envoy stats or tracing to use a different name, you can override the `node.cluster` value that is set by `APPMESH_VIRTUAL_NODE_NAME` with the `APPMESH_VIRTUAL_NODE_CLUSTER` environment variable\.

The following JSON represents a virtual node called `serviceA`\. This virtual node listens on port 8080, and it expects to send traffic to a virtual service within the mesh called `serviceB.simpleapp.local` as a backend\. The task group for the virtual node \(the Amazon ECS service with service discovery support, or Kubernetes deployment\) uses the `serviceA.simpleapp.local` hostname for service discovery\.

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

If the above JSON is saved as a file, you can create the virtual node with the following AWS CLI command\.

```
aws appmesh create-virtual-node --cli-input-json file://serviceA.json
```

**Note**  
You must use at least version 1\.16\.120 of the AWS CLI with App Mesh\. To install the latest version of the AWS CLI, see [Installing the AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/installing.html) in the *AWS Command Line Interface User Guide*\.

For more information about creating virtual nodes with the AWS CLI, see [create\-virtual\-node](https://docs.aws.amazon.com/cli/latest/reference/appmesh/create-virtual-node.html) in the *AWS Command Line Interface User Guide*\.
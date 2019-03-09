# Service Meshes<a name="meshes"></a>

A service mesh is a logical boundary for network traffic between the services that reside within it\. Service mesh components include virtual services, virtual nodes, virtual routers, and routes that distribute traffic between the applications in your mesh\.

The following example AWS CLI command creates a service mesh called `simpleapp`:

```
aws appmesh create-mesh --mesh-name simpleapp
```

**Note**  
You must use at least version 1\.16\.120 of the AWS CLI with App Mesh\. To install the latest version of the AWS CLI, see [Installing the AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/installing.html) in the *AWS Command Line Interface User Guide*\.

For more information about creating service meshes with the AWS CLI, see [create\-mesh](https://docs.aws.amazon.com/cli/latest/reference/appmesh/create-mesh.html) in the *AWS Command Line Interface User Guide*\.
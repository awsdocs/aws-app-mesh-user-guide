# Service Meshes<a name="meshes"></a>

A service mesh is a logical boundary for network traffic between the services that reside within it\. 

After you create your service mesh, you can create virtual services, virtual nodes, virtual routers, and routes to distribute traffic between the applications in your mesh\.

## Creating a Service Mesh<a name="create-mesh"></a>

This topic helps you to create a new service mesh\.

**To create a new service mesh with the AWS Management Console**

1. Open the App Mesh console at [https://console\.aws\.amazon\.com/appmesh/](https://console.aws.amazon.com/appmesh/)\.

1. Choose **Create mesh**\.

1. For **Mesh name**, specify a name for your service mesh\.

1. Choose **Create mesh** to finish\.

**To create a new service mesh with the AWS CLI**
+ The following example AWS CLI command creates a service mesh named `simpleapp`\.

  ```
  aws appmesh create-mesh --mesh-name simpleapp
  ```
**Note**  
You must use at least version 1\.16\.178 of the AWS CLI with App Mesh\. To install the latest version of the AWS CLI, see [Installing the AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/installing.html) in the *AWS Command Line Interface User Guide*\.

  For more information about creating service meshes with the AWS CLI, see [create\-mesh](https://docs.aws.amazon.com/cli/latest/reference/appmesh/create-mesh.html) in the *AWS Command Line Interface User Guide*\.
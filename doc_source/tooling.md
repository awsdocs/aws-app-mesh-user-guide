# App Mesh tooling<a name="tooling"></a>

App Mesh gives customers the ability to interact with its APIs indirectly using tools such as:
+ AWS CloudFormation
+ AWS Cloud Development Kit \(AWS CDK\)
+ App Mesh Controller for Kubernetes
+ Terraform

## App Mesh and AWS CloudFormation<a name="aws-cf-app-mesh"></a>

AWS CloudFormation is a service that lets you create a template with all the resources you need for your application, and then AWS CloudFormation will configure and provision the resouces for you\. It will also configure all the dependencies, so you can focus more on you application and less on managing resources\.

For more information and examples on using AWS CloudFormation with App Mesh, see the [AWS CloudFormation documentation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-appmesh-mesh.html)\.

## App Mesh and AWS CDK<a name="aws-cdk-app-mesh"></a>

AWS CDK is a development framework for using code to define your cloud infrastructure and using AWS CloudFormation to provision it\. AWS CDK supports multiple programming languages including TypeScript, JavaScript, Python, Java, and C\#/\.Net\.

For more information on using AWS CDK with App Mesh, see the [AWS CDK documentation](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-appmesh-readme.html)\.

## App Mesh controller for Kubernetes<a name="app-mesh-kubernetes-controller"></a>

The App Mesh controller for Kubernetes helps you to manage your App Mesh resources for a Kubernetes cluster and inject sidecars into pods\. This controller is specifically for use with Amazon EKS and allows you to manage your resources in a manner that is native to Kubernetes\.

For more information on the App Mesh controller, see the [App Mesh Controller documentation](https://aws.github.io/aws-app-mesh-controller-for-k8s/)\.

To see a guide on implementing App Mesh with Amazon EKS using the App Mesh Controller for Kubernetes, check out the [Amazon EKS Workshop](https://www.eksworkshop.com/advanced/320_servicemesh_with_appmesh/install_app_mesh_controller/)\.

## App Mesh and Terraform<a name="app-mesh-terraform"></a>

[Terraform](https://www.terraform.io/) is an open\-source infrastructure as code software tool\. Terraform can manage cloud services using thier CLI and interacts with APIs using declaritive configuration files\.

To see more about using App Mesh with Terraform, check out the [Terraform documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/appmesh_mesh)\.
# Working with shared meshes<a name="sharing"></a>

A shared mesh allows resources created by different accounts to communicate with each other in the same mesh\.

An AWS Identity and Access Management account can be a mesh resource owner, a mesh consumer, or both\. Consumers can create resources in a mesh that is shared with their account\. Owners can create resources in any mesh the account owns\. A mesh owner can share a mesh with the following types of mesh consumers:
+ Specific AWS accounts inside or outside of its organization in AWS Organizations
+ An organizational unit inside its organization in AWS Organizations
+ Its entire organization in AWS Organizations

For an end\-to\-end walk through of sharing a mesh, see [Cross\-account mesh walk through](https://github.com/aws/aws-app-mesh-examples/tree/master/walkthroughs/howto-cross-account) on GitHub\.

## Shared mesh permissions<a name="sharing-permissions"></a>

A shared mesh has the following permissions:
+ Consumers can list and describe all resources in a mesh that is shared with the account\.
+ Owners can list and describe all resources in any mesh the account owns\.
+ Owners and consumers can modify resources in a mesh that the account created, but they cannot modify resources that other another account created\.
+ Consumers can delete any resource in a mesh that the account created\.
+ Owners can delete any resource in a mesh that any account created\.
+ Owner's resources can only reference other resources in the same account\. For example, a virtual node can only reference AWS Cloud Map or an AWS Certificate Manager certificate that is in the same account as the virtual node's owner\.
+ Owners and consumers can connect an Envoy proxy to App Mesh as a virtual node that the account owns\.

## Prerequisites for sharing meshes<a name="sharing-prereqs"></a>

The following prerequisites must be met in order to share a mesh:
+ You must own the mesh in your AWS account\. You cannot share a mesh that has been shared with you\.
+ To share a mesh with your organization or an organizational unit in AWS Organizations, you must enable sharing with AWS Organizations\. For more information, see [ Enable Sharing with AWS Organizations](https://docs.aws.amazon.com/ram/latest/userguide/getting-started-sharing.html#getting-started-sharing-orgs) in the *AWS RAM User Guide*\.
+ Your services must be deployed in an Amazon VPC that has shared connectivity across the accounts that include the mesh resources that you want to communicate with each other\. One way to share network connectivity is to deploy all of the services that you want to use in your mesh to a shared subnet\. For more information and limitations, see [Sharing a Subnet](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-sharing.html#vpc-sharing-share-subnet)\.
+ Services must be discoverable through DNS or AWS Cloud Map\. For more information about service discovery, see [Virtual nodes](virtual_nodes.md)\.

## Related services<a name="sharing-related"></a>

mesh sharing integrates with AWS Resource Access Manager \(AWS RAM\)\. AWS RAM is a service that enables you to share your AWS resources with any AWS account or through AWS Organizations\. With AWS RAM, you share resources that you own by creating a *resource share*\. A resource share specifies the resources to share, and the consumers with whom to share them\. Consumers can be individual AWS accounts, or organizational units or an entire organization in AWS Organizations\.

For more information about AWS RAM, see the *[AWS RAM User Guide](https://docs.aws.amazon.com/ram/latest/userguide/)*\.

## Sharing a mesh<a name="sharing-share"></a>

Sharing a mesh enables mesh resources created by different accounts to communicate with each other in the same mesh\. You can only share a mesh that you own\. To share a mesh, you must add it to a resource share\. A resource share is an AWS RAM resource that lets you share your resources across AWS accounts\. A resource share specifies the resources to share, and the consumers with whom they are shared\. When you share a mesh using the App Mesh console, you add it to an existing resource share\. To add the mesh to a new resource share, you must first create the resource share using the [AWS RAM console](https://console.aws.amazon.com/ram)\.

If you are part of an organization in AWS Organizations and sharing within your organization is enabled, consumers in your organization can be automatically granted access to the shared mesh\. Otherwise, consumers receive an invitation to join the resource share and are granted access to the shared mesh after accepting the invitation\.

You can share a mesh that you own using the AWS RAM console or the AWS CLI\.

**To share a mesh that you own using the AWS RAM console**  
See [Creating a Resource Share](https://docs.aws.amazon.com/ram/latest/userguide/working-with-sharing.html#working-with-sharing-create) in the *AWS RAM User Guide*\. When selecting a resource type, select **Meshes**, and then select the mesh you want to share\. If no meshes are listed, then you need to create a mesh first\. For more information, see [Creating a service mesh](meshes.md#create-mesh)\.

**To share a mesh that you own using the AWS CLI**  
Use the [create\-resource\-share](https://docs.aws.amazon.com/cli/latest/reference/ram/create-resource-share.html) command\. For the `--resource-arns` option, specify the ARN of the mesh that you want to share\.

## Unsharing a shared mesh<a name="sharing-unshare"></a>

When you unshare a mesh, App Mesh disables further access to the mesh by former consumers of the mesh but does not delete the resources created by the consumers\. Once the mesh is unshared, only the mesh owner can access and delete the resources\. App Mesh prevents the account that owned resources in the mesh, and any other accounts with resources in the mesh, from receiving any configuration information after the mesh is unshared\. Only the owner of the mesh can unshare it\.

To unshare a shared mesh that you own, you must remove it from the resource share\. You can do this using the AWS RAM console or the AWS CLI\.

**To unshare a shared mesh that you own using the AWS RAM console**  
See [Updating a Resource Share](https://docs.aws.amazon.com/ram/latest/userguide/working-with-sharing.html#working-with-sharing-update) in the *AWS RAM User Guide*\.

**To unshare a shared mesh that you own using the AWS CLI**  
Use the [disassociate\-resource\-share](https://docs.aws.amazon.com/cli/latest/reference/ram/disassociate-resource-share.html) command\.

## Identifying a shared mesh<a name="sharing-identify"></a>

Owners and consumers can identify shared meshes and mesh resources using the App Mesh console and AWS CLI

**To identify a shared mesh using the App Mesh console**

1. Open the App Mesh console at [https://console\.aws\.amazon\.com/appmesh/](https://console.aws.amazon.com/appmesh/)\.

1. From the left navigation, select **Meshes**\. The account ID of the mesh owner for each mesh is listed in the **Mesh owner** column\.

1. From the left navigation, select **Virtual services**, **Virtual routers**, or **Virtual nodes**\. You see the account ID for the **Mesh owner** and **Resource owner** for each of the resources\.

**To identify a shared mesh using the AWS CLI**  
Use the `aws appmesh list resource` command, such as `aws appmesh [list\-meshes](https://docs.aws.amazon.com/cli/latest/reference/appmesh/list-meshes.html)`\. The command returns the meshes that you own and the meshes that are shared with you\. The `meshOwner` property shows the AWS account ID of the `meshOwner` and the `resourceOwner` property shows the AWS account ID of the resource owner\. Any command run against any mesh resource returns these properties\.

## Billing and metering<a name="sharing-billing"></a>

There are no charges for sharing a mesh\.

## Instance quotas<a name="sharing-limits"></a>

All quotas for a mesh also apply to shared meshes, regardless of who created resources in the mesh\. Only a mesh owner can request quota increases\. For more information, see [App Mesh service quotas](service-quotas.md)\. The AWS Resource Access Manager service also has quotas\. For more information, see [Service Quotas](https://docs.aws.amazon.com/ram/latest/userguide/what-is.html#what-is-limits)\.
# Working with shared meshes<a name="sharing"></a>

A shared mesh allows resources created by different accounts to communicate with each other in the same mesh\.

An AWS Identity and Access Management account can be a mesh resource owner, a mesh consumer, or both\. Consumers can create resources in a mesh that is shared with their account\. Owners can create resources in any mesh the account owns\. A mesh owner can share a mesh with the following types of mesh consumers:
+ Specific AWS accounts inside or outside of its organization in AWS Organizations
+ An organizational unit inside its organization in AWS Organizations
+ Its entire organization in AWS Organizations

For an end\-to\-end walk through of sharing a mesh, see [Cross\-account mesh walk through](https://github.com/aws/aws-app-mesh-examples/tree/main/walkthroughs/howto-cross-account) on GitHub\.

## Shared mesh permissions<a name="sharing-permissions"></a>

A shared mesh has the following permissions:
+ Consumers can list and describe all resources in a mesh that is shared with the account\.
+ Owners can list and describe all resources in any mesh the account owns\.
+ Owners and consumers can modify resources in a mesh that the account created, but they cannot modify resources that other another account created\.
+ Consumers can delete any resource in a mesh that the account created\.
+ Owners can delete any resource in a mesh that any account created\.
+ Owner's resources can only reference other resources in the same account\. For example, a virtual node can only reference AWS Cloud Map or an AWS Certificate Manager certificate that is in the same account as the virtual node's owner\.
+ Owners and consumers can connect an Envoy proxy to App Mesh as a virtual node that the account owns\.
+ Owners can create virtual gateways and virtual gateway routes\.
+ Owners and consumers can list tags and can tag/untag resources in a mesh that the account created\. They can't list tags and tag/untag resources in a mesh that aren't created by the account\.

Shared meshes migrated from support for account\-level allow\-all authorization to policy\-based authorization\. Any meshes shared before this roll\-out use the former strategy, and any meshes shared after this roll\-out use the latter strategy\. 

With policy\-based authorization, a mesh is shared with with a fixed set of permissions\. These permissions are selected to be added to a resource policy, and an optional IAM policy can also be selected based on /role\. The intersection of permissions allowed in these policies, less any explicit permissions denied, determines a principal's access to the mesh\.

The set of permissions added the resource policy is fixed, and determined by AWS Resource Access Manager \(AWS RAM\):
+ `appmesh:CreateVirtualNode`
+ `appmesh:CreateVirtualRouter`
+ `appmesh:CreateRoute`
+ `appmesh:CreateVirtualService`
+ `appmesh:UpdateVirtualNode`
+ `appmesh:UpdateVirtualRouter`
+ `appmesh:UpdateRoute`
+ `appmesh:UpdateVirtualService`
+ `appmesh:ListVirtualNodes`
+ `appmesh:ListVirtualRouters`
+ `appmesh:ListRoutes`
+ `appmesh:ListVirtualServices`
+ `appmesh:DescribeMesh`
+ `appmesh:DescribeVirtualNode`
+ `appmesh:DescribeVirtualRouter`
+ `appmesh:DescribeRoute`
+ `appmesh:DescribeVirtualService`
+ `appmesh:DeleteVirtualNode`
+ `appmesh:DeleteVirtualRouter`
+ `appmesh:DeleteRoute`
+ `appmesh:DeleteVirtualService`

**Note**  
This set excludes some permissions, such as `appmesh:TagResources`\. If access to `appmesh:TagResources` is required, we can map your account\(s\) to the account\-level allow\-all authorization strategy until we launch support for `appmesh:TagResources`\.

## Prerequisites for sharing meshes<a name="sharing-prereqs"></a>

To share a mesh, you must meet the following prerequisites\.
+ You must own the mesh in your AWS account\. You cannot share a mesh that has been shared with you\.
+ To share a mesh with your organization or an organizational unit in AWS Organizations, you must enable sharing with AWS Organizations\. For more information, see [ Enable Sharing with AWS Organizations](https://docs.aws.amazon.com/ram/latest/userguide/getting-started-sharing.html#getting-started-sharing-orgs) in the *AWS RAM User Guide*\.
+ Your services must be deployed in an Amazon VPC that has shared connectivity across the accounts that include the mesh resources that you want to communicate with each other\. One way to share network connectivity is to deploy all of the services that you want to use in your mesh to a shared subnet\. For more information and limitations, see [Sharing a Subnet](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-sharing.html#vpc-sharing-share-subnet)\.
+ Services must be discoverable through DNS or AWS Cloud Map\. For more information about service discovery, see [Virtual nodes](virtual_nodes.md)\.

## Related services<a name="sharing-related"></a>

Mesh sharing integrates with AWS Resource Access Manager \(AWS RAM\)\. AWS RAM is a service that enables you to share your AWS resources with any AWS account or through AWS Organizations\. With AWS RAM, you share resources that you own by creating a *resource share*\. A resource share specifies the resources to share, and the consumers with whom to share them\. Consumers can be individual AWS accounts, or organizational units or an entire organization in AWS Organizations\.

For more information about AWS RAM, see the *[AWS RAM User Guide](https://docs.aws.amazon.com/ram/latest/userguide/)*\.

## Sharing a mesh<a name="sharing-share"></a>

Sharing a mesh enables mesh resources created by different accounts to communicate with each other in the same mesh\. You can only share a mesh that you own\. To share a mesh, you must add it to a resource share\. A resource share is an AWS RAM resource that lets you share your resources across AWS accounts\. A resource share specifies the resources to share and the consumers with whom they are shared\. When you share a mesh using the Amazon Linux console, you add it to an existing resource share\. To add the mesh to a new resource share, create the resource share using the [AWS RAM console](https://console.aws.amazon.com/ram)\.

If you're part of an organization in AWS Organizations and sharing within your organization is enabled, consumers in your organization can be automatically granted access to the shared mesh\. Otherwise, consumers receive an invitation to join the resource share and are granted access to the shared mesh after accepting the invitation\.

You can share a mesh that you own using the AWS RAM console or the AWS CLI\.

**To share a mesh that you own using the AWS RAM console**  
For instructions, see [Creating a Resource Share](https://docs.aws.amazon.com/ram/latest/userguide/working-with-sharing.html#working-with-sharing-create) in the *AWS RAM User Guide*\. When you select a resource type, select **Meshes**, and then select the mesh that you want to share\. If no meshes are listed, create a mesh first\. For more information, see [Creating a service mesh](meshes.md#create-mesh)\.

**To share a mesh that you own using the AWS CLI**  
Use the [create\-resource\-share](https://docs.aws.amazon.com/cli/latest/reference/ram/create-resource-share.html) command\. For the `--resource-arns` option, specify the ARN of the mesh that you want to share\.

## Unsharing a shared mesh<a name="sharing-unshare"></a>

When you unshare a mesh, App Mesh disables further access to the mesh by former consumers of the mesh\. However, App Mesh doesn't delete the resources created by the consumers\. After the mesh is unshared, only the mesh owner can access and delete the resources\. App Mesh prevents the account that owned resources in the mesh from receiving configuration information after the mesh is unshared\. App Mesh also prevents any other accounts with resources in the mesh from receiving configuration information from an unshared mesh\. Only the owner of the mesh can unshare it\.

To unshare a shared mesh that you own, you must remove it from the resource share\. You can do this using the AWS RAM console or the AWS CLI\.

**To unshare a shared mesh that you own using the AWS RAM console**  
For instructions, see [Updating a Resource Share](https://docs.aws.amazon.com/ram/latest/userguide/working-with-sharing.html#working-with-sharing-update) in the *AWS RAM User Guide*\.

**To unshare a shared mesh that you own using the AWS CLI**  
Use the [disassociate\-resource\-share](https://docs.aws.amazon.com/cli/latest/reference/ram/disassociate-resource-share.html) command\.

## Identifying a shared mesh<a name="sharing-identify"></a>

Owners and consumers can identify shared meshes and mesh resources using the Amazon Linux console and AWS CLI

**To identify a shared mesh using the Amazon Linux console**

1. Open the App Mesh console at [https://console\.aws\.amazon\.com/appmesh/](https://console.aws.amazon.com/appmesh/)\. 

1. From the left navigation, select **Meshes**\. The account ID of the mesh owner for each mesh is listed in the **Mesh owner** column\.

1. From the left navigation, select **Virtual services**, **Virtual routers**, or **Virtual nodes**\. You see the account ID for the **Mesh owner** and **Resource owner** for each of the resources\.

**To identify a shared mesh using the AWS CLI**  
Use the `aws appmesh list resource` command, such as `aws appmesh [list\-meshes](https://docs.aws.amazon.com/cli/latest/reference/appmesh/list-meshes.html)`\. The command returns the meshes that you own and the meshes that are shared with you\. The `meshOwner` property shows the AWS account ID of the `meshOwner` and the `resourceOwner` property shows the AWS account ID of the resource owner\. Any command run against any mesh resource returns these properties\.

The user defined tags that you attach to a shared mesh are available only to your AWS account\. They're not available to the other accounts that the mesh is shared with\. The `aws appmesh list-tags-for-resource` command for a mesh in another account is denied access\.

## Billing and metering<a name="sharing-billing"></a>

There are no charges for sharing a mesh\.

## Instance quotas<a name="sharing-limits"></a>

All quotas for a mesh also apply to shared meshes, regardless of who created resources in the mesh\. Only a mesh owner can request quota increases\. For more information, see [App Mesh service quotas](service-quotas.md)\. The AWS Resource Access Manager service also has quotas\. For more information, see [Service Quotas](https://docs.aws.amazon.com/ram/latest/userguide/what-is.html#what-is-limits)\.
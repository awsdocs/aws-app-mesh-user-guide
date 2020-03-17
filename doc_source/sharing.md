# Working with Shared Meshes<a name="sharing"></a>

A shared mesh allows resources created by different accounts to communicate with each other in the same mesh\.

An AWS Identity and Access Management account can be a mesh resource owner, a mesh consumer, or both\. Consumers can create resources in a mesh that is shared with their account\. Owners can create resources in any mesh the account owns\. A mesh owner can share a mesh with the following types of mesh consumers:
+ Specific AWS accounts inside or outside its organization in AWS Organizations
+ An organizational unit inside its organization in AWS Organizations
+ Its entire organization in AWS Organizations

For an end\-to\-end walk through of sharing a mesh, see [Cross\-account mesh walk through](https://github.com/aws/aws-app-mesh-examples/tree/master/walkthroughs/howto-cross-account) on GitHub\.

## Shared Mesh Permissions<a name="sharing-permissions"></a>

A shared mesh has the following permissions:
+ Consumers can list and describe all resources in a mesh that is shared with the account\.
+ Owners can list and describe all resources in any mesh the account owns\.
+ Owners and consumers can modify resources in a mesh that the account created, but they cannot modify resources that other another account created\.
+ Consumers can delete any resource in a mesh that the account created\.
+ Owners can delete any resource in a mesh that any account created\.
+ When an owner or consumer creates or updates a resource that references other AWS resources, such as AWS Cloud Map or AWS Certificate Manager, the only resources that can be referenced are resources in the same account\. For an example of a virtual node that references a TLS certificate, see the [TLS encryption](virtual-node-tls.md) topic in this guide\.
+ Owners and consumers can connect an Envoy proxy to App Mesh as a virtual node that the account owns\.

## Prerequisites for Sharing Meshes<a name="sharing-prereqs"></a>

The following prerequisites must be met in order to share a mesh:
+ You must own the mesh in your AWS account\. You cannot share a mesh that has been shared with you\.
+ To share a mesh with your organization or an organizational unit in AWS Organizations, you must enable sharing with AWS Organizations\. For more information, see [ Enable Sharing with AWS Organizations](https://docs.aws.amazon.com/ram/latest/userguide/getting-started-sharing.html#getting-started-sharing-orgs) in the *AWS RAM User Guide*\.
+ Your services must be deployed in an Amazon VPC that has shared connectivity across the accounts that include the mesh resources that you want to communicate with each other\. One way to share network connectivity is to deploy all of the services that you want to use in your mesh to a shared subnet\. For more information and limitations, see [Sharing a Subnet](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-sharing.html#vpc-sharing-share-subnet)\.
+ Services must be discoverable through DNS or AWS Cloud Map\. For more information about service discovery, see [Virtual nodes](virtual_nodes.md)\.

## Related Services<a name="sharing-related"></a>

Mesh sharing integrates with AWS Resource Access Manager \(AWS RAM\)\. AWS RAM is a service that enables you to share your AWS resources with any AWS account or through AWS Organizations\. With AWS RAM, you share resources that you own by creating a *resource share*\. A resource share specifies the resources to share and the consumers with whom to share them\. Consumers can be individual AWS accounts, or organizational units or an entire organization in AWS Organizations\.

For more information about AWS RAM, see the *[AWS RAM User Guide](https://docs.aws.amazon.com/ram/latest/userguide/)*\.

## Share a Mesh<a name="sharing-share"></a>

Sharing a mesh enables mesh resources created by different accounts to communicate with each other in the same mesh\. You can only share a mesh that you own\. In all of the examples in this topic, an IAM account with the ID *111122223333* is the owner of a mesh that is shared with account ID *444455556666*\. 

**To share a mesh**

Your account must be assigned the following IAM permissions:
+ `appmesh:GetMeshPolicy`
+ `appmesh:PutMeshPolicy`

1. Using the *111122223333* account, create a mesh with the following command\.

   ```
   aws appmesh create-mesh --region region-code --mesh-name mesh-a
   ```

   Note the value for `arn` that is returned in the output for use in the next step\.

1. Using the *111122223333* account, share the mesh created in the previous step with the *444455556666* account with the following command\.

   ```
   aws ram create-resource-share \
       --region region-code \
       --name mesh-a \
       --principals 444455556666 \
       --resource arn:aws:appmesh:region-code:111122223333:mesh/mesh-a
   ```

   The output returned is similar to the following example output\.

   ```
   {
       "resourceShare": {
           "resourceShareArn": "arn:aws:ram:region-code:111122223333:resource-share/1cb1da11-011b-1c1c-11e1-11111Example",
           "name": "mesh-a",
           "owningAccountId": "111122223333",
           "allowExternalPrincipals": true,
           "status": "ACTIVE",
           "creationTime": 1579284303.988,
           "lastUpdatedTime": 1579284303.988
       }
   }
   ```

1. Accept the sharing request\. The account that you shared the mesh with must accept the sharing request to use the mesh\.

   1. Using the *444455556666* account, view the account's sharing invitations\.

      ```
      aws ram --region region-code get-resource-share-invitations
      ```

      The output returned is similar to the following example output\.

      ```
      {
          "resourceShareInvitations": [
              {
                  "resourceShareInvitationArn": "arn:aws:ram:region-code:111122223333:resource-share-invitation/11111111-1111-1111-1111-11111111111",
                  "resourceShareName": "mesh-a",
                  "resourceShareArn": "arn:aws:ram:region:111122223333:resource-share/1cb1da11-011b-1c1c-11e1-11111Example",
                  "senderAccountId": "111122223333",
                  "receiverAccountId": "444455556666",
                  "invitationTimestamp": 1579145083.23,
                  "status": "PENDING"
              }
          ]
      }
      ```

   1. Using the *444455556666* account, accept the sharing invitation\.

      ```
      aws ram --region region-code accept-resource-share-invitation \
          --resource-share-invitation-arn arn:aws:ram:region-code:111122223333:resource-share-invitation/11111111-1111-1111-1111-11111111111
      ```

      The output returned is similar to the following example output\.

      ```
      {
          "resourceShareInvitation": {
              "resourceShareInvitationArn": "arn:aws:ram:region-code:111122223333:resource-share-invitation/11111111-1111-1111-1111-11111111111",
              "resourceShareName": "mesh-a",
              "resourceShareArn": "arn:aws:ram:region-code:111122223333:resource-share/1cb1da11-011b-1c1c-11e1-11111Example",
              "senderAccountId": "111122223333,
              "receiverAccountId": "444455556666",
              "invitationTimestamp": 1579287509.04,
              "status": "ACCEPTED"
          }
      }
      ```

1. Using the *111122223333* account, view the sharing policy for the shared mesh with the following command\.

   ```
   aws ram get-resource-policies --resource-arns arn:aws:appmesh:region-code:111122223333:mesh/mesh-a
   ```

## Shared Mesh Tasks<a name="shared-mesh-tasks"></a>

You can use all of the App Mesh AWS CLI commands with resources in a shared mesh, just as you would resources in an unshared mesh\. For more information about App Mesh AWS CLI commands, see the [AWS App Mesh Command Line Reference](https://docs.aws.amazon.com/cli/latest/reference/appmesh/)\. You cannot use the AWS Management Console to work with resources in a shared mesh at this time\.

 When using `create`, `update`, or `delete` commands for resources in a shared mesh, you need to specify the `--mesh-owner account-id` option with the command\. When using any of the App Mesh AWS CLI commands, account IDs for the `meshOwner` and `resourceOwner` properties are returned\.

## Unshare a Shared Mesh<a name="sharing-unshare"></a>

When you unshare a mesh, App Mesh disables further access to the mesh by former consumers of the mesh but does not delete the resources created by the consumers\. Once the mesh is unshared, only the mesh owner can access and delete the resources\. App Mesh prevents the account that owned resources in the mesh, and any other accounts with resources in the mesh, from receiving any configuration information after the mesh is unshared\. Only the owner of the mesh can unshare it\.

**To unshare a mesh**

1. View a list of your shared resources with the following command\.

   ```
   aws ram list-resources --resource-owner SELF
   ```

1. The *111122223333* account can unshare a mesh with all accounts by deleting the resource share\.

   ```
   aws ram delete-resource-share \
       --resource-share-arn arn:aws:ram:region-code:111122223333:resource-share/1cb1da11-011b-1c1c-11e1-11111Example
   ```

   Rather than deleting a resource share, you can remove specific principals from it with the [disassociate\-resource\-share](https://docs.aws.amazon.com/cli/latest/reference/ram/disassociate-resource-share.html) command\.

## Billing and Metering<a name="sharing-billing"></a>

There are no additional charges for sharing a mesh\.
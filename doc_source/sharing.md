# Working with Shared Meshes<a name="sharing"></a>

\([App Mesh Preview Channel](https://docs.aws.amazon.com//app-mesh/latest/userguide/preview.html) only\) A shared mesh allows resources created by different accounts to communicate with each other in the same mesh\. To learn more about using features available in the Preview Channel, see [App Mesh Preview Channel](preview.md)\. 

An AWS Identity and Access Management account can be a mesh resource owner, a mesh consumer, or both\. Consumers can create resources in a mesh that is shared with their account\. Owners can create resources in any mesh the account owns\. A mesh owner can share a mesh with the following types of mesh consumers:
+ Specific AWS accounts inside or outside its organization in AWS Organizations
+ An organizational unit inside its organization in AWS Organizations
+ Its entire organization in AWS Organizations

## Shared Mesh Permissions<a name="sharing-permissions"></a>

A shared mesh has the following permissions:
+ Consumers can list and describe all resources in a mesh that is shared with the account\.
+ Owners can list and describe all resources in any mesh the account owns\.
+ Owners and consumers can modify resources in a mesh that the account created, but cannot modify resources that other another account created\.
+ Consumers can delete any resource in a mesh that the account created\.
+ Owners can delete any resource in a mesh that any account created\. When an owner or consumer creates or updates a resource that references other AWS resources, such as AWS Cloud Map or ACM, the only resources that can be referenced are resources in the same account\. An example is a virtual node that references a TLS certificate\.
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

1. Load the preview model into the AWS CLI with the following command\.

   ```
   aws configure add-model \
       --service-name appmesh-preview \
       --service-model https://raw.githubusercontent.com/aws/aws-app-mesh-roadmap/master/appmesh-preview/service-model.json
   ```

1. Using the *111122223333* account, create a mesh in the Preview Channel with the following command\.

   ```
   aws appmesh-preview create-mesh --region us-west-2 --mesh-name mesh-a
   ```

   Note the value for `arn` that is returned in the output for use in the next step\.

1. Using the *111122223333* account, share the mesh created in the previous step with the *444455556666* account with the following command\.

   ```
   aws ram create-resource-share \
       --region us-west-2 \
       --name mesh-a \
       --principals 444455556666 \
       --resource arn:aws:appmesh:us-west-2:111122223333:mesh/mesh-a
   ```

   The output returned is similar to the following example output\.

   ```
   {
       "resourceShare": {
           "resourceShareArn": "arn:aws:ram:us-west-2:111122223333:resource-share/1cb1da11-011b-1c1c-11e1-11111Example",
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

   1. Using the the *444455556666* account, view the account's sharing invitations\.

      ```
      aws ram --region us-west-2 get-resource-share-invitations
      ```

      The output returned is similar to the following example output\.

      ```
      {
          "resourceShareInvitations": [
              {
                  "resourceShareInvitationArn": "arn:aws:ram:us-west-2:111122223333:resource-share-invitation/11111111-1111-1111-1111-11111111111",
                  "resourceShareName": "mesh-a",
                  "resourceShareArn": "arn:aws:ram:us-west-2:111122223333:resource-share/1cb1da11-011b-1c1c-11e1-11111Example",
                  "senderAccountId": "111122223333",
                  "receiverAccountId": "444455556666",
                  "invitationTimestamp": 1579145083.23,
                  "status": "PENDING"
              }
          ]
      }
      ```

   1. Using the the *444455556666* account, accept the sharing invitation\.

      ```
      aws ram --region us-west-2 accept-resource-share-invitation --resource-share-invitation-arn arn:aws:ram:us-west-2:111122223333:resource-share-invitation/11111111-1111-1111-1111-11111111111
      ```

      The output returned is similar to the following example output\.

      ```
      {
          "resourceShareInvitation": {
              "resourceShareInvitationArn": "arn:aws:ram:us-west-2:111122223333:resource-share-invitation/11111111-1111-1111-1111-11111111111",
              "resourceShareName": "mesh-a",
              "resourceShareArn": "arn:aws:ram:us-west-2:111122223333:resource-share/1cb1da11-011b-1c1c-11e1-11111Example",
              "senderAccountId": "111122223333,
              "receiverAccountId": "444455556666",
              "invitationTimestamp": 1579287509.04,
              "status": "ACCEPTED"
          }
      }
      ```

## List Meshes<a name="list-describe-meshes"></a>

You can list the meshes for an account\.

**To list meshes for an account**

1. If you haven't already, load the preview model with the following command\.

   ```
   aws configure add-model \
       --service-name appmesh-preview \
       --service-model https://raw.githubusercontent.com/aws/aws-app-mesh-roadmap/master/appmesh-preview/service-model.json
   ```

1. The *444455556666* account can list the meshes in the Preview Channel with the following command\.

   ```
   aws appmesh-preview list-meshes \
       --region us-west-2
   ```

   The output returned is similar to the following example output\.

   ```
   {
       "meshes": [
           {
               "arn": "arn:aws:appmesh-preview:us-west-2:444455556666:mesh/mesh-b",
               "meshName": "mesh-b"
           },
           {
               "arn": "arn:aws:appmesh-preview:us-west-2:111122223333:mesh/mesh-a",
               "meshName": "mesh-a"
           }
       ]
   }
   ```

   In the example output above, the *444455556666* account created the mesh named `mesh-b` and the *111122223333* account created and shared the mesh named `mesh-a`\.

## Describe a Mesh<a name="sharning-describe-meshes"></a>

You can describe a specific mesh to see its details\.

**To describe a mesh in an account**

1. If you haven't already, load the preview model with the following command\.

   ```
   aws configure add-model \
       --service-name appmesh-preview \
       --service-model https://raw.githubusercontent.com/aws/aws-app-mesh-roadmap/master/appmesh-preview/service-model.json
   ```

1. The *444455556666* account can see details of a mesh that was shared with it in the Preview Channel using the following command\.

   ```
   aws appmesh-preview describe-mesh --mesh-owner 111122223333 --mesh-name mesh-a
   ```

   Output similar to the follow example output is returned\.

   ```
   {
       "mesh": {
           "meshName": "mesh-a",
           "metadata": {
               "arn": "arn:aws:appmesh-preview:us-west-2:111122223333:mesh/mesh-a",
               "createdAt": 1573845755.718,
               "lastUpdatedAt": 1573845755.718,
               "meshOwner": "111122223333",
               "resourceOwner": "111122223333",
               "uid": "a1b2c3d4-5678-90ab-cdef-11111EXAMPLE",
               "version": 1
           },
           "spec": {},
           "status": {
               "status": "ACTIVE"
           }
       }
   }
   ```

   You can see that the `meshOwner` and `resourceOwner` are the 111122223333 account\.

## Add a Resource<a name="add-resource"></a>

The owner of the mesh, and anyone that the mesh is shared with, can create any type of mesh resource in a shared mesh\. A route can only be created for a virtual router owned by the same account that is created the route\. Complete the following steps using the *444455556666* account to add a virtual node to a shared mesh in the Preview Channel\.

**To add a resource**

1. If you haven't already, load the preview model with the following command\.

   ```
   aws configure add-model \
       --service-name appmesh-preview \
       --service-model https://raw.githubusercontent.com/aws/aws-app-mesh-roadmap/master/appmesh-preview/service-model.json
   ```

1. Create a file named `create-virtual-node.json` with the following contents\. This file will be used to create a virtual node in a mesh owned by the *111122223333* account\.

   ```
   {
      "meshName" : "mesh-a",
      "meshOwner" : "111122223333",
      "spec" : {
         "listeners" : [
            {
               "portMapping" : {
                  "port" : 80,
                  "protocol" : "http2"
               }
            }
         ],
         "serviceDiscovery" : {
            "dns" : {
               "hostname" : "virtual-node-a.apps.local"
            }
         }
      },
      "virtualNodeName" : "virtual-node-a"
   }
   ```

1. The *444455556666* account can create a virtual node in the shared mesh with the following command\.

   ```
   aws appmesh-preview create-virtual-node \
       --region us-west-2 \
       --cli-input-json file://create-virtual-node.json
   ```

   Output similar to the following example output is returned\.

   ```
   {
       "virtualNode": {
           "meshName": "mesh-a",
           "metadata": {
               "arn": "arn:aws:appmesh-preview:us-west-2:444455556666:mesh/mesh-a@111122223333/virtualNode/virtual-node-a",
               "createdAt": 1579289013.813,
               "lastUpdatedAt": 1579289013.813,
               "meshOwner": "111122223333",
               "resourceOwner": "444455556666",
               "uid": "a1b2c3d4-5678-90ab-cdef-11111EXAMPLE",
               "version": 1
           },
           "spec": {
               "listeners": [
                   {
                       "portMapping": {
                           "port": 80,
                           "protocol": "http2"
                       }
                   }
               ],
               "serviceDiscovery": {
                   "dns": {
                       "hostname": "virtual-node-a.apps.local"
                   }
               }
           },
           "status": {
               "status": "ACTIVE"
           },
           "virtualNodeName": "virtual-node-a"
       }
   }
   ```

   Notice that the account ID for the `meshOwner` is *111122223333* and the account ID for the `resourceOwner` is *444455556666*\. Both IDs are also listed as part of the `arn` for the created virtual node\.

1. The *444455556666* account can list the virtual nodes in the shared mesh with the following command\.

   ```
   aws appmesh-preview list-virtual-nodes \
       --mesh-name mesh-a \
       --region us-west-2 \
       --owner 111122223333
   ```

   Output similar to the following example output is returned\.

   ```
   {
       "virtualNodes": [
           {
               "arn": "arn:aws:appmesh-preview:us-west-2:444455556666:mesh/mesh-a@111122223333/virtualNode/virtual-node-a",
               "meshName": "mesh-a",
               "meshOwner": "111122223333",
               "resourceOwner": "444455556666",
               "virtualNodeName": "virtual-node-a"
           }
       ]
   }
   ```

1. The *444455556666* account can view the details of a virtual node that it owns with the following command\.

   ```
   aws appmesh-preview describe-virtual-node \
       --region us-west-2 \
       --mesh-name mesh-a \
       --mesh-owner 111122223333 \
       --virtual-node-name virtual-node-a
   ```

   Output similar to the following example output is returned\.

   ```
   {
       "virtualNode": {
           "meshName": "mesh-a",
           "metadata": {
               "arn": "arn:aws:appmesh-preview:us-west-2:444455556666:mesh/mesh-a@111122223333/virtualNode/virtual-node-a",
               "createdAt": 1579289013.813,
               "lastUpdatedAt": 1579289013.813,
               "meshOwner": "111122223333",
               "resourceOwner": "444455556666",
               "uid": "a1b2c3d4-5678-90ab-cdef-11111EXAMPLE",
               "version": 1
           },
           "spec": {
               "backends": [],
               "listeners": [
                   {
                       "portMapping": {
                           "port": 80,
                           "protocol": "http2"
                       }
                   }
               ],
               "serviceDiscovery": {
                   "dns": {
                       "hostname": "virtual-node-a.apps.local"
                   }
               }
           },
           "status": {
               "status": "ACTIVE"
           },
           "virtualNodeName": "virtual-node-a"
       }
   }
   ```

## Update a Resource<a name="update-resource"></a>

To update a resource in a shared mesh, your account must be the owner of the resource\. The owner of the mesh cannot update a resource that it does not own, even when the resource is in a mesh that the account does own\. 

**To update a resource**

1. If you haven't already, load the preview model with the following command\.

   ```
   aws configure add-model \
       --service-name appmesh-preview \
       --service-model https://raw.githubusercontent.com/aws/aws-app-mesh-roadmap/master/appmesh-preview/service-model.json
   ```

1. Create a file named `update-virtual-node.json` with the following contents\. The *444455556666* account owns an existing virtual node in a shared mesh that is owned by the *111122223333* account\. If you don't have an existing virtual node to update, you can create one by completing the steps in [Add a Resource](#add-resource)\.

   ```
   {
      "meshName" : "mesh-a",
      "meshOwner" : "111122223333",
      "spec" : {
         "listeners" : [
            {
               "portMapping" : {
                  "port" : 80,
                  "protocol" : "http"
               }
            }
         ],
         "serviceDiscovery" : {
            "dns" : {
               "hostname" : "virtual-node-a.apps.local"
            }
         }
      },
      "virtualNodeName" : "virtual-node-a"
   }
   ```

1. The *444455556666* account can update a virtual node that it owns in the shared mesh with the following command\.

   ```
   aws appmesh-preview update-virtual-node \
       --region us-west-2 \
       --cli-input-json file://update-virtual-node.json
   ```

## Delete a Resource<a name="delete-resource"></a>

To delete a resource from a shared mesh, your account must be the owner of the resource or the mesh\. 

**To delete a resource**

1. If you haven't already, load the preview model with the following command\.

   ```
   aws configure add-model \
       --service-name appmesh-preview \
       --service-model https://raw.githubusercontent.com/aws/aws-app-mesh-roadmap/master/appmesh-preview/service-model.json
   ```

1. The *444455556666* account can delete a virtual node that it owns from a shared mesh with the following command\.

   ```
   aws appmesh-preview delete-virtual-node \
       --region us-west-2 \
       --mesh-name mesh-a \
       --mesh-owner 111122223333 \
       --virtual-node-name virtual-node-a
   ```

## Unshare a Shared Mesh<a name="sharing-unshare"></a>

When you unshare a mesh, App Mesh disables further access to the mesh by former consumers of the mesh but does not delete the resources created by the consumers\. Once the mesh is unshared, only the mesh owner can access and delete the resources\. App Mesh prevents the account that owned resources in the mesh, and any other accounts with resources in the mesh, from receiving any configuration information after the mesh is unshared\. Only the owner of the mesh can unshare it\.

**To unshare a mesh**

1. If you haven't already, load the preview model with the following command\.

   ```
   aws configure add-model \
       --service-name appmesh-preview \
       --service-model https://raw.githubusercontent.com/aws/aws-app-mesh-roadmap/master/appmesh-preview/service-model.json
   ```

1. View a list of your shared resources with the following command\.

   ```
   aws ram list-resources --resource-owner SELF
   ```

1. The *111122223333* account can unshare a mesh with all accounts by deleting the resource share\.

   ```
   aws ram delete-resource-share \
       --resource-share-arn arn:aws:ram:us-west-2:111122223333:resource-share/1cb1da11-011b-1c1c-11e1-11111Example
   ```

   Rather than deleting a resource share, you can remove specific principals from it with the [disassociate\-resource\-share](https://docs.aws.amazon.com/cli/latest/reference/ram/disassociate-resource-share.html) command\.

## Billing and Metering<a name="sharing-billing"></a>

There are no additional charges for sharing a mesh\.
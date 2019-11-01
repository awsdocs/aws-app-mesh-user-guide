# Virtual Routers<a name="virtual_routers"></a>

Virtual routers handle traffic for one or more virtual services within your mesh\. After you create a virtual router, you can create and associate routes for your virtual router that direct incoming requests to different virtual nodes\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/app-mesh/latest/userguide/images/virtual_router.png)

Any inbound traffic that your virtual router expects should be specified as a *listener*\.

## Creating a Virtual Router<a name="create-virtual-router"></a>

To create a virtual router using the AWS Management Console, complete the following steps\. To create a virtual router using the AWS CLI version 1\.16\.266 or higher, see the example in the AWS CLI reference for the [create\-virtual\-router](https://docs.aws.amazon.com/cli/latest/reference/appmesh/create-virtual-router.html) command\.

1. Open the App Mesh console at [https://console\.aws\.amazon\.com/appmesh/](https://console.aws.amazon.com/appmesh/)\.

1. Choose the mesh that you want to create the virtual router in\.

1. Choose **Virtual routers** in the left navigation\.

1. Choose **Create virtual router**\.

1. For **Virtual router name**, specify a name for your virtual router\. Up to 255 letters, numbers, hyphens, and underscores are allowed\.

1. For **Listener**, specify a **Port** and **Protocol** for your virtual router\.

1. Choose **Create virtual router** to finish\.

## Deleting a Virtual Router<a name="delete-virtual-router"></a>

To delete a virtual router using the AWS Management Console complete the following steps\. To delete a virtual router using the AWS CLI, use the `aws appmesh delete-virtual-router` command\. For an example of deleting a virtual router using the AWS CLI, see [delete\-virtual\-router](https://docs.aws.amazon.com/cli/latest/reference/appmesh/delete-virtual-router.html)\.

**Note**  
You cannot delete a virtual router if it has any [routes](routes.md) or if it is specified as a provider for any [virtual service](virtual_services.md)\.

1. Open the App Mesh console at [https://console\.aws\.amazon\.com/appmesh/](https://console.aws.amazon.com/appmesh/)\.

1. Choose the mesh that you want to delete a virtual router from\.

1. Choose **Virtual routers** in the left navigation\.

1. In the **Virtual Routers** table, choose the virtual router that you want to delete and select **Delete**\.

1. In the confirmation box, type **delete** and then select **Delete**\.
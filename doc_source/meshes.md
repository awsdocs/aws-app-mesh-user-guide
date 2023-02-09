# Service Meshes<a name="meshes"></a>

A service mesh is a logical boundary for network traffic between the services that reside within it\. After you create your service mesh, you can create virtual services, virtual nodes, virtual routers, and routes to distribute traffic between the applications in your mesh\.

## Creating a service mesh<a name="create-mesh"></a>

**Note**  
When creating a Mesh, you must add a namespace selector\. If the namespace selector is empty, it selects all namespaces\. To restrict the namespaces, use a label to associate App Mesh resources to the created mesh\.

------
#### [ AWS Management Console ]

**To create a service mesh using the AWS Management Console**

1. Open the App Mesh console at [https://console\.aws\.amazon\.com/appmesh/](https://console.aws.amazon.com/appmesh/)\. 

1. Choose **Create mesh**\.

1. For **Mesh name**, specify a name for your service mesh\.

1. \(Optional\) Choose **Allow external traffic**\. By default, proxies in the mesh only forward traffic between each other\. If you allow external traffic, the proxies in the mesh also forward TCP traffic directly to services that aren't deployed with a proxy that is defined in the mesh\.
**Note**  
If you specify any backends on a virtual node when using `ALLOW_ALL`, you must specifiy all egress for that virtual node as backends\. Otherwise, `ALLOW_ALL` will no longer work for that virtual node\.

1. 

**IP version preference**

   Control which IP version should be used for traffic within the mesh by toggling on **Override default IP version behavior**\. By default, App Mesh uses a variety of IP versions\.
**Note**  
The mesh applies the IP preference to all of the virtual nodes and virtual gateways within a mesh\. This behavior can be overridden on a individual virtual node by setting the IP preference when you make or edit the node\. The IP preference can't be overridden on a virtual gateway because the configuration for virtual gateways that allows them to listen for both IPv4 and IPv6 traffic is the same regardless of which preference is set on the mesh\.
   + Default
     + Envoy's DNS resolver prefers `IPv6` and falls back to `IPv4`\.
     + We use the `IPv4` address returned by AWS Cloud Map if available and falls back to using the `IPv6` address\.
     + The endpoint created for the local app uses an `IPv4` address\.
     + The Envoy listeners bind to all `IPv4` addresses\.
   + IPv6 preferred
     + Envoy's DNS resolver prefers `IPv6` and falls back to `IPv4`\.
     + The `IPv6` address returned by AWS Cloud Map is used if available and falls back to using the `IPv4` address
     + The endpoint created for the local app uses an `IPv6` address\.
     + The Envoy listeners bind to all `IPv4` and `IPv6` addresses\.
   + IPv4 preferred
     + Envoy's DNS resolver prefers `IPv4` and falls back to `IPv6`\.
     + We use the `IPv4` address returned by AWS Cloud Map if available and falls back to using the `IPv6` address\.
     + The endpoint created for the local app uses an `IPv4` address\.
     + The Envoy listeners bind to all `IPv4` and `IPv6` addresses\.
   + IPv6 only
     + Envoy's DNS resolver only uses `IPv6`\.
     + Only the `IPv6` address returned by AWS Cloud Map is used\. If AWS Cloud Map returns an `IPv4` address, no IP addresses are used and empty results are returned to the Envoy\.
     + The endpoint created for the local app uses an `IPv6` address\.
     + The Envoy listeners bind to all `IPv4` and `IPv6` addresses\.
   + IPv4 only
     + Envoy's DNS resolver only uses `IPv4`\.
     + Only the `IPv4` address returned by AWS Cloud Map is used\. If AWS Cloud Map returns an `IPv6` address, no IP addresses are used and empty results are returned to the Envoy\.
     + The endpoint created for the local app uses an `IPv4` address\.
     + The Envoy listeners bind to all `IPv4` and `IPv6` addresses\.

1. Choose **Create mesh** to finish\.

1. \(Optional\) Share the mesh with other accounts\. A shared mesh allows resources created by different accounts to communicate with each other in the same mesh\. For more information, see [Working with shared meshes](sharing.md)\.

------
#### [ AWS CLI ]

**To create a mesh using the AWS CLI\.**

Create a service mesh using the following command \(replace the *red* values with your own\):

1. 

   ```
   aws appmesh create-mesh --mesh-name meshName
   ```

1. Example output:

   ```
   {
       "mesh":{
           "meshName":"meshName",
           "metadata":{
               "arn":"arn:aws:appmesh:us-west-2:123456789012:mesh/meshName",
               "createdAt":"2022-04-06T08:45:50.072000-05:00",
               "lastUpdatedAt":"2022-04-06T08:45:50.072000-05:00",
               "meshOwner": "123456789012",
               "resourceOwner": "123456789012",
               "uid":"a1b2c3d4-5678-90ab-cdef-11111EXAMPLE",
               "version":1
           },
           "spec":{},
           "status":{
               "status":"ACTIVE"
           }
       }
   }
   ```

For more information on creating a mesh with the AWS CLI for App Mesh, see the [create\-mesh](https://docs.aws.amazon.com/cli/latest/reference/appmesh/create-mesh.html) command in the AWS CLI reference\.

------

## Deleting a mesh<a name="delete-mesh"></a>

------
#### [ AWS Management Console ]

**To delete a virtual gateway using the AWS Management Console**

1. Open the App Mesh console at [https://console\.aws\.amazon\.com/appmesh/](https://console.aws.amazon.com/appmesh/)\. 

1. Choose the mesh you want to delete\. All of the meshes that you own and that have been [shared](sharing.md) with you are listed\.

1. In the confirmation box, type **delete** and then click on **Delete**\.

------
#### [ AWS CLI ]

**To delete a mesh using the AWS CLI**

1. Use the following command to delete your mesh \(replace the *red* values with your own\):

   ```
   aws appmesh delete-mesh \
        --mesh-name meshName
   ```

1. Example output:

   ```
   {
       "mesh": {
           "meshName": "meshName",
           "metadata": {
               "arn":"arn:aws:appmesh:us-west-2:123456789012:mesh/meshName",
               "createdAt": "2022-04-06T08:45:50.072000-05:00",
               "lastUpdatedAt": "2022-04-07T11:06:32.795000-05:00",
               "meshOwner": "123456789012",
               "resourceOwner": "123456789012",
               "uid": "a1b2c3d4-5678-90ab-cdef-11111EXAMPLE",
               "version": 1
           },
           "spec": {},
           "status": {
               "status": "DELETED"
           }
       }
   }
   ```

For more information on deleting a mesh with the AWS CLI for App Mesh, see the [delete\-mesh](https://docs.aws.amazon.com/cli/latest/reference/appmesh/delete-mesh.html) command in the AWS CLI reference\.

------
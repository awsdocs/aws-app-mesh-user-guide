# Virtual services<a name="virtual_services"></a>

A virtual service is an abstraction of a real service that is provided by a virtual node directly or indirectly by means of a virtual router\. Dependent services call your virtual service by its `virtualServiceName`, and those requests are routed to the virtual node or virtual router that is specified as the provider for the virtual service\.

## Creating a virtual service<a name="create-virtual-service"></a>

------
#### [ AWS Management Console ]

**To create a virtual service using the AWS Management Console**

1. Open the App Mesh console at [https://console\.aws\.amazon\.com/appmesh/](https://console.aws.amazon.com/appmesh/)\. 

1. Choose the mesh in which you want to create the virtual service\. All of the meshes that you own and that have been [shared](sharing.md) with you are listed\.

1. Choose **Virtual services** in the left navigation\.

1. Choose **Create virtual service**\.

1. For **Virtual service name**, choose a name for your virtual service\. You can choose any name, but the service discovery name of the real service that you're targeting, such as `my-service.default.svc.cluster.local`, is recommended to make it easier to correlate your virtual services to real services\. This way you don't need to change your code to reference a different name than your code currently references\. The name that you specify must resolve to a non\-loopback IP address because the app container must be able to successfully resolve the name before the request is sent to the Envoy proxy\. You can use any non\-loopback IP address because neither the app or proxy containers communicate with this IP address\. The proxy communicates with other virtual services through the names you’ve configured for them in App Mesh, not through IP addresses to which the names resolve\.

1. For **Provider**, choose the provider type for your virtual service:
   + If you want the virtual service to spread traffic across multiple virtual nodes, select **Virtual router** and then choose the virtual router to use from the drop\-down menu\.
   + If you want the virtual service to reach a virtual node directly without a virtual router, select **Virtual node** and then choose the virtual node to use from the drop\-down menu\.
**Note**  
App Mesh may automatically create a default Envoy route retry policy for each virtual node provider that you define on or after July 29, 2020, even though you can't define such a policy through the App Mesh API\. For more information, see [Default route retry policy](envoy-defaults.md#default-retry-policy)\.
   + If you don't want the virtual service to route traffic at this time \(for example, if your virtual nodes or virtual router doesn't exist yet\), choose **None**\. You can update the provider for this virtual service later\.

1. Choose **Create virtual service** to finish\.

------
#### [ AWS CLI ]

**To create a virtual service using the AWS CLI\.**

Create a virtual service with a virtual node provider using the following command and an input JSON file \(replace the *red* values with your own\):

1. 

   ```
   aws appmesh create-virtual-service \ 
   --cli-input-json file://create-virtual-service-virtual-node.json
   ```

1. Contents of **example** create\-virtual\-service\-virtual\-node\.json:

   ```
   {
       "meshName": "meshName",
       "spec": {
           "provider": {
               "virtualNode": {
                   "virtualNodeName": "nodeName"
               }
           }
       },
       "virtualServiceName": "serviceA.svc.cluster.local"
   }
   ```

1. Example output:

   ```
   {
       "virtualService": {
           "meshName": "meshName",
           "metadata": {
               "arn": "arn:aws:appmesh:us-west-2:210987654321:mesh/meshName/virtualService/serviceA.svc.cluster.local",
               "createdAt": "2022-04-06T09:45:35.890000-05:00",
               "lastUpdatedAt": "2022-04-06T09:45:35.890000-05:00",
               "meshOwner": "123456789012",
               "resourceOwner": "210987654321",
               "uid": "a1b2c3d4-5678-90ab-cdef-11111EXAMPLE",
               "version": 1
           },
           "spec": {
               "provider": {
                   "virtualNode": {
                       "virtualNodeName": "nodeName"
                   }
               }
           },
           "status": {
               "status": "ACTIVE"
           },
           "virtualServiceName": "serviceA.svc.cluster.local"
       }
   }
   ```

For more information on creating a virtual service with the AWS CLI for App Mesh, see the [create\-virtual\-service](https://docs.aws.amazon.com/cli/latest/reference/appmesh/create-virtual-service.html) command in the AWS CLI reference\.

------

## Deleting a virtual service<a name="delete-virtual-service"></a>

**Note**  
You can't delete a virtual service that is referenced by a gateway route\. You need to delete the gateway route first\. 

------
#### [ AWS Management Console ]

**To delete a virtual service using the AWS Management Console**

1. Open the App Mesh console at [https://console\.aws\.amazon\.com/appmesh/](https://console.aws.amazon.com/appmesh/)\. 

1. Choose the mesh from which you want to delete a virtual service\. All of the meshes that you own and that have been [shared](sharing.md) with you are listed\.

1. Choose **Virtual services** in the left navigation\.

1. Choose the virtual service that you want to delete and click on **Delete** in the top right corner\. You can only delete a virtual gateway where your account is listed as **Resource owner**\.

1. In the confirmation box, type **delete** and then click on **Delete**\.

------
#### [ AWS CLI ]

**To delete a virtual service using the AWS CLI**

1. Use the following command to delete your virtual service \(replace the *red* values with your own\):

   ```
   aws appmesh delete-virtual-service \
        --mesh-name meshName \
        --virtual-service-name serviceA.svc.cluster.local
   ```

1. Example output:

   ```
   {
       "virtualService": {
           "meshName": "meshName",
           "metadata": {
               "arn": "arn:aws:appmesh:us-west-2:210987654321:mesh/meshName/virtualService/serviceA.svc.cluster.local",
               "createdAt": "2022-04-06T09:45:35.890000-05:00",
               "lastUpdatedAt": "2022-04-07T10:39:42.772000-05:00",
               "meshOwner": "123456789012",
               "resourceOwner": "210987654321",
               "uid": "a1b2c3d4-5678-90ab-cdef-11111EXAMPLE",
               "version": 2
           },
           "spec": {
               "provider": {
                   "virtualNode": {
                       "virtualNodeName": "nodeName"
                   }
               }
           },
           "status": {
               "status": "DELETED"
           },
           "virtualServiceName": "serviceA.svc.cluster.local"
       }
   }
   ```

For more information on deleting a virtual service with the AWS CLI for App Mesh, see the [delete\-virtual\-service](https://docs.aws.amazon.com/cli/latest/reference/appmesh/delete-virtual-service.html) command in the AWS CLI reference\.

------
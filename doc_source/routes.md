# Routes<a name="routes"></a>

A route is associated with a virtual router\. It is used to match requests for the virtual router and distribute traffic to its associated virtual nodes\. If a route matches a request, it can distribute traffic to one or more target virtual nodes\. You can specify relative weighting for each virtual node\. This topic helps you work with routes in a service mesh\.

## Creating a Route<a name="create-route"></a>

You can use the AWS Management Console or the AWS CLI to create a route\. Select the name of the tool that you want to use to create a route\.

------
#### [ AWS Management Console ]

1. Open the App Mesh console at [https://console\.aws\.amazon\.com/appmesh/](https://console.aws.amazon.com/appmesh/)\.

1. Choose the mesh that you want to create the route in\.

1. Choose **Virtual routers** in the left navigation\.

1. Choose the virtual router that you want to associate a new route with\. 

1. In the **Routes** table, choose **Create route**\.

1. For **Route name**, specify the name to use for your route\.

1. For **Route type**, choose the protocol for your route\.

1. For **Virtual node name**, choose the virtual node that this route will serve traffic to\. If none are listed, then you need to [create a virtual node](https://docs.aws.amazon.com//app-mesh/latest/userguide/virtual_nodes.html) first\.

1. For **Weight**, choose a relative weight for the route\. Select **Add target** to add additional virtual nodes\. The total weight for all targets combined must be less than or equal to 100\.

1. To use HTTP path\-based routing, choose **Additional configuration** and then specify the path that the route should match\. For additional information about path\-based routing, see [Path\-based Routing](route-path.md)\. 

1. Choose **Create route** to finish\.

------
#### [ AWS CLI ]

1. Create a JSON file with a route configuration\. The following example JSON will create a route with two targets\. One target is version 1 of a service, while the other target is version 2 of the service\. Ninety percent of traffic is sent to one of the targets, while the remaining ten percent is sent to the other target\. A route such as this allows you to test a new version of a service with a small amount of traffic to decrease the impact of bugs in new service code\. The total weight for all targets must be less than or equal to 100\.

   ```
   {
      "meshName" : "app1",
      "routeName" : "route-weighted1",
      "spec" : {
         "httpRoute" : {
            "action" : {
               "weightedTargets" : [
                  {
                     "virtualNode" : "serviceBv1",
                     "weight" : 90
                  },
                  {
                     "virtualNode" : "serviceBv2",
                     "weight" : 10
                  }
               ]
            },
            "match" : {
               "prefix" : "/"
            }
         }
      },
      "virtualRouterName" : "virtual-router1"
   }
   ```

1. Create the route with the following command\.

   ```
   aws appmesh create-route --cli-input-json file://route.json
   ```

\([App Mesh Preview Channel](https://docs.aws.amazon.com//app-mesh/latest/userguide/preview.html) only\) If you want to assign a priority to the route, add `"priority": value-between-1-1000` to a JSON file for a route\. Routes are matched based on the specified value, where 0 is the highest priority\. If no value is specified, then the route with the longest prefix is selected\. [Download](https://raw.githubusercontent.com/aws/aws-app-mesh-roadmap/master/appmesh-preview/service-model.json) the service model for use with this feature\. For instructions on how to use features in the Preview Channel, see [How can I use features in the Preview Channel?](preview.md#try-out)\.

For JSON examples of different routing options, see [Path\-based Routing](route-path.md) and [HTTP Headers](route-http-headers.md)\.  

You can always find the latest options that you can specify in an input JSON file by running the `aws appmesh create-route --generate-cli-skeleton` command\. 

------

## Deleting a Route<a name="delete-route"></a>

You can use the AWS Management Console or the AWS CLI to delete a route\. Select the name of the tool that you want to use to delete a route\.

------
#### [ AWS Management Console ]

1. Open the App Mesh console at [https://console\.aws\.amazon\.com/appmesh/](https://console.aws.amazon.com/appmesh/)\.

1. Choose the mesh that you want to delete a route from\.

1. Choose **Virtual routers** in the left navigation\.

1. Choose the router that you want to delete a route from\.

1. In the **Routes** table, choose the route that you want to delete and select **Delete**\.

1. In the confirmation box, type `delete` and then select **Delete**\.

------
#### [ AWS CLI ]

Use the [appmesh delete\-route](https://docs.aws.amazon.com/cli/latest/reference/appmesh/delete-route.html) command to delete a route\. The following example command deletes a route named *route\-weighted1*\.

```
aws appmesh delete-route --mesh-name app1 \
    --virtual-router-name virtual-router1 \
    --route-name route-weighted1
```

------
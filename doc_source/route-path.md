# Path\-based Routing<a name="route-path"></a>

You can route traffic based on the path in the request\. You can use the AWS Management Console or the AWS CLI to create a route\. Select the name of the tool that you want to use to create a route\.

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

1. To use HTTP path\-based routing, choose **Additional configuration** and then specify the path that the route should match\. For example, if your virtual service name is `service-b.local` and you want the route to match requests to `service-b.local/metrics`, your prefix should be `/metrics`\.

1. Choose **Create route** to finish\.

------
#### [ AWS CLI ]

1. Create a JSON file with a route configuration\. The following example JSON will create a route that routes all traffic that has */metrics* in its request path to a virtual node named *serviceB*\.

   ```
   {
      "meshName" : "app1",
      "routeName" : "route-path1",
      "spec" : {
         "httpRoute" : {
            "action" : {
               "weightedTargets" : [
                  {
                     "virtualNode" : "serviceB",
                     "weight" : 100
                  }
               ]
            },
            "match" : {
               "prefix" : "/metrics"
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

For more information about creating routes with the AWS CLI, see [create\-route](https://docs.aws.amazon.com/cli/latest/reference/appmesh/create-route.html) in the *AWS Command Line Interface User Guide*\.

You can always find the latest options that you can specify in an input JSON file by running the `aws appmesh create-route --generate-cli-skeleton` command\. 

------
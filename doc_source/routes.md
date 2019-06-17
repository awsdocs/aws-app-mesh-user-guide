# Routes<a name="routes"></a>

A route is associated with a virtual router, and it's used to match requests for a virtual router and distribute traffic accordingly to its associated virtual nodes\.

You can use the `prefix` parameter in your route specification for path\-based routing of requests\. For example, if your virtual service name is `my-service.local` and you want the route to match requests to `my-service.local/metrics`, your prefix should be `/metrics`\.

If your route matches a request, you can distribute traffic to one or more target virtual nodes with relative weighting\.

## Creating a Route<a name="create-route"></a>

This topic helps you to create a route in your service mesh\.

**Creating a route in the AWS Management Console\.**

1. Open the App Mesh console at [https://console\.aws\.amazon\.com/appmesh/](https://console.aws.amazon.com/appmesh/)\.

1. Choose the mesh that you want to create the route in\.

1. Choose **Virtual routers** in the left navigation\.

1. Choose the router that you want to associate a new route with\.

1. In the **Routes** table, choose **Create route**\.

1. For **Route name**, specify the name to use for your route\.

1. For **Route type**, choose the protocol for your route\.

1. For **Virtual node name**, choose the virtual node that this route will serve traffic to\.

1. For **Weight**, choose a relative weight for the route\. The total weight for all routes must be less than 100\.

1. To use HTTP path\-based routing, choose **Additional configuration** and then specify the path that the route should match\. For example, if your virtual service name is `my-service.local` and you want the route to match requests to `my-service.local/metrics`, your prefix should be `/metrics`\.

1. Choose **Create route** to finish\.

**Creating a route in the AWS CLI\.**
+ The following JSON represents a route named `serviceB-route` for the virtual router `serviceB`\. This route directs traffic to two virtual nodes: 90% to `serviceBv1` and 10% to `serviceBv2`\. This canary\-style deployment allows you to test a new version of your service with a small amount of traffic to decrease the blast radius of new service code\.

  ```
  {
    "meshName": "simpleapp",
    "routeName": "serviceB-route",
    "spec": {
      "httpRoute": {
        "action": {
          "weightedTargets": [
            {
              "virtualNode": "serviceBv1",
              "weight": 90
            },
            {
              "virtualNode": "serviceBv2",
              "weight": 10
            }
          ]
        },
        "match": {
          "prefix": "/"
        }
      }
    },
    "virtualRouterName": "serviceB"
  }
  ```

  If you save the preceding JSON as a file, you can create the route with the following AWS CLI command\.

  ```
  aws appmesh create-route --cli-input-json file://serviceB-routes.json
  ```
**Note**  
You must use at least version 1\.16\.178 of the AWS CLI with App Mesh\. To install the latest version of the AWS CLI, see [Installing the AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/installing.html) in the *AWS Command Line Interface User Guide*\.

  For more information about creating routes with the AWS CLI, see [create\-route](https://docs.aws.amazon.com/cli/latest/reference/appmesh/create-route.html) in the *AWS Command Line Interface User Guide*\.
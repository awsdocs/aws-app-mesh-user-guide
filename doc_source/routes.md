# Routes<a name="routes"></a>

A route is associated with a virtual router, and it is used to match requests for a virtual router and distribute traffic accordingly to its associated virtual nodes\.

You can use the `prefix` parameter in your route specification for path\-based routing of requests\. For example, if your virtual service name is `my-service.local`, and you want the route to match requests to `my-service.local/metrics`, then your prefix should be `/metrics`\.

If your route matches a request, you can distribute traffic to one or more target virtual nodes with relative weighting\.

The following JSON represents a route called `serviceB-route`, for the virtual router `serviceB`\. This route directs traffic to two virtual nodes; 90% to `serviceBv1`, and 10% to `serviceBv2`\. This canary\-style deployment allows you to test a new version of your service with a small amount of traffic to decrease the blast radius of new service code\.

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

If the above JSON is saved as a file, you can create the route with the following AWS CLI command\.

```
aws appmesh create-route --cli-input-json file://serviceB-routes.json
```

**Note**  
You must use at least version 1\.16\.120 of the AWS CLI with App Mesh\. To install the latest version of the AWS CLI, see [Installing the AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/installing.html) in the *AWS Command Line Interface User Guide*\.

For more information about creating routes with the AWS CLI, see [create\-route](https://docs.aws.amazon.com/cli/latest/reference/appmesh/create-route.html) in the *AWS Command Line Interface User Guide*\.
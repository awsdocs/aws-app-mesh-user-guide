# HTTP Headers<a name="route-http-headers"></a>

\([App Mesh Preview Channel](https://docs.aws.amazon.com//app-mesh/latest/userguide/preview.html) only\) You can route traffic based on the presence and values of headers in a request\. [Download](https://raw.githubusercontent.com/aws/aws-app-mesh-roadmap/master/appmesh-preview/service-model.json) the service model for use with this feature\. For instructions on how to use features in the Preview Channel, see [How can I use features in the Preview Channel?](preview.md#try-out)\. You can also view the [GitHub issue](https://github.com/aws/aws-app-mesh-roadmap/issues/15) that this feature is based on\.

Create a JSON file with a route configuration\. The following example JSON will route all requests to *`serviceB`* that have any path prefix in an *HTTPS** post* where the `clientRequestId` header has a `prefix` of `123`\.

```
{
   "meshName" : "app1",
   "routeName" : "route-headers1",
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
            "headers" : [
               {
                  "invert" : false,
                  "match" : {
                     "prefix" : "123"
                  },
                  "name" : "clientRequestId"
               }
            ],
            "method" : "POST",
            "prefix" : "/",
            "scheme" : "https"
         }
      }
   },
   "virtualRouterName" : "virtual-router1"
}
```

Valid values for `method` are: `GET`, `HEAD`, `POST`, `PUT`, `DELETE`, `CONNECT`, `OPTIONS`, `TRACE`, and `PATCH`\. Valid values for `scheme` are `HTTP` and `HTTPS`\. Valid values for `match` type are: `exact`, `regex`, `range`, `prefix`, and `suffix`\. For `name`, specify the name of the HTTP header that you want to match on\. If you set `invert` to `true`, the match is on the *opposite* of the match type and value\.
# Retry Policy<a name="route-retry-policy"></a>

\([App Mesh Preview Channel](https://docs.aws.amazon.com//app-mesh/latest/userguide/preview.html) only\) A retry policy enables clients to protect themselves from intermittent network failures or intermittent server\-side failures\. You can add retry logic to a route\. Before you use this feature, [download](https://raw.githubusercontent.com/aws/aws-app-mesh-roadmap/master/appmesh-preview/service-model.json) the service model\.

 For instructions on how to use features in the Preview Channel, see [How can I use features in the Preview Channel?](preview.md#try-out)\. You can also view the [GitHub issue](https://github.com/aws/aws-app-mesh-roadmap/issues/7) that this feature is based on\. 

To add retry logic to a route, create a JSON file with a route configuration\. In the following JSON example, a route is created that attempts to route traffic *3* times when it receives a TCP `connection error` or an HTTP `server-error` or `gateway-error`\. Each retry attempt waits *15* *seconds* before timing out\. 

```
{
   "meshName" : "App1",
   "routeName" : "Route-retries1",
   "spec" : {
      "httpRoute" : {
         "action" : {
            "weightedTargets" : [
               {
                  "virtualNode" : "ServiceB",
                  "weight" : 100
               }
            ]
         },
         "match" : {
            "prefix" : "/"
         },
         "retryPolicy" : {
            "perRetryTimeout" : {
               "value": 15,
               "unit": "s"
            },
            "maxRetries" : 3,
            "httpRetryEvents" : [ "server-error", "gateway-error" ],
            "tcpRetryEvents" : [ "connection-error" ]
         }
      }
   },
   "virtualRouterName" : "Virtual-router1"
}
```

You must specify at least one value for `httpRetryEvents`, or at least one value for `tcpRetryEvents`\. You can also specify at least one value for each setting\. The only valid value for `tcpRetryEvents` is `connection-error`\. Valid values for `httpRetryEvents` include the following: 
+ `server-error` – HTTP status codes `500`, `501`, `502`, `503`, `504`, `505`, `506`, `507`, `508`, `510`, and `511`
+ `gateway-error` – HTTP status codes `502`, `503`, and `504`
+ `client-error` – HTTP status code `409`
+ `stream-error` – Retry on refused stream

The `maxRetries` value is optional, and the default is `1`\. The `perRetryTimeout` settings are optional, and the default is `15s`\. Valid values for `unit` include the following:
+ `ms` – milliseconds
+ `s` – seconds
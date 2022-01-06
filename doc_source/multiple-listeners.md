# App Mesh Preview Channel only – Multiple listeners<a name="multiple-listeners"></a>

Depending on your specific requirements, your applications might need to expose multiple ports\. For example, this is true if your application supports multiple protocols\. It's also the case if your application must access multiple domains, health checks, or application metrics\. For these use cases, App Mesh now supports multiple listeners for virtual gateways, virtual nodes, and virtual routers\. To use this feature, specify a match port and a target port in both your routes and gateway routes\.

## Creating a mesh for an application with multiple listeners\.<a name="multiple-listeners-walkthrough"></a>

To set multiple listeners on a resource, you must specify the gateway routes and routes\. Doing so ensures that App Mesh sends your traffic to the correct destination listener\. More specifcally, for routes that handle or send traffic to a resource with multiple listeners, set a match and a target port\.

For a full walkthrough with a running Amazon ECS cluster and mesh example, please see the [multiple listeners walkthrough](https://github.com/aws/aws-app-mesh-examples/tree/main/walkthroughs/howto-multiple-listeners)\.

**Important**  
Multiple listeners is an upcoming feature and is not generally available yet\. To use the feature in the preview channel, you must install the [App Mesh Preview Channel](https://docs.aws.amazon.com/app-mesh/latest/userguide/preview.html) service model for your AWS CLI\.

### Create multiple listeners<a name="creating-multiple-listeners"></a>

1. Create virtual gateways, virtual routes, and virtual nodes with the necessary listeners\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/app-mesh/latest/userguide/images/multi-list-1.png)

   ```
   # Example Virtual Node
   {
       "spec": {
             "listeners": [
                { # First listener
                  "portMapping": {
                    "port": 5050,
                    "protocol": "http"
                  }
                },
               { # Second listener
                 "portMapping": {
                   "port": 6060,  
                   "protocol": "http"
                 }
               }
             ],
             "serviceDiscovery": {
                "dns": {
                   "hostname": "my-application.local"
                }
             }
       }
   }
   ```

1. Create the virtual service with the appropriate virtual routers and virtual nodes\. Then, set both the virtual routes and virtual nodes as providers\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/app-mesh/latest/userguide/images/multi-list-2.png)

1. Create routes both from the virtual gateways to the virtual services and from the vitual routers to the virtual nodes\. For routes that receive traffic from a resource with multiple listeners, you must specify the `MatchPort` value\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/app-mesh/latest/userguide/images/multi-list-3.png)

   ```
   # Example Gateway Route
   {
       "spec": {
           "httpRoute" : {
               "match" : {
                   "port": 1010 # The Virtual Gateway Listener match port
               },
               "action" : {
                   "target" : {
                       "virtualService": {
                           "virtualServiceName": "my-virtual-service"
                       },
                       "port": 3030 # Target listener port from the Virtual service provider
                   }
               }
           }
       }
   }
   ```
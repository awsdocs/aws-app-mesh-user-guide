# App Mesh Preview Channel only – Multiple listeners<a name="multiple-listeners"></a>

Depending on your specific requirements, your applications might need to expose multiple ports\. For example, this is true if your application supports multiple protocols\. It's also the case if your application must access multiple domains, health checks, or application metrics\. For these use cases, App Mesh now supports multiple listeners for virtual gateways, virtual nodes, and virtual routers\. To use this feature, specify a match port and a target port in both your routes and gateway routes\.

## Multiple listeners walkthrough<a name="multiple-listeners-walkthrough"></a>

To set multiple listeners on a resource, you must specify the gateway routes and routes\. Doing so ensures that App Mesh sends your traffic to the correct destination listener\. More specifcally, for routes that handle or send traffic to a resource with multiple listeners, set a match and a target port\.

### Create multiple listeners<a name="creating-multiple-listeners"></a>

1. Create virtual gateways, virtual routes, and virtual nodes with the necessary listeners\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/app-mesh/latest/userguide/images/multi-list-1.png)

   ```
   aws appmesh create-virtual-gateway --mesh-name my-mesh \
     --virtual-gateway-name my-virtual-gateway
     --cli-input-json file://virtual-gateway-spec.json
   
   # virtual-gateway-spec.json
   {
       "spec": {
         "listeners": [
           {
             "portMapping": {
               "port": 1010,
               "protocol": "http"
             }
           },
           {
             "portMapping": {
               "port": 2020,
               "protocol": "http"
             }
           }
         ]
       }
   }
   
   aws appmesh create-virtual-router --mesh-name my-mesh \
     --virtual-router-name my-virtual-router
     --cli-input-json file://virtual-router-spec.json
   
   # virtual-router-spec.json
   {
       "spec": {
         "listeners": [
           {
             "portMapping": {
               "port": 3030,
               "protocol": "http"
             }
           },
           {
             "portMapping": {
               "port": 4040,
               "protocol": "http"
             }
           }
         ]
       }
   }
   
   aws appmesh create-virtual-node --mesh-name my-mesh \
     --virtual-node-name my-virtual-node
     --cli-input-json file://virtual-node-spec.json
       
   # Virtual Node spec
   {
       "spec": {
             "listeners": [
                {
                  "portMapping": {
                    "port": 5050,
                    "protocol": "http"
                  }
                },
               {
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

   ```
   aws appmesh create-virtual-service --mesh-name my-mesh \
     --virtual-service-name my-virtual-service
     --cli-input-json file://virtual-service-spec.json
   
   # virtual-service-spec.json
   {
       "spec": {
             "provider": {
                "virtualRouter": {
                   "virtualRouterName": "my-virtual-router"
                }
             }
       }
   }
   ```

1. Create routes both from the virtual gateways to the virtual services and from the vitual routers to the virtual nodes\. For routes that receive traffic from a resource with multiple listeners, you must specify the `MatchPort` value\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/app-mesh/latest/userguide/images/multi-list-3.png)

   ```
   aws appmesh create-gateway-route --mesh-name my-mesh \
     --virtual-gateway-name my-virtual-gateway \
     --gateway-route-name my-gateway-route-1
     --cli-input-json file://gateway-route-spec.json
   
   # Gateway Route spec
   {
       "spec": {
           "httpRoute" : {
               "match" : {
                   "port": 1010
               },
               "action" : {
                   "target" : {
                       "virtualService": {
                           "virtualServiceName": "my-virtual-service"
                       },
                       "port": 3030
                   }
               }
           }
       }
   }
   aws appmesh create-route --mesh-name my-mesh \
     --virtual-router-name my-virtual-router \
     --route-name my-route-1
     --cli-input-json file://route-spec.json
   # route-spec.json
   {
       "spec": {
           "httpRoute": {
               "action": {
                   "weightedTargets": [
                       {
                           "virtualNode": "my-virtual-node",
                           "weight": 1,
                           "port": 5050
                       }
                   ]
               },
               "match": {
                   "port": 3030
               }
           }
       }
   }
   ```

### Migrate an existing mesh to use multiple listeners<a name="migrating-multiple-listeners"></a>

The followingis a mesh configuration for a gateway and a node that has an application container and an admin container\.

**Note**  
The steps described in this topic only apply to the App Mesh config\. They don’t show how you can modify your application during a migration\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/app-mesh/latest/userguide/images/multi-list-4.png)

1. Before adding multiple listeners to an existing application, explicitly match routes to the existing single listeners\. This clarifies the mesh routing so that you can add a second listener to the virtual gateway, virtual node, and virtual router\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/app-mesh/latest/userguide/images/multi-list-5.png)

1. Now, the existing routes have a specific listener to target\. So, you can add a second listener to the virtual gateway, virtual node, and virtual router\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/app-mesh/latest/userguide/images/multi-list-6.png)

1. Update the routes to use the new listeners\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/app-mesh/latest/userguide/images/multi-list-7.png)
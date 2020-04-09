# Virtual gateways<a name="virtual_gateways"></a>

\([App Mesh Preview Channel](https://docs.aws.amazon.com/app-mesh/latest/userguide/preview.html) only\) A virtual gateway allows resources outside your mesh to communicate to resources that are inside your mesh\. The virtual gateway represents an Envoy proxy running in an Amazon ECS, in a Kubernetes service, or on an Amazon EC2 instance\. Unlike a virtual node, which represents a proxy running with an application, a virtual gateway represents the proxy deployed by itself\. 

External resources must be able to resolve a DNS name to an IP address assigned to the service or instance that runs the proxy\. The proxy can then access all of the App Mesh configuration for resources that are inside of the mesh\. To learn more about the related GitHub issue, see [Use App Mesh for ingress routing](https://github.com/aws/aws-app-mesh-roadmap/issues/37)\. To complete an end\-to\-end walkthrough, see [Configuring Ingress Gateway](https://github.com/aws/aws-app-mesh-examples/tree/master/walkthroughs/howto-ingress-gateway)\.

## Creating a virtual gateway<a name="vg-create-virtual-gateway"></a>

**To create a virtual gateway and route**

1. Add the Preview Channel service model to the AWS CLI with the following command\.

   ```
   aws configure add-model \
       --service-name appmesh-preview \
       --service-model https://raw.githubusercontent.com/aws/aws-app-mesh-roadmap/master/appmesh-preview/service-model.json
   ```

1. Create a virtual gateway\.

   1. Create a JSON file named `virtual-gateway.json` with a virtual gateway configuration\. In the following example configuration, the virtual gateway is created in an existing mesh named `myApps`\. Though not shown in the following example configuration, you can optionally do the following:
      + Specify that the virtual gateway use Transport Layer Security \(TLS\)\. Resources outside of the mesh can connect to the gateway using TLS, and then the gateway can communicate with other proxies within the mesh using TLS\. For more information about using TLS with App Mesh resources, see [Transport Layer Security \(TLS\)](tls.md)\. 
      + Define a health check policy for a virtual gateway\. For more information see [HealthCheckPolicy](https://docs.aws.amazon.com/app-mesh/latest/APIReference/API_HealthCheckPolicy.html) in the AWS App Mesh API Reference\. 

      You can only specify one listener for each gateway\. If you want to route traffic over multiple ports or protocols, then you must create a separate virtual gateway for each port and protocol\. You can specify `grpc` or `http` protocols\.

      ```
      {
         "meshName" : "myApps",
         "spec" : {
            "listeners" : [
               {
                  "portMapping" : {
                     "port" : 80,
                     "protocol" : "grpc"
                  }
               }
            ]
         },
         "virtualGatewayName" : "myVirtualGateway"
      }
      ```

      You can view all of the available settings for a virtual gateway with the following command\.

      ```
      aws appmesh-preview create-virtual-gateway --generate-cli-skeleton
      ```

   1. Create the virtual gateway\.

      ```
      aws appmesh-preview create-virtual-gateway --cli-input-json file://virtual-gateway.json
      ```

1. Create a gateway route\. A gateway route is attached to a virtual gateway and routes traffic to an existing virtual service\.

   1. Create a JSON file named `gateway-route.json`\. Select the tab name of the protocol that you'd like to route\. The route is configured to use the virtual gateway that you created in the previous step and to route all traffic to a virtual service named `myservicea.svc.cluster.local`\.

------
#### [ gRPC  ]

      ```
      {
         "gatewayRouteName" : "myGatewayRoute",
         "meshName" : "myApps",
         "spec" : {
            "grpcRoute" : {
               "action" : {
                  "target" : {
                     "virtualService" : {
                        "virtualServiceName" : "myservicea.svc.cluster.local"
                     }
                  }
               },
               "match" : {
                  "serviceName" : "myservicea.svc.cluster.local"
               }
            }
         },
         "virtualGatewayName" : "myVirtualGateway"
      }
      ```

      Depending on how you configure your virtual service, it could route the request to different virtual nodes based on method name or specific metadata, for example\. To learn more about virtual services, see [Virtual services](virtual_services.md)\.

------
#### [ HTTP/2 or HTTP ]

      ```
      {
         "gatewayRouteName" : "myGatewayRoute",
         "meshName" : "myApps",
         "spec" : {
            "http2Route" : {
               "action" : {
                  "target" : {
                     "virtualService" : {
                        "virtualServiceName" : "myservicea.svc.cluster.local"
                     }
                  }
               },
               "match" : {
                  "prefix" : "/"
               }
            }
         },
         "virtualGatewayName" : "myVirtualGateway"
      }
      ```

      A matched request by a gateway route is rewritten to the target virtual service's name and the matched prefix is rewritten to `/`, by default\. For example, a request to `www.example.com/chapter/page` would match the prefix in the previous example\. The request would be rewritten to `myservicea.svc.cluster.local/chapter/page`\. 

      Depending on how you configure your virtual service, it could use a virtual router to route the request to different virtual nodes, based on specific prefixes or headers\. If the prefix in the gateway route example were `/chapter`, rather than `/`:
      + A request to `www.example.com/chapter/page` would be re\-written to `myservicea.svc.cluster.local/page`\. Since `chapter` was the matched prefix, it would be removed in the re\-written request\. 
      + A request to `www.example.com/chapter1` would be rewritten to `myservicea.svc.cluster.local/1`\. Again, `chapter` was removed, since it was the matched prefix, but `/1` would be added to the rewritten request from the original request\. 

      Provided the virtual service’s virtual router has a route defined with a prefix of `/1` or `/page`, the request would be routed to a virtual node\. If the virtual service's virtual router doesn't have such a route however, the request would be dropped\. This behavior enables you to minimize the number of gateway routes that are needed, but does require you to ensure that you have routes for a virtual service’s virtual router for each of the expected prefixes that may be requested\. Learn more about [Virtual services](virtual_services.md), [Virtual routers](virtual_routers.md), and [Routes](routes.md)\.

------

      You can view all of the available settings for a gateway route with the following command\.

      ```
      aws appmesh-preview create-gateway-route --generate-cli-skeleton
      ```

   1. Create the gateway route with the following command\.

      ```
      aws appmesh-preview create-gateway-route --cli-input-json file://gateway-route.json
      ```

1. Deploy an Amazon ECS or Kubernetes service that contains only the [Envoy container](envoy.md)\. You can also deploy the Envoy container on an [Amazon EC2](https://docs.aws.amazon.com/app-mesh/latest/userguide/appmesh-getting-started.html#update-services) instance\. For more information see [Getting started with App Mesh and Amazon ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/appmesh-getting-started.html#update-services) or [Getting started with App Mesh and Kubernetes](https://docs.aws.amazon.com/eks/latest/userguide/appmesh-getting-started.html#update-services)\. You need to set the `APPMESH_VIRTUAL_NODE_NAME` environment variable to `mesh/mesh-name/virtualGateway/virtual-gateway-name`\.

   We recommend that you deploy multiple instances of the container and set up a Network Load Balancer to load balance traffic to the instances\. The service discovery name of the load balancer is the name that you want external services to use to access resources that are in the mesh, such as *myapp\.apps\.com*\. For more information see [Creating a Network Load Balancer](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/create-network-load-balancer.html) \(Amazon ECS\), [Creating an External Load Balancer](https://kubernetes.io/docs/tasks/access-application-cluster/create-external-load-balancer/) \(Kubernetes\), or [Tutorial: Increase the availability of your application on Amazon EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-increase-availability.html)\.

   Enable proxy authorization for the proxy\. For more information, see [Proxy authorization](proxy-authorization.md)\.
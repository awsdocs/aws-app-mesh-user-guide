# Getting Started with AWS App Mesh<a name="appmesh-getting-started"></a>

This topic helps you use AWS App Mesh with an actual service that is running on Amazon ECS, Kubernetes, or Amazon EC2\. 

## Scenario<a name="scenario"></a>

To illustrate how to use App Mesh, assume that you have an application with the following characteristics:
+ Includes two services named `serviceA` and `serviceB`\. 
+ Both services are registered to a namespace named `apps.local`\.
+ `ServiceA` communicates with `serviceB` over HTTP/2, port 80\.
+  You've already deployed version 2 of `serviceB` and registered it with the name `serviceBv2` in the `apps.local` namespace\.

You have the following requirements:
+ You want to send 75 percent of the traffic from `serviceA` to `serviceB` and 25 percent of the traffic to `serviceBv2` to ensure that `serviceBv2` is bug free before you send 100 percent of the traffic from `serviceA` to it\. 
+ You want to be able to easily adjust the traffic weighting so that 100 percent of the traffic goes to `serviceBv2` once it's proven to be reliable\. Once all traffic is being sent to `serviceBv2`, you want to deprecate `serviceB`\.
+ You don't want to have to change any existing application code or service discovery registration for your actual services to meet the previous requirements\. 

To meet your requirements, you've decided to create an App Mesh service mesh with virtual services, virtual nodes, a virtual router, and a route\. After implementing your mesh, you update the  services hosting your actual services to use the Envoy proxy\. Once updated, your services communicate with each other through the Envoy proxy rather than directly with each other\.

## Prerequisites<a name="prerequisites"></a>

App Mesh supports Linux services that are registered with DNS, AWS Cloud Map, or both\. To use this getting started guide, we recommend that you have three existing services that are registered with DNS\. You can create a service mesh and its resources even if the services don't exist, but you can't use the mesh until you have deployed actual services\.

If you don't already have services running, you can:
+ [Create an Amazon ECS service with service discovery](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/create-service-discovery.html)\.
+ [Setup an Amazon EKS cluster](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html) and deploy a [ cluster](https://docs.aws.amazon.com/eks/latest/userguide/eks-guestbook.html) and deploy a [sample application](https://docs.aws.amazon.com/eks/latest/userguide/eks-guestbook.html) to it\.
+ [Launch Amazon EC2 instances](https://docs.aws.amazon.com/app-mesh/latest/userguide/appmesh-getting-started.html#update-services) and deploy applications to them\.

The remaining steps assume that the actual services are named `serviceA`, `serviceB`, and `serviceBv2` and that all services are discoverable through a namespace named `apps.local`\. 

## Step 1: Create a Mesh and Virtual Service<a name="create-mesh-and-virtual-service"></a>

A service mesh is a logical boundary for network traffic between the services that reside within it\. For more information, see [Service Meshes](https://docs.aws.amazon.com/app-mesh/latest/userguide/meshes.html)\. A virtual service is an abstraction of an actual service\. For more information, see [Virtual Services](https://docs.aws.amazon.com/app-mesh/latest/userguide/virtual_services.html)\. 

Create the following resources:
+ A mesh named `apps`, since all of the services in the scenario are registered to the `apps.local` namespace\.
+ A virtual service named `serviceb.apps.local`, since the virtual service represents a service that is discoverable with that name, and you don't want to change your code to reference another name\. A virtual service named `servicea.apps.local` is added in a later step\.

You can use the AWS Management Console or the AWS CLI version 1\.18\.16 or higher to complete the following steps\. If using the AWS CLI, use the `aws --version` command to check your installed AWS CLI version\. If you don't have version 1\.18\.16 or higher installed, you must [install or update the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html)\. Select the tab for the tool that you want to use\.

------
#### [ AWS Management Console ]

1. Open the App Mesh console first\-run wizard at [https://console\.aws\.amazon\.com/appmesh/get\-started](https://console.aws.amazon.com/appmesh/get-started)\.

1. For **Mesh name**, enter **apps**\.

1. For **Virtual service name**, enter **serviceb\.apps\.local**\.

1. To continue, choose **Next**\.

------
#### [ AWS CLI ]

1. Create a mesh with the `[create\-mesh](https://docs.aws.amazon.com/cli/latest/reference/appmesh/create-mesh.html)` command\.

   ```
   aws appmesh create-mesh --mesh-name apps
   ```

1. Create a virtual service with the `[create\-virtual\-service](https://docs.aws.amazon.com/cli/latest/reference/appmesh/create-virtual-service.html)` command\.

   ```
   aws appmesh create-virtual-service --mesh-name apps --virtual-service-name serviceb.apps.local --spec {}
   ```

------

## Step 2: Create a Virtual Node<a name="create-virtual-node"></a>

A virtual node acts as a logical pointer to an actual service\. For more information, see [Virtual Nodes](https://docs.aws.amazon.com/app-mesh/latest/userguide/virtual_nodes.html)\. 

Create a virtual node named `serviceB`, since one of the virtual nodes represents the actual service named `serviceB`\. The actual service that the virtual node represents is discoverable through `DNS` with a hostname of `serviceb.apps.local`\. Alternately, you can discover actual services using AWS Cloud Map\. The virtual node will listen for traffic using the HTTP/2 protocol on port 80\. Other protocols are also supported, as are health checks\. You will create virtual nodes for `serviceA` and `serviceBv2` in a later step\.

------
#### [ AWS Management Console ]

1. For **Virtual node name**, enter **serviceB**\. 

1. For **Service discovery method**, choose `DNS` and enter **serviceb\.apps\.local** for **DNS hostname**\.

1. Under **Listener**, enter **80** for **Port** and choose `http2` for **Protocol**\.

1. To continue, choose **Next**\.

------
#### [ AWS CLI ]

1. Create a file named `create-virtual-node-serviceb.json` with the following contents:

   ```
   {
       "meshName": "apps",
       "spec": {
           "listeners": [
               {
                   "portMapping": {
                       "port": 80,
                       "protocol": "http2"
                   }
               }
           ],
           "serviceDiscovery": {
               "dns": {
                   "hostname": "serviceB.apps.local"
               }
           }
       },
       "virtualNodeName": "serviceB"
   }
   ```

1. Create the virtual node with the [create\-virtual\-node](https://docs.aws.amazon.com/cli/latest/reference/appmesh/create-virtual-node.html) command using the JSON file as input\.

   ```
   aws appmesh create-virtual-node --cli-input-json file://create-virtual-node-serviceb.json
   ```

------

## Step 3: Create a Virtual Router and Route<a name="create-virtual-router-and-route"></a>

Virtual routers route traffic for one or more virtual services within your mesh\. For more information, see [Virtual Routers](https://docs.aws.amazon.com/app-mesh/latest/userguide/virtual_routers.html) and [Routes](https://docs.aws.amazon.com/app-mesh/latest/userguide/routes.html)\.

Create the following resources:
+ A virtual router named `serviceB`, since the `serviceB.apps.local` virtual service doesn't initiate outbound communication with any other service\. Remember that the virtual service that you created previously is an abstraction of your actual `serviceb.apps.local` service\. The virtual service sends traffic to the virtual router\. The virtual router will listen for traffic using the HTTP/2 protocol on port 80\. Other protocols are also supported\. 
+ A route named `serviceB`\. It will route 100 percent of its traffic to the `serviceB` virtual node\. You'll change the weight in a later step once you've added the `serviceBv2` virtual node\. Though not covered in this guide, you can add additional filter criteria for the route and add a retry policy to cause the Envoy proxy to make multiple attempts to send traffic to a virtual node when it experiences a communication problem\.

------
#### [ AWS Management Console ]

1. For **Virtual router name,** enter **serviceB**\.

1. Under **Listener**, specify **80** for **Port** and choose `http2` for **Protocol**\.

1. For **Route name**, enter **serviceB**\. 

1. For **Route type**, choose `http2`\.

1. For** Virtual node name**, select `serviceB` and enter **100** for **Weight**\.

1. To continue, choose **Next**\.

------
#### [ AWS CLI ]

1. Create a virtual router\.

   1. Create a file named `create-virtual-router.json` with the following contents:

      ```
      {
          "meshName": "apps",
          "spec": {
              "listeners": [
                  {
                      "portMapping": {
                          "port": 80,
                          "protocol": "http2"
                      }
                  }
              ]
          },
          "virtualRouterName": "serviceB"
      }
      ```

   1. Create the virtual router with the [create\-virtual\-router](https://docs.aws.amazon.com/cli/latest/reference/appmesh/create-virtual-router.html) command using the JSON file as input\.

      ```
      aws appmesh create-virtual-router --cli-input-json file://create-virtual-router.json
      ```

1. Create a route\.

   1. Create a file named `create-route.json` with the following contents:

      ```
      {
          "meshName" : "apps",
          "routeName" : "serviceB",
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
                      "prefix" : "/"
                  }
              }
          },
          "virtualRouterName" : "serviceB"
      }
      ```

   1. Create the route with the [create\-route](https://docs.aws.amazon.com/cli/latest/reference/appmesh/create-route.html) command using the JSON file as input\.

      ```
      aws appmesh create-route --cli-input-json file://create-route.json
      ```

------

## Step 4: Review and Create<a name="review-create"></a>

Review the settings against the previous instructions\.

------
#### [ AWS Management Console ]

Choose **Edit** if you need to make any changes in any section\. Once you're satisfied with the settings, choose **Create mesh service**\.

------
#### [ AWS CLI ]

Review the settings of the mesh you created with the [describe\-mesh](https://docs.aws.amazon.com/cli/latest/reference/appmesh/describe-mesh.html) command\.

```
aws appmesh describe-mesh --mesh-name apps
```

Review the settings of the virtual service that you created with the [describe\-virtual\-service](https://docs.aws.amazon.com/cli/latest/reference/appmesh/describe-virtual-service.html) command\.

```
aws appmesh describe-virtual-service --mesh-name apps --virtual-service-name serviceb.apps.local
```

Review the settings of the virtual node that you created with the [describe\-virtual\-node](https://docs.aws.amazon.com/cli/latest/reference/appmesh/describe-virtual-node.html) command\.

```
aws appmesh describe-virtual-node --mesh-name apps --virtual-node-name serviceB
```

Review the settings of the virtual router that you created with the [describe\-virtual\-router](https://docs.aws.amazon.com/cli/latest/reference/appmesh/describe-virtual-router.html) command\.

```
aws appmesh describe-virtual-router --mesh-name apps --virtual-router-name serviceB
```

Review the settings of the route that you created with the [describe\-route](https://docs.aws.amazon.com/cli/latest/reference/appmesh/describe-route.html) command\.

```
aws appmesh describe-route --mesh-name apps \
    --virtual-router-name serviceB  --route-name serviceB
```

------

## Step 5: Create Additional Resources<a name="create-additional-resources"></a>

To complete the scenario, you need to:
+ Create one virtual node named `serviceBv2` and another named `serviceA`\. Both virtual nodes listen for requests over HTTP/2 port 80\. For the `serviceA` virtual node, configure a backend of `serviceb.apps.local`, since all outbound traffic from the `serviceA` virtual node is sent to the virtual service named `serviceb.apps.local`\. Though not covered in this guide, you can also specify a file path to write access logs to for a virtual node\.
+ Create one additional virtual service named `servicea.apps.local`, which will send all traffic directly to the `serviceA` virtual node\.
+ Update the `serviceB` route that you created in a previous step to send 75 percent of its traffic to the `serviceB` virtual node and 25 percent of its traffic to the `serviceBv2` virtual node\. Over time, you can continue to modify the weights until `serviceBv2` receives 100 percent of the traffic\. Once all traffic is sent to `serviceBv2`, you can deprecate the `serviceB` virtual node and actual service\. As you change weights, your code doesn't require any modification, because the `serviceb.apps.local` virtual and actual service names don't change\. Recall that the `serviceb.apps.local` virtual service sends traffic to the virtual router, which routes the traffic to the virtual nodes\. The service discovery names for the virtual nodes can be changed at any time\.

------
#### [ AWS Management Console ]

1. In the left navigation pane, select **Meshes**\.

1. Select the `apps` mesh that you created in a previous step\.

1. In the left navigation pane, select **Virtual nodes**\.

1. Choose **Create virtual node**\.

1. For **Virtual node name**, enter **serviceBv2**, for **Service discovery method**, choose `DNS`, and for **DNS hostname**, enter **servicebv2\.apps\.local**\.

1. For **Listener**, enter **80** for **Port** and select `http2` for **Protocol**\.

1. Choose **Create virtual node**\.

1. Choose **Create virtual node** again, and enter **serviceA** for the **Virtual node name**, for **Service discovery method**, choose `DNS`, and for **DNS hostname**, enter **servicea\.apps\.local**\.

1. Expand **Additional configuration**\.

1. Select **Add backend**\. Enter **serviceb\.apps\.local**\.

1. Enter **80** for **Port**, choose `http2` for **Protocol**, and then choose **Create virtual node**\.

1. In the left navigation pane, select** Virtual routers** and then select the `serviceB` virtual router from the list\.

1. Under **Routes**, select the route named `ServiceB` that you created in a previous step, and choose **Edit**\.

1. Under **Virtual node name**, change the value of **Weight** for `serviceB` to **75**\.

1. Choose **Add target**, choose `serviceBv2` from the drop\-down list, and set the value of **Weight** to **25**\.

1. Choose **Save**\.

1. In the left navigation pane, select** Virtual services** and then choose **Create virtual service**\.

1. Enter **servicea\.apps\.local** for **Virtual service name**, select `Virtual node` for **Provider**, select `serviceA` for **Virtual node**, and then choose **Create virtual service\.**

------
#### [ AWS CLI ]

1. Create the `serviceBv2` virtual node\.

   1. Create a file named `create-virtual-node-servicebv2.json` with the following contents:

      ```
      {
          "meshName": "apps",
          "spec": {
              "listeners": [
                  {
                      "portMapping": {
                          "port": 80,
                          "protocol": "http2"
                      }
                  }
              ],
              "serviceDiscovery": {
                  "dns": {
                      "hostname": "serviceBv2.apps.local"
                  }
              }
          },
          "virtualNodeName": "serviceBv2"
      }
      ```

   1. Create the virtual node\.

      ```
      aws appmesh create-virtual-node --cli-input-json file://create-virtual-node-servicebv2.json
      ```

1. Create the `serviceA` virtual node\.

   1. Create a file named `create-virtual-node-servicea.json` with the following contents:

      ```
      {
         "meshName" : "apps",
         "spec" : {
            "backends" : [
               {
                  "virtualService" : {
                     "virtualServiceName" : "serviceb.apps.local"
                  }
               }
            ],
            "listeners" : [
               {
                  "portMapping" : {
                     "port" : 80,
                     "protocol" : "http2"
                  }
               }
            ],
            "serviceDiscovery" : {
               "dns" : {
                  "hostname" : "servicea.apps.local"
               }
            }
         },
         "virtualNodeName" : "serviceA"
      }
      ```

   1. Create the virtual node\.

      ```
      aws appmesh create-virtual-node --cli-input-json file://create-virtual-node-servicea.json
      ```

1. Update the `serviceb.apps.local` virtual service that you created in a previous step to send its traffic to the `serviceB` virtual router\. When the virtual service was originally created, it didn't send traffic anywhere, since the `serviceB` virtual router hadn't been created yet\.

   1. Create a file named `update-virtual-service.json` with the following contents:

      ```
      {
         "meshName" : "apps",
         "spec" : {
            "provider" : {
               "virtualRouter" : {
                  "virtualRouterName" : "serviceB"
               }
            }
         },
         "virtualServiceName" : "serviceb.apps.local"
      }
      ```

   1. Update the virtual service with the [update\-virtual\-service](https://docs.aws.amazon.com/cli/latest/reference/appmesh/update-virtual-service.html) command\.

      ```
      aws appmesh update-virtual-service --cli-input-json file://update-virtual-service.json
      ```

1. Update the `serviceB` route that you created in a previous step\.

   1. Create a file named `update-route.json` with the following contents:

      ```
      {
         "meshName" : "apps",
         "routeName" : "serviceB",
         "spec" : {
            "http2Route" : {
               "action" : {
                  "weightedTargets" : [
                     {
                        "virtualNode" : "serviceB",
                        "weight" : 75
                     },
                     {
                        "virtualNode" : "serviceBv2",
                        "weight" : 25
                     }
                  ]
               },
               "match" : {
                  "prefix" : "/"
               }
            }
         },
         "virtualRouterName" : "serviceB"
      }
      ```

   1. Update the route with the [update\-route](https://docs.aws.amazon.com/cli/latest/reference/appmesh/update-route.html) command\.

      ```
      aws appmesh update-route --cli-input-json file://update-route.json
      ```

1. Create the `serviceA` virtual service\.

   1. Create a file named `create-virtual-servicea.json` with the following contents:

      ```
      {
         "meshName" : "apps",
         "spec" : {
            "provider" : {
               "virtualNode" : {
                  "virtualNodeName" : "serviceA"
               }
            }
         },
         "virtualServiceName" : "servicea.apps.local"
      }
      ```

   1. Create the virtual service\.

      ```
      aws appmesh create-virtual-service --cli-input-json file://create-virtual-servicea.json
      ```

------

**Mesh summary**  
Before you created the service mesh, you had three actual services named `servicea.apps.local`, `serviceb.apps.local`, and `servicebv2.apps.local`\. In addition to the actual services, you now have a service mesh that contains the following resources that represent the actual services:
+ Two virtual services\. The proxy sends all traffic from the `servicea.apps.local` virtual service to the `serviceb.apps.local` virtual service through a virtual router\. 
+ Three virtual nodes named `serviceA`, `serviceB`, and `serviceBv2`\. The Envoy proxy uses the service discovery information configured for the virtual nodes to look up the IP addresses of the actual services\. 
+ One virtual router with one route that instructs the Envoy proxy to route 75 percent of inbound traffic to the `serviceB` virtual node and 25 percent of the traffic to the `serviceBv2` virtual node\. 

## Step 6: Update Services<a name="update-services"></a>

After creating your mesh, you need to complete the following tasks:
+ Authorize the Envoy proxy that you deploy with each service to read the configuration of one or more virtual nodes\. For more information about how to authorize the proxy, see [Proxy authorization](https://docs.aws.amazon.com/app-mesh/latest/userguide/proxy-authorization.html)\.
+ Update each of your existing services to use the Envoy proxy\. To update your existing service that is running on Amazon ECS, see [Getting Started with App Mesh and Amazon ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/appmesh-getting-started.html#update-services)\. To update your existing service that is running on Kubernetes, see [Getting Started with App Mesh and Kubernetes](https://docs.aws.amazon.com/eks/latest/userguide/appmesh-getting-started.html#update-services)\. To update your existing service that is running on Amazon EC2, complete the steps that follow\.

**To configure an Amazon EC2 instance as a virtual node member**

1. Launch an Amazon EC2 instance with an IAM role that allows read permissions from Amazon ECR\. This is so that the instance can pull the App Mesh Envoy container image\. For more information, see [Amazon ECR Managed Policies](https://docs.aws.amazon.com/AmazonECR/latest/userguide/ecr_managed_policies.html)\.

1. Connect to your instance via SSH\.

1. Install Docker and the AWS CLI on your instance according to your operating system documentation\.

1. Authenticate to the Envoy Amazon ECR repository in the region that you want your Docker client to pull the image from:
   + All regions except `me-south-1`\. You can replace *us\-west\-2* with any [supported Region](https://docs.aws.amazon.com/general/latest/gr/appmesh.html) except `me-south-1`\.

     ```
     $(aws ecr get-login --no-include-email --region us-west-2 --registry-ids 840364872350)
     ```
   + `me-south-1` Region

     ```
     $(aws ecr get-login --no-include-email --region me-south-1 --registry-ids 772975370895)
     ```

1. Run one of the following commands to start the App Mesh Envoy container on your instance, depending on which region you want to pull the image from\. The *apps* and *serviceB* values are the mesh and virtual node names defined in the scenario\. To complete the scenario, you also need to complete these steps for the Amazon EC2 instances that host the services represented by the `serviceBv2` and `serviceA` virtual nodes\. For your own application, replace these values with your own\.
   + All regions except `me-south-1`\. You can replace *us\-west\-2* with any [supported Region](https://docs.aws.amazon.com/general/latest/gr/appmesh.html) except `me-south-1` Region\.

     ```
     sudo docker run --detach --env APPMESH_VIRTUAL_NODE_NAME=mesh/apps/virtualNode/serviceB  \
     -u 1337 --network host 840364872350.dkr.ecr.us-west-2.amazonaws.com/aws-appmesh-envoy:v1.12.2.1-prod
     ```
   + `me-south-1` Region

     ```
     sudo docker run --detach --env APPMESH_VIRTUAL_NODE_NAME=mesh/apps/virtualNode/serviceB  \
     -u 1337 --network host 772975370895.dkr.ecr.me-south-1.amazonaws.com/aws-appmesh-envoy:v1.12.2.1-prod
     ```

1. Select `more` below and run the script on your instance to configure the networking policies\. Replace the `APPMESH_APP_PORTS` value with the ports that your application code uses for ingress\.

   ```
   #!/bin/bash -e
   
   #
   # Start of configurable options
   #
   
   
   #APPMESH_START_ENABLED="0"
   APPMESH_IGNORE_UID="1337"
   APPMESH_APP_PORTS="8000"
   APPMESH_ENVOY_EGRESS_PORT="15001"
   APPMESH_ENVOY_INGRESS_PORT="15000"
   APPMESH_EGRESS_IGNORED_IP="169.254.169.254,169.254.170.2" 
   
   # Enable routing on the application start.
   [ -z "$APPMESH_START_ENABLED" ] && APPMESH_START_ENABLED="0"
   
   # Egress traffic from the processess owned by the following UID/GID will be ignored.
   if [ -z "$APPMESH_IGNORE_UID" ] && [ -z "$APPMESH_IGNORE_GID" ]; then
       echo "Variables APPMESH_IGNORE_UID and/or APPMESH_IGNORE_GID must be set."
       echo "Envoy must run under those IDs to be able to properly route it's egress traffic."
       exit 1
   fi
   
   # Port numbers Application and Envoy are listening on.
   if [ -z "$APPMESH_ENVOY_INGRESS_PORT" ] || [ -z "$APPMESH_ENVOY_EGRESS_PORT" ] || [ -z "$APPMESH_APP_PORTS" ]; then
       echo "All of APPMESH_ENVOY_INGRESS_PORT, APPMESH_ENVOY_EGRESS_PORT and APPMESH_APP_PORTS variables must be set."
       echo "If any one of them is not set we will not be able to route either ingress, egress, or both directions."
       exit 1
   fi
   
   # Comma separated list of ports for which egress traffic will be ignored, we always refuse to route SSH traffic.
   if [ -z "$APPMESH_EGRESS_IGNORED_PORTS" ]; then
       APPMESH_EGRESS_IGNORED_PORTS="22"
   else
       APPMESH_EGRESS_IGNORED_PORTS="$APPMESH_EGRESS_IGNORED_PORTS,22"
   fi
   
   #
   # End of configurable options
   #
   
   APPMESH_LOCAL_ROUTE_TABLE_ID="100"
   APPMESH_PACKET_MARK="0x1e7700ce"
   
   function initialize() {
       echo "=== Initializing ==="
       iptables -t mangle -N APPMESH_INGRESS
       iptables -t nat -N APPMESH_INGRESS
       iptables -t nat -N APPMESH_EGRESS
   
       ip rule add fwmark "$APPMESH_PACKET_MARK" lookup $APPMESH_LOCAL_ROUTE_TABLE_ID
       ip route add local default dev lo table $APPMESH_LOCAL_ROUTE_TABLE_ID
   }
   
   function enable_egress_routing() {
       # Stuff to ignore
       [ ! -z "$APPMESH_IGNORE_UID" ] && \
           iptables -t nat -A APPMESH_EGRESS \
           -m owner --uid-owner $APPMESH_IGNORE_UID \
           -j RETURN
   
       [ ! -z "$APPMESH_IGNORE_GID" ] && \
           iptables -t nat -A APPMESH_EGRESS \
           -m owner --gid-owner $APPMESH_IGNORE_GID \
           -j RETURN
   
       [ ! -z "$APPMESH_EGRESS_IGNORED_PORTS" ] && \
           iptables -t nat -A APPMESH_EGRESS \
           -p tcp \
           -m multiport --dports "$APPMESH_EGRESS_IGNORED_PORTS" \
           -j RETURN
   
       [ ! -z "$APPMESH_EGRESS_IGNORED_IP" ] && \
           iptables -t nat -A APPMESH_EGRESS \
           -p tcp \
           -d "$APPMESH_EGRESS_IGNORED_IP" \
           -j RETURN
   
       # Redirect everything that is not ignored
       iptables -t nat -A APPMESH_EGRESS \
           -p tcp \
           -j REDIRECT --to $APPMESH_ENVOY_EGRESS_PORT
   
       # Apply APPMESH_EGRESS chain to non local traffic
       iptables -t nat -A OUTPUT \
           -p tcp \
           -m addrtype ! --dst-type LOCAL \
           -j APPMESH_EGRESS
   }
   
   function enable_ingress_redirect_routing() {
       # Route everything arriving at the application port to Envoy
       iptables -t nat -A APPMESH_INGRESS \
           -p tcp \
           -m multiport --dports "$APPMESH_APP_PORTS" \
           -j REDIRECT --to-port "$APPMESH_ENVOY_INGRESS_PORT"
   
       # Apply AppMesh ingress chain to everything non-local
       iptables -t nat -A PREROUTING \
           -p tcp \
           -m addrtype ! --src-type LOCAL \
           -j APPMESH_INGRESS
   }
   
   function enable_routing() {
       echo "=== Enabling routing ==="
       enable_egress_routing
       enable_ingress_redirect_routing
   }
   
   function disable_routing() {
       echo "=== Disabling routing ==="
       iptables -F
       iptables -F -t nat
       iptables -F -t mangle
   }
   
   function dump_status() {
       echo "=== Routing rules ==="
       ip rule
       echo "=== AppMesh routing table ==="
       ip route list table $APPMESH_LOCAL_ROUTE_TABLE_ID
       echo "=== iptables FORWARD table ==="
       iptables -L -v -n
       echo "=== iptables NAT table ==="
       iptables -t nat -L -v -n
       echo "=== iptables MANGLE table ==="
       iptables -t mangle -L -v -n
   }
   
   function main_loop() {
       echo "=== Entering main loop ==="
       while read -p '> ' cmd; do
           case "$cmd" in
               "quit")
                   break
                   ;;
               "status")
                   dump_status
                   ;;
               "enable")
                   enable_routing
                   ;;
               "disable")
                   disable_routing
                   ;;
               *)
                   echo "Available commands: quit, status, enable, disable"
                   ;;
           esac
       done
   }
   
   function print_config() {
       echo "=== Input configuration ==="
       env | grep APPMESH_ || true
   }
   
   print_config
   
   initialize
   
   if [ "$APPMESH_START_ENABLED" == "1" ]; then
       enable_routing
   fi
   
   main_loop
   ```

1. Start your virtual node application code\.
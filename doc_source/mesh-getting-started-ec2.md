# Getting Started with AWS App Mesh and Amazon EC2<a name="mesh-getting-started-ec2"></a>

This topic helps you to use AWS App Mesh with an existing microservice application running on Amazon EC2 instances\.

## Prerequisites<a name="mesh-gs-ec2-prerequisites"></a>

App Mesh supports microservice applications that use service discovery naming for their components\. To use this getting started guide, you must have a microservice application running on Amazon EC2 instances\.

This guide also assumes that you have completed the [Getting Started with AWS App Mesh](getting_started.md) guide, and that you have the following App Mesh resources:
+ A service mesh
+ Virtual nodes for each microservice in your application
+ Virtual routers and routes for each microservice in your application \(except for virtual services that are provided by a virtual node directly\)
+ Virtual services for each microservice in your application

## Configure Your Amazon EC2 Instances<a name="mesh-gs-ec2-configure-instances"></a>

AWS App Mesh is a service mesh based on the [Envoy](https://www.envoyproxy.io/) proxy\. After you create your service mesh, virtual nodes, virtual routers, and routes, you must configure your Amazon EC2 instances to be compatible with App Mesh\. The following procedure describes this process\.

**To configure an Amazon EC2 instance as a virtual node member**

1. Launch an Amazon EC2 instance with an IAM role that allows read permissions from Amazon ECR\. This is so that the instance can pull the App Mesh Envoy container image\. For more information, see [Amazon ECR Managed Policies](https://docs.aws.amazon.com/AmazonECR/latest/userguide/ecr_managed_policies.html)\.

1. Connect to your instance via SSH\.

1. Install Docker and the AWS CLI on your instance according to your operating system documentation\.

1. Authenticate to the Envoy Amazon ECR repository so that your Docker client can pull the container image\.

   ```
   $(aws ecr get-login --no-include-email --region us-west-2 --registry-ids 111345817488)
   ```

1. Run the following command to start the App Mesh Envoy container on your instance\. Substitute your mesh name and virtual node name\.

   ```
   sudo docker run --detach --env APPMESH_VIRTUAL_NODE_NAME=mesh/meshName/virtualNode/virtualNodeName  \
   -u 1337 --network host 111345817488.dkr.ecr.us-west-2.amazonaws.com/aws-appmesh-envoy:v1.9.0.0-prod
   ```

1. Run the following script on your instance to configure the networking policies\. Replace the `APPMESH_APP_PORTS` value with the ports that your application code uses for ingress\.

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
       APPMESH_EGRESS_IGNORED_PORTS=",22"
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
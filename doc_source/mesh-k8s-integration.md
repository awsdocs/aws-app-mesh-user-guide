# Tutorial: Configure App Mesh integration with Kubernetes<a name="mesh-k8s-integration"></a>

When you integrate AWS App Mesh with Kubernetes, you manage App Mesh resources, such as meshes, virtual services, virtual nodes and virtual routers, through Kubernetes\. You also automatically add the App Mesh sidecar container images to Kubernetes pod specifications\. This tutorial guides you through the installation of the following open source components that enable this integration:
+ **App Mesh manager for Kubernetes** – The manager is accompanied by the deployment of four Kubernetes custom resource definitions: `mesh`, `virtual service`, `virtual node`, and `virtual router` \. The controller watches for creation, modification, and deletion of the custom resources and makes changes to the corresponding App Mesh `[mesh](https://docs.aws.amazon.com/app-mesh/latest/userguide/meshes.html)`, `[virtual service](https://docs.aws.amazon.com/app-mesh/latest/userguide/virtual_services.html)`, `[virtual router](https://docs.aws.amazon.com/app-mesh/latest/userguide/virtual_routers.html)` (including `[route](https://docs.aws.amazon.com/app-mesh/latest/userguide/routes.html)`\), and `[virtual node](https://docs.aws.amazon.com/app-mesh/latest/userguide/virtual_nodes.html)` resources through the App Mesh API\. To learn more or contribute to the controller, see the [GitHub project](https://github.com/aws/aws-app-mesh-controller-for-k8s)\.
+  **App Mesh sidecar injector for Kubernetes** – The injector installs as a webhook and injects the following containers into Kubernetes pods that are running in specific, labeled namespaces\. The sidecar injector runs inside the App Mesh manager.
    + **App Mesh Envoy proxy** –Envoy uses the configuration defined in the App Mesh control plane to determine where to send your application traffic\.\.
    + **App Mesh proxy route manager **– The route manager sets up a pod’s network namespace with `iptables` rules that route ingress and egress traffic through Envoy\. This runs as a Kubernetes init container inside a pod 


|  | 
| --- |
| The features discussed in this topic are available as an open\-source beta\. This means that these features are well tested\. Support for the features will not be dropped, though details may change\. If the schema or schematics of a feature changes, instructions for migrating to the next version will be provided\. This migration may require deleting, editing, and re\-creating Kubernetes API objects\. | 

## Prerequisites<a name="mesh-k8s-integration-prerequisites"></a>
+ An existing understanding of Kubernetes concepts\. For more information, see [What is Kubernetes\.](https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/)
+ An existing Kubernetes cluster running version 1\.13 or later\. If you don't have an existing cluster, you can deploy one using the [Getting Started with Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html) guide\. 
+ The AWS CLI version 1\.18\.16 or later installed\. To install or upgrade the, see [Installing the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html)\. 
+ A `kubectl` client that is configured to communicate with your Kubernetes cluster\. If you're using Amazon Elastic Kubernetes Service, you can use the instructions for installing `[kubectl](https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html)` and configuring a `[kubeconfig](https://docs.aws.amazon.com/eks/latest/userguide/create-kubeconfig.html)` file\.
+ Helm version 3\.0 or later installed\. If you don't have Helm installed, you can install it by completing the instructions in [Using Helm with Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/helm.html)\.

## Step 1: Install the integration components<a name="install-controller"></a>

Install the integration components one time to each cluster that hosts pods that you want to use with App Mesh

**To install the integration components**

1. Add the `eks-charts` repository to Helm\.

   ```
   helm repo add eks https://aws.github.io/eks-charts
   ```

1. Install the App Mesh Kubernetes custom resource definitions \(CRD\)\.

   ```
   kubectl apply -k https://github.com/aws/eks-charts/stable/appmesh-manager/crds?ref=master
   ```

1. Create a Kubernetes namespace for the controller\.

   ```
   kubectl create ns appmesh-system
   ```

1. Set the following variables\. Replace `cluster-name` and `region-code` with the values for your existing cluster\.

   ```
   export CLUSTER_NAME=cluster-name
   export AWS_REGION=region-code
   ```

1. Create an OpenID Connect \(OIDC\) identity provider for your cluster\. If you don't have `eksctl` installed, you can install it with the instructions in [Installing or Upgrading `eksctl`](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html#installing-eksctl)\. If you'd prefer to create the provider using the console, see [Enabling IAM Roles for Service Accounts on your Cluster](https://docs.aws.amazon.com/eks/latest/userguide/enable-iam-roles-for-service-accounts.html)\.

   ```
   eksctl utils associate-iam-oidc-provider \
       --region=$AWS_REGION \
       --cluster $CLUSTER_NAME \
       --approve
   ```

1. Create an IAM role, attach the [AWSAppMeshFullAccess](https://console.aws.amazon.com/iam/home?#policies/arn:aws:iam::aws:policy/AWSAppMeshFullAccess$jsonEditor) and [AWSCloudMapFullAccess](https://console.aws.amazon.com/iam/home?#policies/arn:aws:iam::aws:policy/AWSCloudMapFullAccess$jsonEditor) AWS managed policies to it, and bind it to the `appmesh-manager` Kubernetes service account\. The role enables the controller to add, remove, and change App Mesh resources\.
**Note**  
The command creates an AWS IAM role with an auto\-generated name\. You are not able to specify the IAM role name that is created\.

   ```
   eksctl create iamserviceaccount \
       --cluster $CLUSTER_NAME \
       --namespace appmesh-system \
       --name appmesh-manager \
       --attach-policy-arn  arn:aws:iam::aws:policy/AWSCloudMapFullAccess,arn:aws:iam::aws:policy/AWSAppMeshFullAccess \
       --override-existing-serviceaccounts \
       --approve
   ```

   If you prefer to create the service account using the AWS Management Console or AWS CLI, see [Creating an IAM Role and Policy for your Service Account](https://docs.aws.amazon.com/eks/latest/userguide/create-service-account-iam-policy-and-role.html#create-service-account-iam-role)\. If you use the AWS Management Console or AWS CLI to create the account, you also need to map the role to a Kubernetes service account\. For more information, see [Specifying an IAM Role for your Service Account](https://docs.aws.amazon.com/eks/latest/userguide/specify-service-account-role.html)\. 

1. Deploy the App Mesh controller manager\. For a list of all configuration options, see [Configuration](https://github.com/aws/eks-charts/blob/master/stable/appmesh-manager/README.md#configuration) on GitHub\.

   ```
   helm upgrade -i appmesh-manager eks/appmesh-manager \
    --namespace appmesh-system \
    --set region=$AWS_REGION \
    --set serviceAccount.create=false \
    --set serviceAccount.name=appmesh-manager
   ```

1. Confirm that the controller version is `v1.0.0` or later\. You can view the [change log](https://github.com/aws/aws-app-mesh-controller-for-k8s/blob/master/CHANGELOG.md) on GitHub\.

   ```
   kubectl get deployment -n appmesh-system appmesh-manager -o json  | jq -r ".spec.template.spec.containers[].image" | cut -f2 -d ':'
   ```

**Note**  
If you view the log for the running container, you may see a line that includes the following text, which can be safely ignored\.  

   ```
   Neither --kubeconfig nor --master was specified. Using the inClusterConfig. This might not work.
   ```

## Step 2: Deploy App Mesh resources<a name="configure-app-mesh"></a>

When you deploy an application in Kubernetes, you also create the Kubernetes custom resources so that the controller can create the corresponding App Mesh resources\.

**To deploy App Mesh resources**

1. Create a Kubernetes namespace to deploy App Mesh resources to\. Write the
   following yaml to a file named `namespace.yaml`

   ```
   apiVersion: v1
   kind: Namespace
   metadata:
     name: my-app-1
     labels:
       mesh: my-mesh
       appmesh.k8s.aws/sidecarInjectorWebhook: enabled
   ```

   ```
   kubectl apply -f namespace.yaml
   ```

1. Create a Mesh
   1. Create a file named `mesh.yaml` with the following contents\. The file will be used to create a Mesh resource named `my-mesh` referncing to *`my-app-1`* namespace\. A service mesh is a logical boundary for network traffic between the services that reside within it. After you create your service mesh, you can create virtual services, virtual nodes, virtual routers, and routes to distribute traffic between the applications in your mesh.
      ```
      apiVersion: appmesh.k8s.aws/v1beta2
      kind: Mesh
      metadata:
        name: my-mesh
      spec:
        namespaceSelector:
          matchLabels:
            mesh: my-mesh
      ```

   1. Deploy the mesh\.
     ```
     kubectl apply -f mesh.yaml
     ```

1. Create an App Mesh virtual node\. 

   1. Create a file named `virtual-node.yaml` with the following contents\. The file will be used to create an App Mesh virtual node named `my-service-a` in the *`my-app-1`* namespace\. The virtual node represents a Kubernetes service that is created in a later step\.

      ```
      apiVersion: appmesh.k8s.aws/v1beta2
      kind: VirtualNode
      metadata:
        name: my-service-a
        namespace: my-app-1
      spec:
        listeners:
          - portMapping:
              port: 80
              protocol: http
        serviceDiscovery:
          dns:
            hostname: my-service-a.my-app-1.svc.cluster.local
      ```

      Virtual nodes have capabilities, such as end\-to\-end encryption and health checks, that aren't covered in this tutorial\. For more information, see [Virtual nodes](https://docs.aws.amazon.com/app-mesh/latest/userguide/virtual_nodes.html)\. To see all available settings for a virtual node that you can set in the preceding spec, run the following command\.

      ```
      aws appmesh create-virtual-node --generate-cli-skeleton yaml-input
      ```

   1. Deploy the virtual node\.

      ```
      kubectl apply -f virtual-node.yaml
      ```

   1. View all of the resources in the `my-app-1` namespace\.

      ```
      kubectl -n my-app-1 get all
      ```

      Output

      ```
      NAME                                            AGE
      virtualnode.appmesh.k8s.aws/my-service-a   16m
      ```

      This output shows the virtual node that exist in the *`my-app-1`* namespace\.

   1.  Confirm that the virtual node was created in App Mesh\.
**Note**  
Even though the name of the virtual node created in Kubernetes is `my-service-a`, the name of the virtual node created in App Mesh is `my-service-a_my-app-1`\. The controller appends the Kubernetes namespace name to the App Mesh virtual node name when it creates the App Mesh resource\. The namespace name is added because in Kubernetes you can create virtual nodes with the same name in different namespaces, but in App Mesh a virtual node name must be unique within a mesh\.

      ```
      aws appmesh describe-virtual-node --mesh-name my-mesh --virtual-node-name my-service-a_my-app-1
      ```

      Output

      ```
      {
          "virtualNode": {
              "meshName": "my-mesh",
              "metadata": {
                  "arn": "arn:aws:appmesh:us-west-2:999999999999:mesh/my-mesh/virtualNode/my-service-a_my-app-1",
                  "createdAt": 1590688032.748,
                  "lastUpdatedAt": 1590688032.748,
                  "meshOwner": "889966900099",
                  "resourceOwner": "889966900099",
                  "uid": "d5245b73-5719-470c-9a52-ef846ca761d3",
                  "version": 1
              },
              "spec": {
                  "backends": [],
                  "listeners": [
                      {
                          "portMapping": {
                              "port": 80,
                              "protocol": "http"
                          }
                      }
                  ],
                  "serviceDiscovery": {
                      "dns": {
                          "hostname": "my-service-a.my-app-1.svc.cluster.local"
                      }
                  }
              },
              "status": {
                  "status": "ACTIVE"
              },
              "virtualNodeName": "my-service-a_my-app-1"
          }
      }
      ```

      Note the value of `arn` in the preceding output\. You will use it in a later step\.

1. Create an App Mesh virtual router

   1. Create a file named `virtual-router.yaml` with the following contents\. The file will be used to create a virtual router to route traffic to the virtual node named `my-service-a` that was created in the previous step\. The value for `name` is the fully qualified domain name \(FQDN\) of the actual Kubernetes service that this virtual service abstracts\. The service is created in [Step 3: Create or update services](#integration-update-services)\. The controller will create the App Mesh virtual router, and route resources\. You can specify many more capabilities for your routes and use protocols other than `http`\. For more information, see [Virtual router](https://docs.aws.amazon.com/app-mesh/latest/userguide/virtual_routers.html), and [Route](https://docs.aws.amazon.com/app-mesh/latest/userguide/routes.html) \.

      ```
      apiVersion: appmesh.k8s.aws/v1beta2
      kind: VirtualRouter
      metadata:
        namespace: my-app-1
        name: my-service-a-virtual-router
      spec:
        listeners:
          - portMapping:
              port: 80
              protocol: http
        routes:
          - name: my-service-a-route
            httpRoute:
              match:
                prefix: /
              action:
                weightedTargets:
                  - virtualNodeRef:
                      name: my-service-a
                    weight: 1
      ```

      To see all available settings for a virtual router that you can set in the preceding spec, run any of the following command\.

      ```
      aws appmesh create-virtual-router --generate-cli-skeleton yaml-input
      ```

   1. Create the virtual router\.

      ```
      kubectl apply -f virtual-router.yaml
      ```

   1. View the virtual service resource\.

      ```
      kubectl describe virtualrouter my-service-a-virtual-router -n my-app-1
      ```

      Abbreviated output

      ```
      Name:         my-service-a-virtual-router
      Namespace:    my-app-1
      Labels:       <none>
      Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                      {"apiVersion":"appmesh.k8s.aws/v1beta2","kind":"VirtualRouter","metadata":{"annotations":{},"name":"my-service-a-virtual-router","namespac...
      API Version:  appmesh.k8s.aws/v1beta2
      Kind:         VirtualRouter
      Metadata:
        Creation Timestamp:  2020-05-28T17:46:12Z
        Finalizers:
          finalizers.appmesh.k8s.aws/aws-appmesh-resources
        Generation:        1
        Resource Version:  7608122
        Self Link:         /apis/appmesh.k8s.aws/v1beta2/namespaces/my-app-1/virtualrouters/my-service-a-virtual-router
        UID:               64cdc90f-6e3c-471e-b068-4e419c0d5924
      Spec:
        Aws Name:  my-service-a-virtual-router_my-app-1
        Listeners:
          Port Mapping:
            Port:      80
            Protocol:  http
        Mesh Ref:
          Name:  my-mesh
          UID:   1940d565-dacb-4a8a-9b8e-c1a00930fda1
        Routes:
          Http Route:
            Action:
              Weighted Targets:
                Virtual Node Ref:
                  Name:  my-service-a
                Weight:  1
            Match:
              Prefix:  /
          Name:        my-service-a-route
      Status:
        Conditions:
          Last Transition Time:  2020-05-28T17:47:12Z
          Status:                True
          Type:                  VirtualRouterActive
        Observed Generation:     1
        Route AR Ns:
          My - Service - A - Route:  arn:aws:appmesh:us-west-2:999999999999:mesh/my-mesh/virtualRouter/my-service-a-virtual-router_my-app-1/route/my-service-a-route
        Virtual Router ARN:          arn:aws:appmesh:us-west-2:999999999999:mesh/my-mesh/virtualRouter/my-service-a-virtual-router_my-app-1
      Events:                        <none>
      ```

   1. Confirm that the virtual router was created in your mesh\.

      ```
      aws appmesh describe-virtual-router --virtual-router-name my-service-a-virtual-router_my-app-1 --mesh-name=my-mesh
      ```

      Output

      ```
      {
          "virtualRouter": {
              "meshName": "my-mesh",
              "metadata": {
                  "arn": "arn:aws:appmesh:us-west-2:999999999999:mesh/my-mesh/virtualRouter/my-service-a-virtual-router_my-app-1",
                  "createdAt": 1590688032.79,
                  "lastUpdatedAt": 1590688032.79,
                  "meshOwner": "999999999999",
                  "resourceOwner": "999999999999",
                  "uid": "00dd50af-3fd7-41e1-85de-fb38c91ed4a2",
                  "version": 1
              },
              "spec": {
                  "listeners": [
                      {
                          "portMapping": {
                              "port": 80,
                              "protocol": "http"
                          }
                      }
                  ]
              },
              "status": {
                  "status": "ACTIVE"
              },
              "virtualRouterName": "my-service-a-virtual-router_my-app-1"
          }
      }
      ```

   1. Confirm that the route was created in your mesh\. The Kubernetes controller did not append the Kubernetes namespace name to the App Mesh route name when it created the route in App Mesh because route names are unique to a virtual router\.

      ```
      aws appmesh describe-route --route-name my-service-a-route --virtual-router-name my-service-a-virtual-router_my-app-1 --mesh-name my-mesh
      ```

      Output

      ```
      {
          "route": {
              "meshName": "my-mesh",
              "metadata": {
                  "arn": "arn:aws:appmesh:us-west-2:999999999999:mesh/my-mesh/virtualRouter/my-service-a-virtual-router_my-app-1/route/my-service-a-route",
                  "createdAt": 1590688032.817,
                  "lastUpdatedAt": 1590688032.817,
                  "meshOwner": "999999999999",
                  "resourceOwner": "999999999999",
                  "uid": "58ac82cb-c718-45f3-a299-e9432afb6326",
                  "version": 1
              },
              "routeName": "my-service-a-route",
              "spec": {
                  "httpRoute": {
                      "action": {
                          "weightedTargets": [
                              {
                                  "virtualNode": "my-service-a_my-app-1",
                                  "weight": 1
                              }
                          ]
                      },
                      "match": {
                          "prefix": "/"
                      }
                  }
              },
              "status": {
                  "status": "ACTIVE"
              },
              "virtualRouterName": "my-service-a-virtual-router_my-app-1"
          }
      }
      ```

1. Create an App Mesh virtual service 

   1. Create a file named `virtual-service.yaml` with the following contents\. The file will be used to create a virtual service that uses a virtual router provider to route traffic to the virtual node named `my-service-a` that was created in the previous step\. The value for `name` is the fully qualified domain name \(FQDN\) of the actual Kubernetes service that this virtual service abstracts\. The service is created in [Step 3: Create or update services](#integration-update-services)\. The controller will create the App Mesh virtual service, virtual router, and route resources\. You can specify many more capabilities for your routes and use protocols other than `http`\. For more information, see [Virtual services](https://docs.aws.amazon.com/app-mesh/latest/userguide/virtual_services.html), [Virtual router](https://docs.aws.amazon.com/app-mesh/latest/userguide/virtual_routers.html), and [Route](https://docs.aws.amazon.com/app-mesh/latest/userguide/routes.html) \. 

      ```
      apiVersion: appmesh.k8s.aws/v1beta2
      kind: VirtualService
      metadata:
        name: my-service-a
        namespace: my-app-1
      spec:
        awsName: my-service-a.my-app-1.svc.cluster.local
        provider:
          virtualRouter:
            virtualRouterRef:
              name: my-service-a-virtual-router
      ```

      To see all available settings for a virtual service that you can set in the preceding spec, run any of the following command\.

      ```
      aws appmesh create-virtual-service --generate-cli-skeleton yaml-input
      ```

   1. Create the virtual service\.

      ```
      kubectl apply -f virtual-service.yaml
      ```

   1. View the virtual service resource\.

      ```
      kubectl describe virtualservice my-service-a -n my-app-1
      ```

      Abbreviated output

      ```
      Name:         my-service-a
      Namespace:    my-app-1
      Labels:       <none>
      Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                      {"apiVersion":"appmesh.k8s.aws/v1beta2","kind":"VirtualService","metadata":{"annotations":{},"name":"my-service-a","namespace":"my-app-1"}...
      API Version:  appmesh.k8s.aws/v1beta2
      Kind:         VirtualService
      Metadata:
        Creation Timestamp:  2020-05-28T17:52:29Z
        Finalizers:
          finalizers.appmesh.k8s.aws/aws-appmesh-resources
        Generation:        1
        Resource Version:  7609335
        Self Link:         /apis/appmesh.k8s.aws/v1beta2/namespaces/my-app-1/virtualservices/my-service-a
        UID:               6dda2d91-3f93-478d-9084-8b9ab1e966ee
      Spec:
        Aws Name:  my-service-a.my-app-1.svc.cluster.local
        Mesh Ref:
          Name:  my-mesh
          UID:   1940d565-dacb-4a8a-9b8e-c1a00930fda1
        Provider:
          Virtual Router:
            Virtual Router Ref:
              Name:  my-service-a-virtual-router
      Status:
        Conditions:
          Last Transition Time:  2020-05-28T17:52:29Z
          Status:                True
          Type:                  VirtualServiceActive
        Observed Generation:     1
        Virtual Service ARN:     arn:aws:appmesh:us-west-2:999999999999:mesh/my-mesh/virtualService/my-service-a.my-app-1.svc.cluster.local
      Events:                    <none>
      ```

   1. Confirm that the virtual service was created in your mesh\. The Kubernetes controller did not append the Kubernetes namespace name to the App Mesh virtual service name when it created the virtual service in App Mesh because the virtual service's name is a unique FQDN\.

      ```
      aws appmesh describe-virtual-service --virtual-service-name my-service-a.my-app-1.svc.cluster.local --mesh-name=my-mesh
      ```

      Output

      ```
      {
          "virtualService": {
              "meshName": "my-mesh",
              "metadata": {
                  "arn": "arn:aws:appmesh:us-west-2:999999999999:mesh/my-mesh/virtualService/my-service-a.my-app-1.svc.cluster.local",
                  "createdAt": 1590688349.741,
                  "lastUpdatedAt": 1590688349.741,
                  "meshOwner": "999999999999",
                  "resourceOwner": "999999999999",
                  "uid": "962a27b4-a6f6-4a2b-b0a1-70967b4ade5f",
                  "version": 1
              },
              "spec": {
                  "provider": {
                      "virtualRouter": {
                          "virtualRouterName": "my-service-a-virtual-router_my-app-1"
                      }
                  }
              },
              "status": {
                  "status": "ACTIVE"
              },
              "virtualServiceName": "my-service-a.my-app-1.svc.cluster.local"
          }
      }
      ```

## Step 3: Create or update services<a name="integration-update-services"></a>

Any pods that you want to use with App Mesh must have the App Mesh sidecar containers added to them\. The injector automatically adds the sidecar containers to any pod deployed into a namespace that you specify\.

**To create or update services**

1. To enable sidecar injection for the namespace, label the namespace\.

   ```
   kubectl label namespace my-app-1 appmesh.k8s.aws/sidecarInjectorWebhook=enabled
   ```

1.  Enable proxy authorization\. We recommend that you enable each Kubernetes deployment to stream the configuration for its own App Mesh virtual node\.

   1. Create a file named `proxy-auth.json` with the following contents\. 

      ```
      {
          "Version": "2012-10-17",
          "Statement": [
              {
                  "Effect": "Allow",
                  "Action": "appmesh:StreamAggregatedResources",
                  "Resource": [
                      "arn:aws:appmesh:region-code:111122223333:mesh/my-mesh/virtualNode/my-service-a-my-app-1"
                  ]
              }
          ]
      }
      ```

   1. Create the policy\.

      ```
      aws iam create-policy --policy-name my-policy --policy-document file://proxy-auth.json
      ```

      Note the ARN of the policy in the output returned\. You'll use it in the next step\.

   1. Create an IAM role, attach the policy you created in the previous step to it, create a Kubernetes service account and bind the policy to the Kubernetes service account\. The role enables the controller to add, remove, and change App Mesh resources\.

      ```
      eksctl create iamserviceaccount \
          --cluster $CLUSTER_NAME \
          --namespace my-app-1 \
          --name my-service-a \
          --attach-policy-arn  arn:aws:iam::111122223333:policy/my-policy \
          --override-existing-serviceaccounts \
          --approve
      ```

      If you prefer to create the service account using the AWS Management Console or AWS CLI, see [Creating an IAM Role and Policy for your Service Account](https://docs.aws.amazon.com/eks/latest/userguide/create-service-account-iam-policy-and-role.html#create-service-account-iam-role)\. If you use the AWS Management Console or AWS CLI to create the account, you also need to map the role to a Kubernetes service account\. For more information, see [Specifying an IAM Role for your Service Account](https://docs.aws.amazon.com/eks/latest/userguide/specify-service-account-role.html)\. 

1. Create a Kubernetes service and deployment\. If you have an existing deployment that you want to use with App Mesh, you need to update its namespace to *`my-app-1`* so that the sidecar containers are automatically added to the pods and the pods are redeployed\.

   1. Create a file named `example-service.yaml` with the following contents\.

      ```
      apiVersion: v1
      kind: Service
      metadata:
        name: my-service-a
        namespace: my-app-1
        labels:
          app: nginx
      spec:
        selector:
          app: nginx
        ports:
          - protocol: TCP
            port: 80
            targetPort: 80
      ---
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: my-service-a
        namespace: my-app-1
        labels:
          app: nginx
      spec:
        replicas: 3
        selector:
          matchLabels:
            app: nginx
        template:
          metadata:
            labels:
              app: nginx
          spec:
            serviceAccountName: my-service-a
            containers:
            - name: nginx
              image: nginx:1.14.2
              ports:
              - containerPort: 80
      ```

      For sidecar injection to work, everypod must have a corresponding virtual node. You will need to add podSelector->matchLabels in your virtual node definition. See the example below:

      ```
      apiVersion: appmesh.k8s.aws/v1beta2
      kind: VirtualNode
      metadata:
        name: my-service-a
        namespace: my-app-1
      spec:
        podSelector:
          matchLabels:
            app: nginx
        listeners:
          - portMapping:
              port: 80
              protocol: http
        serviceDiscovery:
          dns:
            hostname: my-service-a.my-app-1.svc.cluster.local
      ```

   1. Deploy the service\.

      ```
      kubectl apply -f example-service.yaml
      ```

   1. View the service and deployment\.

      ```
      kubectl -n my-app-1 get pods
      ```

      Output

      ```
      NAME                            READY   STATUS    RESTARTS   AGE
      my-service-a-574b87c764-5gps5   2/2     Running   0          3m7s
      my-service-a-574b87c764-g8skb   2/2     Running   0          3m7s
      my-service-a-574b87c764-m5592   2/2     Running   0          3m7s
      ```

   1. View the details for one of the pods that was deployed\.

      ```
      kubectl -n my-app-1 describe pod my-service-a-574b87c764-5gps5
      ```

      Abbreviated output

      ```
      Name:           my-service-a-574b87c764-5gps5
      Namespace:      my-app-1
      Priority:       0
      Node:           ip-192-168-11-213.us-west-2.compute.internal/192.168.11.213
      Start Time:     Thu, 28 May 2020 18:13:24 +0000
      Labels:         app=nginx
                      pod-template-hash=574b87c764
      Annotations:    kubernetes.io/psp: eks.privileged
      Status:         Running
      IP:             192.168.10.170
      Controlled By:  ReplicaSet/my-service-a-574b87c764
      Init Containers:
        proxyinit:
          Container ID:   docker://2c255a985ce6f66d9d00a6c235fc80f087d7a002b2a9d5f1c0cf5699852a6d17
          Image:          111345817488.dkr.ecr.us-west-2.amazonaws.com/aws-appmesh-proxy-route-manager:v2
          Image ID:       docker-pullable://111345817488.dkr.ecr.us-west-2.amazonaws.com/aws-appmesh-proxy-route-manager@sha256:4534295b39133bab10f70846e220d5d4c56c6039ecf6d9aab5e2ab609f377dc6
          Port:           <none>
          Host Port:      <none>
          State:          Terminated
            Reason:       Completed
            Exit Code:    0
            Started:      Thu, 28 May 2020 18:13:25 +0000
            Finished:     Thu, 28 May 2020 18:13:26 +0000
          Ready:          True
          Restart Count:  0
          Requests:
            cpu:     10m
            memory:  32Mi
          Environment:
            APPMESH_START_ENABLED:         1
            APPMESH_IGNORE_UID:            1337
            APPMESH_ENVOY_INGRESS_PORT:    15000
            APPMESH_ENVOY_EGRESS_PORT:     15001
            APPMESH_APP_PORTS:             80
            APPMESH_EGRESS_IGNORED_IP:     169.254.169.254
            APPMESH_EGRESS_IGNORED_PORTS:  22
          Mounts:
            /var/run/secrets/kubernetes.io/serviceaccount from default-token-gq7wk (ro)
      Containers:
        nginx:
          Container ID:   docker://a733fd342fb2087bbd682db4d7b9390a99a6d152a57db503eeaa23de181590ef
          Image:          nginx:1.14.2
          Image ID:       docker-pullable://nginx@sha256:f7988fb6c02e0ce69257d9bd9cf37ae20a60f1df7563c3a2a6abe24160306b8d
          Port:           80/TCP
          Host Port:      0/TCP
          State:          Running
            Started:      Thu, 28 May 2020 18:13:32 +0000
          Ready:          True
          Restart Count:  0
          Environment:    <none>
          Mounts:
            /var/run/secrets/kubernetes.io/serviceaccount from default-token-gq7wk (ro)
        envoy:
          Container ID:   docker://9b5fc60c17e0feb4f367038734782afd3113e4d80b500e4ce1fe71ed0cd36ee3
          Image:          840364872350.dkr.ecr.us-west-2.amazonaws.com/aws-appmesh-envoy:v1.12.3.0-prod
          Image ID:       docker-pullable://840364872350.dkr.ecr.us-west-2.amazonaws.com/aws-appmesh-envoy@sha256:f7ba6f019430c43f4fbadf3035e0a7c1555362a56a79d2d84280b2967595eeaf
          Port:           9901/TCP
          Host Port:      0/TCP
          State:          Running
            Started:      Thu, 28 May 2020 18:13:32 +0000
          Ready:          True
          Restart Count:  0
          Requests:
            cpu:     10m
            memory:  32Mi
          Environment:
            APPMESH_VIRTUAL_NODE_NAME:  mesh/my-mesh/virtualNode/my-service-a_my-app-1
            APPMESH_PREVIEW:            0
            ENVOY_LOG_LEVEL:            info
            AWS_REGION:                 us-west-2
          Mounts:
            /var/run/secrets/kubernetes.io/serviceaccount from default-token-gq7wk (ro)
      Conditions:
        Type              Status
        Initialized       True
        Ready             True
        ContainersReady   True
        PodScheduled      True
      Volumes:
        default-token-gq7wk:
          Type:        Secret (a volume populated by a Secret)
          SecretName:  default-token-gq7wk
          Optional:    false
      QoS Class:       Burstable
      Node-Selectors:  <none>
      Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                       node.kubernetes.io/unreachable:NoExecute for 300s
      Events:
        Type    Reason     Age    From                                                   Message
        ----    ------     ----   ----                                                   -------
        Normal  Scheduled  3m40s  default-scheduler                                      Successfully assigned my-app-1/my-service-a-574b87c764-5gps5 to ip-192-168-11-213.us-west-2.compute.internal
        Normal  Pulled     3m39s  kubelet, ip-192-168-11-213.us-west-2.compute.internal  Container image "111345817488.dkr.ecr.us-west-2.amazonaws.com/aws-appmesh-proxy-route-manager:v2" already present on machine
        Normal  Created    3m39s  kubelet, ip-192-168-11-213.us-west-2.compute.internal  Created container proxyinit
        Normal  Started    3m39s  kubelet, ip-192-168-11-213.us-west-2.compute.internal  Started container proxyinit
        Normal  Pulling    3m37s  kubelet, ip-192-168-11-213.us-west-2.compute.internal  Pulling image "nginx:1.14.2"
        Normal  Pulled     3m33s  kubelet, ip-192-168-11-213.us-west-2.compute.internal  Successfully pulled image "nginx:1.14.2"
        Normal  Created    3m32s  kubelet, ip-192-168-11-213.us-west-2.compute.internal  Created container nginx
        Normal  Started    3m32s  kubelet, ip-192-168-11-213.us-west-2.compute.internal  Started container nginx
        Normal  Pulled     3m32s  kubelet, ip-192-168-11-213.us-west-2.compute.internal  Container image "840364872350.dkr.ecr.us-west-2.amazonaws.com/aws-appmesh-envoy:v1.12.3.0-prod" already present on machine
        Normal  Created    3m32s  kubelet, ip-192-168-11-213.us-west-2.compute.internal  Created container envoy
        Normal  Started    3m32s  kubelet, ip-192-168-11-213.us-west-2.compute.internal  Started container envoy
      ...
      ```

      In the preceding output, you can see that the `proxyinit` and `envoy` containers were added to the pod\.

## Step 4: Clean up<a name="remove-integration"></a>

Remove all of the example resources created in this tutorial\. The controller also removes the resources that were created in App Mesh\.

```
kubectl delete namespace my-app-1
```

\(Optional\) You can remove the Kubernetes integration components\.

```
helm delete appmesh-manager -n appmesh-system
```

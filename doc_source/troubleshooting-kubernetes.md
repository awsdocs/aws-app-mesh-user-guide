# App Mesh Kubernetes troubleshooting<a name="troubleshooting-kubernetes"></a>

This topic details common issues that you may experience when you use App Mesh with Kubernetes\.

## App Mesh resources created in Kubernetes cannot be found in App Mesh<a name="ts-kubernetes-missing-resources"></a>

**Symptoms**  
You have created the App Mesh resources using the Kubernetes custom resource definition \(CRD\), but the resources that you created are not visible in App Mesh when you use the AWS Management Console or APIs\.

**Resolution**  
The likely cause is an error in the Kubernetes manager for App Mesh\. For more information, see [Troubleshooting](https://github.com/aws/aws-app-mesh-controller-for-k8s/blob/master/docs/troubleshoot.md) on GitHub\. Check the manager logs for any errors or warnings indicating that the App Mesh manager could not create any resources\. 

```
kubectl logs -n appmesh-system -f \
    $(kubectl get pods -n appmesh-system -o name | grep manager)
```

If your issue is still not resolved, then you can provide us with details on what you're experiencing by filing a [GitHub issue](https://github.com/aws/aws-app-mesh-controller-for-k8s/issues)\.

## Pods are failing readiness and liveliness checks after Envoy sidecar is injected<a name="ts-kubernetes-pods-after-injection"></a>

**Symptoms**  
Pods for your application were previously running successfully, but after the Envoy sidecar is injected into a pod, readiness and liveliness checks begin failing\.

**Resolution**  
Make sure that the pod has a mesh and virtual node associated with it\. A pod must be part of a namespace that has sidecar injection enabled, is part of a mesh and has a corresponding virtual node\. Virtual node is mapped to a pod using podSelector->matchLabels \.

Make sure your namespace has mesh label and sidecar injection label set:

```
---
apiVersion: v1
kind: Namespace
metadata:
  name: app-ns
  labels:
    mesh: my-mesh
    appmesh.k8s.aws/sidecarInjectorWebhook: enabled
```

Here's a sample virtual node and a pod:

```
---
apiVersion: appmesh.k8s.aws/v1beta2
kind: VirtualNode
metadata:
  name: my-app
  namespace: app-ns
spec:
  podSelector:
    matchLabels:
      app: my-app
```


```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  namespace: app-ns
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
...
```

Make sure that the Envoy container that was injected into the pod has bootstrapped with App Mesh’s Envoy management service\. You can verify any errors by referencing the error codes in [Envoy disconnected from App Mesh Envoy management service with error text](troubleshooting-setup.md#ts-setup-grpc-error-codes)\. You can use the following command to inspect Envoy logs for the relevant pod\.

```
kubectl logs -n appmesh-system -f \
    $(kubectl get pods -n appmesh-system -o name | grep manager) \
    | grep "gRPC config stream closed"
```

If your issue is still not resolved, then you can provide us with details on what you're experiencing by filing a [GitHub issue](https://github.com/aws/aws-app-mesh-controller-for-k8s/issues)\.

## Pods not registering or deregistering as AWS Cloud Map instances<a name="ts-kubernetes-pods-cmap"></a>

**Symptoms**  
Your Kubernetes pods are not being registered in or de\-registered from AWS Cloud Map as part of their lifecycle\. A pod may start successfully and be ready to serve traffic, but not receive any\. When a pod is terminated, clients may still retain its IP address and attempt to send traffic to it, failing\.

**Resolution**  

To mitigate this issue:
+ Make sure that you are running the latest version of the App Mesh manager for Kubernetes\.

```
kubectl get deployment -n appmesh-system appmesh-manager -o json | \
            jq -r ".spec.template.spec.containers[].image" | cut -f2 -d ':'
```

+ Make sure you have podSelector label for the pod in your virtual node definition\.
+ If your issue persists, run the following command to inspect your manager logs for errors that may help reveal the underlying issue\.

  ```
  kubectl logs -n appmesh-system \
      $(kubectl get pods -n appmesh-system -o name | grep appmesh-manager)
  ```
+ Consider using the following command to restart your manager pods\. This may fix any synchronization issues\. Please consider filing a bug if this was related to synchronization\.

  ```
  kubectl delete -n appmesh-system \
      $(kubectl get pods -n appmesh-system -o name | grep appmesh-manager)
  ```

If your issue is still not resolved, then you can provide us with details on what you're experiencing by filing a [GitHub issue](https://github.com/aws/aws-app-mesh-controller-for-k8s/issues)\.

## Cannot determine where a pod for an App Mesh resource is running<a name="ts-kubernetes-where-pod-running"></a>

**Symptoms**  
When you run App Mesh on a Kubernetes cluster, an operator cannot determine where a workload, or pod, is running for a given App Mesh resource\.

**Resolution**  
You can query which pods are running for a given virtual node name with the following command\.

```
kubectl get pods --all-namespaces -o json | \
            jq '.items[]? | select(.spec.containers[]?.env[]? | .name == "APPMESH_VIRTUAL_NODE_NAME" and select(.value | contains("virtual-node-name"))) | .metadata.name'
```

If your issue is still not resolved, then you can provide us with details on what you're experiencing by filing a [GitHub issue](https://github.com/aws/aws-app-mesh-controller-for-k8s/issues)\.

## Cannot determine if Envoy sidecar is pointing to the correct virtual node in App Mesh<a name="ts-kubernetes-envoy-vn-mapping"></a>

**Symptoms**
With changes in virtual node configuration, traffic is not changing. Verify envoy is pointed to the correct virtual node\.

**Resolution**
Ensure that envoy is pointing to the virtual node that selects the pod using podSelector. This can be done by looking at Envoy config\.

Exec into the Envoy container and dump the config\.

```
kubectl exec -it -n <NAMESPACE> <POD_NAME> -c envoy curl localhost:9901/config_dump | grep cluster
```

The config dump should return something like following and you will be able to see the full path `mesh/MESH_NAME/virtualNode/VIRTUAL_NODE_NAME/`\.
```
*"cluster": "mesh/howto-k8s-http2/virtualNode/red_howto-k8s-http2"*,
   "dynamic_active_clusters": [
     "cluster": {
     "cluster": {
       "cluster_name": "cds_ingress_howto-k8s-http2_red_howto-k8s-http2_http2_8080",
           "cluster": "cds_egress_howto-k8s-http2_amazonaws"
           "cluster": "cds_ingress_howto-k8s-http2_red_howto-k8s-http2_http2_8080"
```

If envoy does not point to the correct virtual node, restart the pod and it should point to the correct virtual node

If your issue is still not resolved, then you can provide us with details on what you're experiencing by filing a [GitHub issue](https://github.com/aws/aws-app-mesh-controller-for-k8s/issues)\.

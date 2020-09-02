# App Mesh Kubernetes troubleshooting<a name="troubleshooting-kubernetes"></a>

This topic details common issues that you may experience when you use App Mesh with Kubernetes\.

## App Mesh resources created in Kubernetes cannot be found in App Mesh<a name="ts-kubernetes-missing-resources"></a>

**Symptoms**  
You have created the App Mesh resources using the Kubernetes custom resource definition \(CRD\), but the resources that you created are not visible in App Mesh when you use the AWS Management Console or APIs\.

**Resolution**  
The likely cause is an error in the Kubernetes controller for App Mesh\. For more information, see [Troubleshooting](https://github.com/aws/aws-app-mesh-controller-for-k8s/blob/master/docs/guide/troubleshooting.md) on GitHub\. Check the controller logs for any errors or warnings indicating that the controller could not create any resources\. 

```
kubectl logs -n appmesh-system -f \
    $(kubectl get pods -n appmesh-system -o name | grep controller)
```

If your issue is still not resolved, then consider opening a [GitHub issue](https://github.com/aws/aws-app-mesh-roadmap/issues/new?assignees=&labels=Bug&template=issue--bug-report.md&title=Bug%3A+describe+bug+here) or contact [AWS Support](http://aws.amazon.com/premiumsupport/)\.

## Pods are failing readiness and liveliness checks after Envoy sidecar is injected<a name="ts-kubernetes-pods-after-injection"></a>

**Symptoms**  
Pods for your application were previously running successfully, but after the Envoy sidecar is injected into a pod, readiness and liveliness checks begin failing\.

**Resolution**  
Make sure that the pod has a mesh and virtual node associated with it\. Mesh and virtual node settings are set through annotations on the associated deployment object, as in the following example\. 

```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  namespace: default
spec:
  replicas: 3
  template:
    metadata:
      annotations:
        *appmesh**.k8s.aws/mesh: my-mesh
        appmesh.k8s.aws/virtualNode:my-virtual-node
...
```

Make sure that the Envoy container that was injected into the pod has bootstrapped with App Mesh’s Envoy management service\. You can verify any errors by referencing the error codes in [Envoy disconnected from App Mesh Envoy management service with error text](troubleshooting-setup.md#ts-setup-grpc-error-codes)\. You can use the following command to inspect Envoy logs for the relevant pod\.

```
kubectl logs -n appmesh-system -f \
    $(kubectl get pods -n appmesh-system -o name | grep controller) \
    | grep "gRPC config stream closed"
```

If your issue is still not resolved, then consider opening a [GitHub issue](https://github.com/aws/aws-app-mesh-roadmap/issues/new?assignees=&labels=Bug&template=issue--bug-report.md&title=Bug%3A+describe+bug+here) or contact [AWS Support](http://aws.amazon.com/premiumsupport/)\.

## Pods not registering or deregistering as AWS Cloud Map instances<a name="ts-kubernetes-pods-cmap"></a>

**Symptoms**  
Your Kubernetes pods are not being registered in or de\-registered from AWS Cloud Map as part of their life cycle\. A pod may start successfully and be ready to serve traffic, but not receive any\. When a pod is terminated, clients may still retain its IP address and attempt to send traffic to it, failing\.

**Resolution**  
This is a known issue\. For more information, see the [Pods don't get auto registered/deregistered in kubernetes with AWS Cloud Map](https://github.com/aws/aws-app-mesh-controller-for-k8s/issues/159) GitHub issue\. Due to the relationship between pods, App Mesh virtual nodes, and AWS Cloud Map resources, the [App Mesh controller for Kubernetes](https://github.com/aws/aws-app-mesh-controller-for-k8s) may become desynchronized and lose resources\. For example, this can happen if a virtual node resource is deleted from Kubernetes before terminating its associated pods\. 

To mitigate this issue:
+ Make sure that you are running the latest version of the App Mesh controller for Kubernetes\.
+ Make sure that the AWS Cloud Map `namespaceName` and `serviceName` are correct in your virtual node definition\.
+ Make sure that you delete any associated pods prior to deleting your virtual node definition\. If you need help identifying which pods are associated with a virtual node, see [Cannot determine where a pod for an App Mesh resource is running](#ts-kubernetes-where-pod-running)\.
+ If your issue persists, run the following command to inspect your controller logs for errors that may help reveal the underlying issue\.

  ```
  kubectl logs -n appmesh-system \
      $(kubectl get pods -n appmesh-system -o name | grep appmesh-controller)
  ```
+ Consider using the following command to restart your controller pods\. This may fix synchronization issues\.

  ```
  kubectl delete -n appmesh-system \
      $(kubectl get pods -n appmesh-system -o name | grep appmesh-controller)
  ```

If your issue is still not resolved, then consider opening a [GitHub issue](https://github.com/aws/aws-app-mesh-roadmap/issues/new?assignees=&labels=Bug&template=issue--bug-report.md&title=Bug%3A+describe+bug+here) or contact [AWS Support](http://aws.amazon.com/premiumsupport/)\.

## Cannot determine where a pod for an App Mesh resource is running<a name="ts-kubernetes-where-pod-running"></a>

**Symptoms**  
When you run App Mesh on a Kubernetes cluster, an operator cannot determine where a workload, or pod, is running for a given App Mesh resource\.

**Resolution**  
Kubernetes pod resources are annotated with the mesh and virtual node that they are associated to\. You can query which pods are running for a given virtual node name with the following command\.

```
kubectl get pods --all-namespaces -o json | \
    jq '.items[] | { metadata } | select(.metadata.annotations."appmesh.k8s.aws/virtualNode" == "virtual-node-name")'
```

If your issue is still not resolved, then consider opening a [GitHub issue](https://github.com/aws/aws-app-mesh-roadmap/issues/new?assignees=&labels=Bug&template=issue--bug-report.md&title=Bug%3A+describe+bug+here) or contact [AWS Support](http://aws.amazon.com/premiumsupport/)\.

## Cannot determine what App Mesh resource a pod is running as<a name="ts-kubernetes-pod-running-as"></a>

**Symptoms**  
When running App Mesh on a Kubernetes cluster, an operator cannot determine what App Mesh resource a given pod is running as\.

**Resolution**  
Kubernetes pod resources are annotated with the mesh and virtual node that they are associated to\. You can output the mesh and virtual node names by querying the pod directly using the following command\.

```
kubectl get pod pod-name -n namespace -o json | \
    jq '{ "mesh": .metadata.annotations."appmesh.k8s.aws/mesh", "virtualNode": .metadata.annotations."appmesh.k8s.aws/virtualNode" }'
```

If your issue is still not resolved, then consider opening a [GitHub issue](https://github.com/aws/aws-app-mesh-roadmap/issues/new?assignees=&labels=Bug&template=issue--bug-report.md&title=Bug%3A+describe+bug+here) or contact [AWS Support](http://aws.amazon.com/premiumsupport/)\.
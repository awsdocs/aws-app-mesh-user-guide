# Monitoring your application using Envoy metrics<a name="envoy-metrics"></a>

Envoy classifies its metrics into the following major categories:
+ **Downstream**—Metrics that relate to connections and requests that come into the proxy\.
+ **Upstream**—Metrics that relate to outgoing connections and requests made by the proxy\.
+ **Server**—Metrics that describe the internal state of Envoy\. These include metrics like uptime or allocated memory\.

In App Mesh, the proxy intercepts upstream and downstream traffic\. For example, requests received from your clients as well as requests made by your service container are classified as downstream traffic by Envoy\. To distinguish between these different types of upstream and downstream traffic, App Mesh further categorizes Envoy metrics depending on the traffic direction relative to your service:
+ **Ingress**—Metrics and resources relating to connections and requests that flow to your service container\.
+ **Egress**—Metrics and resources relating to connections and requests that flow from your service container and ultimately out of your Amazon ECS task or Kubernetes pod\.

The following picture shows the communication between the proxy and service containers\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/app-mesh/latest/userguide/images/task-proxy-container.png)

**Resource naming conventions**

It's useful to understand how Envoy views your mesh and how its resources map back to the resources you define in App Mesh\. These are the primary Envoy resources that App Mesh configures:
+ **Listeners**—The addresses and ports the proxy listens for downstream connections on\. In the previous picture, App Mesh creates an ingress listener for traffic coming into your Amazon ECS task or Kubernetes pod and an egress listener for traffic leaving your service container\.
+ **Clusters**—A named group of upstream endpoints that the proxy connects and routes traffic to\. In App Mesh, your service container is represented as a cluster, as well as all other virtual nodes your service can connect to\.
+ **Routes**—These correspond to routes you define in your mesh\. They contain the conditions by which the proxy matches a request as well as the target cluster a request is sent to\.
+ **Endpoints and cluster load assignments**—The IP addresses of upstream clusters\. When using AWS Cloud Map as your service discovery mechanism for virtual nodes, App Mesh sends discovered service instances as endpoint resources to your proxy\.
+ **Secrets**—These include, but are not limited to, your encryption keys and TLS certificates\. When using AWS Certificate Manager as a source for client and server certificates, App Mesh sends public and private certificates to your proxy as secret resources\.

App Mesh uses a consistent scheme for naming Envoy resources that you can use to relate back to your mesh\.

Understanding the naming scheme for listeners and clusters is important in understanding Envoy’s metrics in App Mesh\.

**Listener names**

Listeners are named using the following format:

```
lds_<traffic direction>_<listener IP address>_<listening port>
```

You will typically see the following listeners configured in Envoy:
+ `lds_ingress_0.0.0.0_15000`
+ `lds_egress_0.0.0.0_15001`

Using either a Kubernetes CNI plugin or IP tables rules, traffic in your Amazon ECS task or Kubernetes pod is directed to the ports `15000` and `15001`\. App Mesh configures Envoy with these two listeners to accept ingress \(incoming\) and egress \(outgoing\) traffic\. If you do not have a listener configured on your virtual node, you shouldn't see an ingress listener\.

**Cluster names**

Most clusters use the following format:

```
cds_<traffic direction>_<mesh name>_<virtual node name>_<protocol>_<port>
```

Virtual nodes that your services communicate with each have their own cluster\. As mentioned previously, App Mesh creates a cluster for the service running next to Envoy so the proxy can send ingress traffic to it\.

For example, if you have a virtual node named `my-virtual-node` that listens for http traffic on port `8080` and that virtual node is in a mesh named `my-mesh`, App Mesh creates a cluster named `cds_ingress_my-mesh_my-virtual-node_http_8080`\. This cluster serves as the destination for traffic into `my-virtual-node`’s service container\.

App Mesh may also create the following types of additional special clusters\. These other clusters do not necessarily correspond to resources that you explicitly define in your mesh\.
+ Clusters used to reach other AWS services\. This type allows your mesh to reach most AWS services by default: `cds_egress_<mesh name>_amazonaws`\.
+ Cluster used to perform routing for virtual gateways\. This can generally be safely ignored: \.
  + For single listeners: `cds_ingress_<mesh name>_<virtual gateway name>_self_redirect_<protocol>_<port>`
  + For multiple listeners: `cds_ingress_<mesh name>_<virtual gateway name>_self_redirect_<ingress_listener_port>_<protocol>_<port>`
+ The cluster who’s endpoint you can define, such as TLS, when you retrieve secrets using Envoy’s Secret Discovery Service: `static_cluster_sds_unix_socket`\.

## Example application metrics<a name="envoy-metrics-examples"></a>

To illustrate the metrics available in Envoy, the following sample application has three virtual nodes\. The virtual services, virtual routers, and routes in the mesh can be ignored since they are not reflected in Envoy’s metrics\. In this example, all services listen for http traffic on port 8080\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/app-mesh/latest/userguide/images/envoy-metric-example1.png)

We recommend adding the environment variable `ENABLE_ENVOY_STATS_TAGS=1` to the Envoy proxy containers running in your mesh\. This adds the following metric dimensions to all metrics emitted by the proxy:
+ `appmesh.mesh`
+ `appmesh.virtual_node`
+ `appmesh.virtual_gateway`

These tags are set to the name of mesh, virtual node, or virtual gateway to allow filtering metrics using the names of resources in your mesh\.

**Resource names**

The website virtual node’s proxy has the following resources:
+ Two listeners for ingress and egress traffic:
  + `lds_ingress_0.0.0.0_15000`
  + `lds_egress_0.0.0.0_15001`
+ Two egress clusters, representing the two virtual node back ends:
  + `cds_egress_online-store_product-details_http_8080`
  + `cds_egress_online-store_cart_http_8080`
+ An ingress cluster for the website service container:
  + `cds_ingress_online-store_website_http_8080`

**Example listener metrics**
+ `listener.0.0.0.0_15000.downstream_cx_active`—Number of active ingress network connections to Envoy\.
+ `listener.0.0.0.0_15001.downstream_cx_active`—Number of active egress network connections to Envoy\. Connections made by your application to external services is included in this count\.
+ `listener.0.0.0.0_15000.downstream_cx_total`—Total number of ingress network connections to Envoy\.
+ `listener.0.0.0.0_15001.downstream_cx_total`—Total number of egress network connections to Envoy\.

For the full set of listener metrics, see [Statistics](https://www.envoyproxy.io/docs/envoy/latest/configuration/listeners/stats) in the Envoy documentation\.

**Example cluster metrics**
+ `cluster_manager.active_clusters`—The total number of clusters that Envoy has established at least one connection to\.
+ `cluster_manager.warming_clusters`—The total number of clusters that Envoy has yet to connect to\.

The following cluster metrics use the format of `cluster.<cluster name>.<metric name>`\. These metric names are unique to the application example and are emitted by the website Envoy container:
+ `cluster.cds_egress_online-store_product-details_http_8080.upstream_cx_total`—Total number of connections between website and product\-details\.
+ `cluster.cds_egress_online-store_product-details_http_8080.upstream_cx_connect_fail`—Total number of failed connections between website and product\-details\.
+ `cluster.cds_egress_online-store_product-details_http_8080.health_check.failure`—Total number of failed health checks between website and product\-details\.
+ `cluster.cds_egress_online-store_product-details_http_8080.upstream_rq_total`—Total number of requests made between website and product\-details\.
+ `cluster.cds_egress_online-store_product-details_http_8080.upstream_rq_time`—Time taken by requests made between website and product\-details\.
+ `cluster.cds_egress_online-store_product-details_http_8080.upstream_rq_2xx`—Number of HTTP 2xx responses received by website from product\-details\.

For the full set of HTTP metrics, see [Statistics](https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_conn_man/stats) in the Envoy documentation\.

**Management server metrics**

Envoy also emits metrics related to its connection to the App Mesh control plane, which acts as Envoy’s management server\. We recommend monitoring some of these metrics as a way to notify you when your proxies become desynchronized from the control plane for extended periods of time\. Loss of connectivity to the control plane or failed updates prevent your proxies from receiving new configuration from App Mesh, including mesh changes made via App Mesh APIs\.
+ `control_plane.connected_state`—This metric is set to 1 when the proxy is connected to App Mesh, otherwise it is 0\.
+ `*.update_rejected`—Total number of configuration updates that are rejected by Envoy\. These are usually due to user misconfiguration\. For example, if you configure App Mesh to read a TLS certificate from a file that cannot be read by Envoy, the update containing the path to that certificate is rejected\.
  + For Listener updated rejected, the stats will be `listener_manager.lds.update_rejected`\.
  + For Cluster updated rejected, the stats will be `cluster_manager.cds.update_rejected`\.
+ `*.update_success`—Number of successful configuration updates made by App Mesh to your proxy\. These include the initial configuration payload sent when a new Envoy container is started\.
  + For Listener updated success, the stats will be `listener_manager.lds.update_success`\.
  + For Cluster updated success, the stats will be `cluster_manager.cds.update_success`\.

For the set of management server metrics, see [Management Server](https://www.envoyproxy.io/docs/envoy/latest/configuration/overview/mgmt_server) in the Envoy documentation\.
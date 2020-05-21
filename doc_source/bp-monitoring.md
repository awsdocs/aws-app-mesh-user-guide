# Monitoring and failure resolution<a name="bp-monitoring"></a>

We recommend that you monitor the metrics discussed in this topic when using App Mesh\.

**Application metrics**  
While the observability metrics Envoy provides you will help you better diagnose issues between services, it’s also important to remember that failures in your service mesh may not be translating into issues for your services and customers\. Similarly, seeing no issues in your Envoy metrics doesn’t mean that your applications are healthy\.

**Envoy metrics**  
Envoy can be configured to emit metrics to a variety of sinks such as [`DogStatsD`](troubleshooting-best-practices.md#ts-bp-enable-envoy-statsd-integration) which can be published to Amazon CloudWatch\. These metrics can also be queried for from the Envoy admin interface, which is typically [http://address:9901](http://address:9901/)\. For more information, see [Enable the Envoy proxy administration interface](troubleshooting-best-practices.md#ts-bp-enable-proxy-admin-interface)\. For simplicity, output from the admin interface is shown in the examples that follow\.

Within Envoy, metrics are categorized as either applying to the `egress` traffic leaving your application or `ingress` traffic coming to your application\. Envoy further categories traffic based on which side of the connection the metric refers to, where `downstream` is the original sender of traffic and `upstream` is the next receiver of traffic\.

You can start understanding what Envoy is doing by viewing the classes of HTTP responses that Envoy has returned to your application `downstream` for outbound requests \(`egress`\)\.

```
http.egress.downstream_rq_1xx: 0
http.egress.downstream_rq_2xx: 29953240
http.egress.downstream_rq_3xx: 0
http.egress.downstream_rq_4xx: 189
http.egress.downstream_rq_5xx: 1852
http.egress.downstream_rq_active: 0
http.egress.downstream_rq_completed: 29955281
```

You can also view example metrics for responses returned to applications that are calling your service \(`ingress`\)\.

```
http.ingress.downstream_rq_1xx: 0
http.ingress.downstream_rq_2xx: 12598783
http.ingress.downstream_rq_3xx: 0
http.ingress.downstream_rq_4xx: 16
http.ingress.downstream_rq_5xx: 2042
http.ingress.downstream_rq_active: 0
http.ingress.downstream_rq_completed: 126
```

These previous metrics are a view of what your application is seeing, and what other applications in your service mesh should be seeing\. To understand more about the `upstream` traffic, you need to look at what Envoy sees from its connection pools\. Metrics for these pools have the following format:

```
cluster.cds_${ingress|egress}_${mesh}_${destination_virtual_node}_${protocol}_${port}.${metric}
```

For ingress traffic, the *destination\_virtual\_node* is always your application's virtual node\.

The following example metrics are for a virtual node named *virtual\-node\-a* that's in a mesh named *prod*\. 

```
cluster.cds_egress_prod_virtual-node-a_http_3000.upstream_rq_200: 8974819
cluster.cds_egress_prod_virtual-node-a_http_3000.upstream_rq_2xx: 8974819
cluster.cds_egress_prod_virtual-node-a_http_3000.upstream_rq_409: 189
cluster.cds_egress_prod_virtual-node-a_http_3000.upstream_rq_4xx: 189
cluster.cds_egress_prod_virtual-node-a_http_3000.upstream_rq_500: 5
cluster.cds_egress_prod_virtual-node-a_http_3000.upstream_rq_504: 2
cluster.cds_egress_prod_virtual-node-a_http_3000.upstream_rq_5xx: 7
cluster.cds_egress_prod_virtual-node-a_http_3000.upstream_rq_per_try_timeout: 3
cluster.cds_egress_prod_virtual-node-a_http_3000.upstream_rq_retry: 1
cluster.cds_egress_prod_virtual-node-a_http_3000.upstream_rq_retry_overflow: 0
cluster.cds_egress_prod_virtual-node-a_http_3000.upstream_rq_retry_success: 1
```

The previous example metrics show nearly 9 million `200` requests, with a small number of conflict and server exceptions\.

You can also see metrics on request timeouts, as well as when retries have occurred, and whether they succeeded\.

```
cluster.cds_egress_prod_virtual-node-a_http_3000.health_check.attempt: 94540
cluster.cds_egress_prod_virtual-node-a_http_3000.health_check.degraded: 0
cluster.cds_egress_prod_virtual-node-a_http_3000.health_check.failure: 0
cluster.cds_egress_prod_virtual-node-a_http_3000.health_check.healthy: 3
cluster.cds_egress_prod_virtual-node-a_http_3000.health_check.network_failure: 0
cluster.cds_egress_prod_virtual-node-a_http_3000.health_check.passive_failure: 0
cluster.cds_egress_prod_virtual-node-a_http_3000.health_check.success: 94540
cluster.cds_egress_prod_virtual-node-a_http_3000.health_check.verify_cluster: 0
cluster.cds_egress_prod_virtual-node-a_http_3000.membership_change: 1
cluster.cds_egress_prod_virtual-node-a_http_3000.membership_degraded: 0
cluster.cds_egress_prod_virtual-node-a_http_3000.membership_excluded: 0
cluster.cds_egress_prod_virtual-node-a_http_3000.membership_healthy: 3
cluster.cds_egress_prod_virtual-node-a_http_3000.membership_total: 3
cluster.cds_egress_prod_virtual-node-a_http_3000.lb_healthy_panic: 0
```

Next, you can look at health checks and membership to understand the number of endpoints available and how they are passing healtchecks\. In the following example, you can see that three endpoints are available \(with a single change in membership\), and that they have all passed health checks\. 

```
cluster.cds_egress_prod_virtual-node-a_http_3000.upstream_cx_connect_fail: 0
cluster.cds_egress_prod_virtual-node-a_http_3000.upstream_cx_connect_timeout: 0
cluster.cds_egress_prod_virtual-node-a_http_3000.upstream_cx_destroy_local_with_active_rq: 48
cluster.cds_egress_prod_virtual-node-a_http_3000.upstream_cx_destroy_remote_with_active_rq: 0
cluster.cds_egress_prod_virtual-node-a_http_3000.upstream_cx_destroy_with_active_rq: 48
cluster.cds_egress_prod_virtual-node-a_http_3000.upstream_cx_idle_timeout: 436
cluster.cds_egress_prod_virtual-node-a_http_3000.upstream_cx_none_healthy: 0
cluster.cds_egress_prod_virtual-node-a_http_3000.upstream_cx_overflow: 0
cluster.cds_egress_prod_virtual-node-a_http_3000.upstream_cx_pool_overflow: 0
cluster.cds_egress_prod_virtual-node-a_http_3000.upstream_cx_protocol_error: 0
```

You can see that the destination in the previous example has never gone into `panic mode`, which means that less than 50% of endpoints are known healthy\. Had the destination gone into panic mode, you would see a line similar to the following:

```
cluster.cds_egress_prod_virtual-node-a_http_3000.lb_healthy_panic: 559
```

While panic mode is helpful when there is a systemic issue and you want to route requests somewhere, it does mean you are more likely to see failed requests than when otherwise healthy\.

Finally, you can look at connection metrics, which tell you how Envoy is managing the underlying TCP connections to destinations\. One of the more important metrics to monitor is the `destroy_with_active` metrics, because without retries in place, any in\-flight request is converted into `503` errors\. For guidance in troubleshooting common `503` errors, see [HTTP 503 errors during application deployment](troubleshooting-setup.md#ts-setup-503-during-deployment) and [HTTP 503 – Upstream connect or disconnects or resets before headers errors](troubleshoot-connectivity.md#ts-connectivity-upstream-connect-failures)\.
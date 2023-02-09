# Envoy defaults set by App Mesh<a name="envoy-defaults"></a>

The following sections provide information about the Envoy defaults for the route retry policy and circuit breaker that are set by App Mesh\.

## Default route retry policy<a name="default-retry-policy"></a>

If you had no meshes in your account before July 29, 2020, App Mesh automatically creates a default Envoy route retry policy for all HTTP, HTTP/2, and gRPC requests in any mesh in your account on or after July 29, 2020\. If you had any meshes in your account before July 29, 2020, then no default policy was created for any Envoy routes that existed before, on, or after July 29, 2020\. This is unless you [open a ticket with AWS support](https://console.aws.amazon.com/support/home#/case/create)\. After support processes the ticket, the default policy is created for any future Envoy routes that App Mesh creates on or after the date that the ticket was processed\. For more information about Envoy route retry policies, see [config\.route\.v3\.RetryPolicy](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/route_components.proto#envoy-v3-api-msg-config-route-v3-retrypolicy) in the Envoy documentation\.

App Mesh creates an Envoy route when you either create an App Mesh [route](routes.md) or define a virtual node provider for an App Mesh [virtual service](virtual_services.md)\. Though you can create an App Mesh route retry policy, you can't create an App Mesh retry policy for a virtual node provider\.

The default policy isn't visible through the App Mesh API\. The default policy is only visible through Envoy\. To view the configuration, [enable the administration interface](troubleshooting-best-practices.md#ts-bp-enable-proxy-admin-interface) and send a request to Envoy for a `config_dump`\. The default policy includes the following settings:
+ **Max retries** – `2`
+ **gRPC retry events** – `UNAVAILABLE`
+ **HTTP retry events** – `503`
**Note**  
It's not possible to create an App Mesh route retry policy that looks for a specific HTTP error code\. However, an App Mesh route retry policy can look for `server-error` or `gateway-error`\. Both of these include `503` errors\. For more information, see [Routes](routes.md)\.
+ **TCP retry event** – `connect-failure` and `refused-stream`
**Note**  
It's not possible to create an App Mesh route retry policy that looks for either of these events\. However, an App Mesh route retry policy can look for `connection-error`, which is equivalent to `connect-failure`\. For more information, see [Routes](routes.md)\.
+ **Reset** – Envoy attempts a retry if the upstream server doesn't respond at all \(disconnect/reset/read timeout\)\.

## Default circuit breaker<a name="default-circuit-breaker"></a>

When you deploy an Envoy in App Mesh, Envoy default values are set for some of the circuit breaker settings\. For more information, see [cluster\.CircuitBreakers\.Thresholds](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/cluster/v3/circuit_breaker.proto.html#envoy-v3-api-msg-config-cluster-v3-circuitbreakers-thresholds) in the Envoy documentation\. These settings aren't visible through the App Mesh API\. The settings are only visible through Envoy\. To view the configuration, [enable the administration interface](troubleshooting-best-practices.md#ts-bp-enable-proxy-admin-interface) and send a request to Envoy for a `config_dump`\.

If you had no meshes in your account before July 29, 2020, then for each Envoy that you deploy in a mesh created on or after July 29, 2020, App Mesh effectively disables circuit breakers by changing the Envoy default values for the settings that follow\. If you had any meshes in your account before July 29, 2020, the Envoy default values are set for any Envoy that you deploy in App Mesh on, or after July 29, 2020, unless you [open a ticket with AWS support](https://console.aws.amazon.com/support/home#/case/create)\. Once support processes the ticket, then the App Mesh default values for the following Envoy settings are set by App Mesh on all Envoys that you deploy after the date that the ticket is processed:
+ `max_requests` – `2147483647`
+ `max_pending_requests` – `2147483647`
+ `max_connections` – `2147483647`
+ `max_retries` – `2147483647`

**Note**  
No matter if your Envoys have the Envoy or App Mesh default circuit breaker values, you cannot modify the values\.
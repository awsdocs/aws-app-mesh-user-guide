# App Mesh security troubleshooting<a name="troubleshooting-security"></a>

This topic details common issues that you may experience with App Mesh security\.

## Unable to connect to a backend virtual service with a TLS client policy<a name="ts-security-tls-client-policy"></a>

**Symptoms**  
When adding a TLS client policy to a virtual service backend in a virtual node, connectivity to that backend fails\. When attempting to send traffic to the backend service, the requests fail with an `HTTP 503` response code and the error message: `upstream connect error or disconnect/reset before headers. reset reason: connection failure`\.

**Resolution**  
In order to determine the root cause of the issue, we recommend using the Envoy proxy process logs to help you diagnose the issue\. For more information, see [Enable Envoy debug logging in pre\-production environments](troubleshooting-best-practices.md#ts-bp-enable-envoy-debug-logging)\. Use the following list to determine the cause of the connection failure:
+ Make sure connectivity to the backend is succeeding by ruling out the errors mentioned in [Unable to connect to a virtual service backend](troubleshoot-connectivity.md#ts-connectivity-virtual-service-backend)\.
+ In the Envoy process logs, look for the following errors \(logged at debug level\)\.

  ```
  TLS error: 268435581:SSL routines:OPENSSL_internal:CERTIFICATE_VERIFY_FAILED
  ```

  This error is caused by one or more of the following reasons:
  + The certificate was not signed by one of the certificate authorities defined in the TLS client policy trust bundle\.
  + The certificate is no longer valid \(expired\)\.
  + The Subject Alternative Name \(SAN\) does not match the requested DNS hostname\.
  + Make sure that the certificate offered by the backend service is valid, that it is signed by one of the certificate authorities in your TLS client policies trust bundle, and that it meets the criteria defined in [Transport Layer Security \(TLS\)](tls.md)\.

If your issue is still not resolved, then consider opening a [GitHub issue](https://github.com/aws/aws-app-mesh-roadmap/issues/new?assignees=&labels=Bug&template=issue-bug-report.md&title=Bug%3A+describe+bug+here) or contact [AWS Support](http://aws.amazon.com/premiumsupport/)\. If you believe that you’ve found a security vulnerability or have questions about App Mesh’s security, then see the [AWS vulnerability reporting guidelines](http://aws.amazon.com/security/vulnerability-reporting/)\.

## Unable to connect to a backend virtual service when application is originating TLS<a name="ts-security-originating-tls"></a>

**Symptoms**  
When originating a TLS session from an application, instead of from the Envoy proxy, connectivity to a backend virtual service fails\.

**Resolution**  
This is a known issue\. For more information, see the [Feature Request: TLS negotiation between the downstream application and upstream proxy](https://github.com/aws/aws-app-mesh-roadmap/issues/162) GitHub issue\. In App Mesh, TLS origination is currently supported from the Envoy proxy but not from the application\. To use TLS origination support at the Envoy, disable TLS origination in the application\. This allows the Envoy to read the egress request headers and forward the request to the appropriate destination through a TLS session\. For more information, see [Transport Layer Security \(TLS\)](tls.md)\. 

If your issue is still not resolved, then consider opening a [GitHub issue](https://github.com/aws/aws-app-mesh-roadmap/issues/new?assignees=&labels=Bug&template=issue--bug-report.md&title=Bug%3A+describe+bug+here) or contact [AWS Support](http://aws.amazon.com/premiumsupport/)\. If you believe that you’ve found a security vulnerability or have questions about App Mesh’s security, then see the [AWS vulnerability reporting guidelines](http://aws.amazon.com/security/vulnerability-reporting/)\.

## Unable to assert that connectivity between Envoy proxies is using TLS<a name="ts-security-tls-between-proxies"></a>

**Symptoms**  
Your application has enabled TLS termination on the virtual node or virtual gateway listener, or TLS origination on the backend TLS client policy, but you are unable to assert that connectivity between Envoy proxies is occurring over a TLS\-negotiated session\.

**Resolution**  
Steps defined in this resolution make use of the Envoy administration interface and Envoy statistics\. For help configuring these, see [Enable the Envoy proxy administration interface](troubleshooting-best-practices.md#ts-bp-enable-proxy-admin-interface) and [Enable Envoy DogStatsD integration for metric offload](troubleshooting-best-practices.md#ts-bp-enable-envoy-statsd-integration)\. The following statistics examples use the administration interface for simplicity\.
+ For the Envoy proxy performing TLS termination:
  + Make sure that the TLS certificate has been bootstrapped in the Envoy configuration with the following command\.

    ```
    curl http://my-app.default.svc.cluster.local:9901/certs
    ```

    In the returned output, you should see at least one entry under `certificates[].cert_chain` for the certificate used in TLS termination\.
  + Make sure that the number of successful inbound connections to the proxy’s listener is exactly the same as the number of SSL handshakes plus the number of SSL sessions re\-used, as shown by the following example commands and output\.

    ```
    curl -s http://my-app.default.svc.cluster.local:9901/stats | grep "listener.0.0.0.0_15000" | grep downstream_cx_total
    listener.0.0.0.0_15000.downstream_cx_total: 11
    curl -s http://my-app.default.svc.cluster.local:9901/stats | grep "listener.0.0.0.0_15000" | grep ssl.connection_error
    listener.0.0.0.0_15000.ssl.connection_error: 1
    curl -s http://my-app.default.svc.cluster.local:9901/stats | grep "listener.0.0.0.0_15000" | grep ssl.handshake
    listener.0.0.0.0_15000.ssl.handshake: 9
    curl -s http://my-app.default.svc.cluster.local:9901/stats | grep "listener.0.0.0.0_15000" | grep ssl.session_reused
    listener.0.0.0.0_15000.ssl.session_reused: 1
    # Total CX (11) - SSL Connection Errors (1) == SSL Handshakes (9) + SSL Sessions Re-used (1)
    ```
+ For the Envoy proxy performing TLS origination:
  + Make sure that the TLS trust store has been bootstrapped in the Envoy configuration with the following command\.

    ```
    curl http://my-app.default.svc.cluster.local:9901/certs
    ```

    You should see at least one entry under `certificates[].ca_certs` for the certificates used in validating the backend’s certificate during TLS origination\.
  + Make sure that the number of successful outbound connections to the backend cluster is exactly the same as the number of SSL handshakes plus the number of SSL sessions re\-used, as shown by the following example commands and output\.

    ```
    curl -s http://my-app.default.svc.cluster.local:9901/stats | grep "virtual-node-name" | grep upstream_cx_total
    cluster.cds_egress_mesh-name_virtual-node-name_protocol_port.upstream_cx_total: 11
    curl -s http://my-app.default.svc.cluster.local:9901/stats | grep "virtual-node-name" | grep ssl.connection_error
    cluster.cds_egress_mesh-name_virtual-node-name_protocol_port.ssl.connection_error: 1
    curl -s http://my-app.default.svc.cluster.local:9901/stats | grep "virtual-node-name" | grep ssl.handshake
    cluster.cds_egress_mesh-name_virtual-node-name_protocol_port.ssl.handshake: 9
    curl -s http://my-app.default.svc.cluster.local:9901/stats | grep "virtual-node-name" | grep ssl.session_reused
    cluster.cds_egress_mesh-name_virtual-node-name_protocol_port.ssl.session_reused: 1
    # Total CX (11) - SSL Connection Errors (1) == SSL Handshakes (9) + SSL Sessions Re-used (1)
    ```

If your issue is still not resolved, then consider opening a [GitHub issue](https://github.com/aws/aws-app-mesh-roadmap/issues/new?assignees=&labels=Bug&template=issue--bug-report.md&title=Bug%3A+describe+bug+here) or contact [AWS Support](http://aws.amazon.com/premiumsupport/)\. If you believe that you’ve found a security vulnerability or have questions about App Mesh’s security, then see the [AWS vulnerability reporting guidelines](http://aws.amazon.com/security/vulnerability-reporting/)\.

## Troubleshooting TLS with Elastic Load Balancing<a name="ts-security-tls-elb"></a>

**Symptoms**  
When attempting to configure an Application Load Balancer or Network Load Balancer to encrypt traffic to a virtual node, connectivity and load balancer health checks can fail\.

**Resolution**  
In order to determine the root cause of the issue, you need to check the following:
+ For the Envoy proxy performing TLS termination, you need to rule out any misconfiguration\. Follow the steps provided above in the [Unable to connect to a backend virtual service with a TLS client policy](#ts-security-tls-client-policy)\.
+ For the load balancer, you need to look at the configuration of the `TargetGroup:`
  + Make sure that the `TargetGroup` port matches the virtual node’s defined listener port\.
  + For Application Load Balancers that are originating TLS connections over HTTP to your service, make sure that the `TargetGroup` protocol is set to `HTTPS`\. If health checks are being utilized, make sure that `HealthCheckProtocol` is set to `HTTPS`\. 
  + For Network Load Balancers that are originating TLS connections over TCP to your service, make sure that the `TargetGroup` protocol is set to `TLS`\. If health checks are being utilized, make sure that `HealthCheckProtocol` is set to `TCP`\.
**Note**  
Any updates to `TargetGroup` require changing the `TargetGroup` name\.

With this configured properly, your load balancer should provide a secure connection to your service using the certificate provided to the Envoy proxy\.

If your issue is still not resolved, then consider opening a [GitHub issue](https://github.com/aws/aws-app-mesh-roadmap/issues/new?assignees=&labels=Bug&template=issue--bug-report.md&title=Bug%3A+describe+bug+here) or contact [AWS Support](http://aws.amazon.com/premiumsupport/)\. If you believe that you’ve found a security vulnerability or have questions about App Mesh’s security, then see the [AWS vulnerability reporting guidelines](http://aws.amazon.com/security/vulnerability-reporting/)\.
# Transport Layer Security \(TLS\)<a name="tls"></a>

In App Mesh, Transport Layer Security \(TLS\) encrypts communication between the Envoy proxies deployed on compute resources that are represented in App Mesh by mesh endpoints, such as [Virtual nodes](virtual_nodes.md) and [Virtual gateways](virtual_gateways.md)\. The proxy negotiates and terminates TLS\. When the proxy is deployed with an application, your application code is not responsible for negotiating a TLS session\. The proxy negotiates TLS on your application's behalf\. 

App Mesh allows you to provide the TLS certificate to the proxy in the following ways:
+ A private certificate from AWS Certificate Manager \(ACM\) that is issued by an AWS Certificate Manager Private Certificate Authority \(ACM PCA\)
+ A certificate stored on the local file system of a virtual node that is issued by your own certificate authority \(CA\) 

[Proxy authorization](proxy-authorization.md) must be enabled for the deployed Envoy proxy represented by a mesh endpoint\. We recommend that when you enable proxy authorization, you restrict access to only the mesh endpoint that you're enabling encryption for\.

## Certificate requirements<a name="virtual-node-tls-prerequisites"></a>

One of the Subject Alternative Names \(SANs\) on the certificate must match specific criteria, depending on how the actual service represented by a mesh endpoint is discovered\. 
+ **DNS** – One of the certificate SANs must match the value provided in the DNS service discovery settings\. For an application with the service discovery name `mesh-endpoint.apps.local`, you can create a certificate matching that name, or a certificate with the wild card `*.apps.local`\.
+ **AWS Cloud Map** – One of the certificate SANs must match the value provided in the AWS Cloud Map service discovery settings using the format `service-name.namespace-name`\. For an application with the AWS Cloud Map service discovery settings of serviceName `mesh-endpoint` and the namespaceName `apps.local`, you can create a certificate matching the name `mesh-endpoint.apps.local`, or a certificate with the wild card `*.apps.local.`

For both discovery mechanisms, if none of the certificate SANs match the DNS service discovery settings, the connection between Envoys fails with the following error message, as seen from the client Envoy\. 

```
TLS error: 268435581:SSL routines:OPENSSL_internal:CERTIFICATE_VERIFY_FAILED
```

For additional requirements, select the issuer of the certificate that you're using\.

### ACM PCA<a name="certificate-pca"></a>

The certificate or CA's certificate must be stored in ACM in the same Region and AWS account as the mesh endpoint that will use the certificate\. If you don't have an ACM Private CA, then you must [create one](https://docs.aws.amazon.com/acm-pca/latest/userguide/PcaCreateCa.html) before you can request a certificate from it\. For more information about requesting a certificate from an existing ACM PCA using ACM, see [Request a Private Certificate](https://docs.aws.amazon.com/acm/latest/userguide/gs-acm-request-private.html)\. The certificate cannot be a public certificate\.

The private CAs that you use for TLS client policies must be root CAs\.

To configure a virtual node with certificates and CAs from ACM PCA, the principal \(such as a user or role\) that you use to call App Mesh must have the following IAM permissions: 
+ For any certificates that you add to a listener's TLS configuration, the principal must have the `acm:DescribeCertificate` permission\.
+ For any CAs configured on a TLS client policy, the principal must have the `acm-pca:DescribeCertificateAuthority` permission\.

You can add these permissions to an existing IAM policy that is attached to a principal or create a new principal and policy and attach the policy to the principal\. For more information, see [Editing IAM Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-edit.html), [Creating IAM Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_create-console.html), and [Adding IAM Identity Permissions](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-attach-detach.html#add-policies-console)\.

**Note**  
You pay a monthly fee for the operation of each ACM PCA until you delete it\. You also pay for the private certificates you issue each month and private certificates that you export\. For more information, see [AWS Certificate Manager Pricing](http://aws.amazon.com/certificate-manager/pricing/)\.

When you enable [proxy authorization](proxy-authorization.md) for the Envoy Proxy that a mesh endpoint represents, the IAM role that you use must be assigned the following IAM permissions:
+ For any certificates configured on a virtual node’s listener, the role must have the `acm:ExportCertificate` permission\.
+ For any CAs configured on a TLS client policy, the role must have the `acm-pca:GetCertificateAuthorityCertificate` permission\.

### Your own CA<a name="certificate-file"></a>

When using a certificate from your own CA, you'll need a public key, a private key, and a certificate chain from the CA\. The files must all be stored in a location accessible to the proxy\.

## How App Mesh configures Envoys to negotiate TLS<a name="envoy-configuration-tls"></a>

App Mesh uses the mesh endpoint configuration of both the client and server when determining how to configure the communication between Envoys in a mesh\.

**With client policies**  
When a client policy is enforcing the use of TLS, and one of the ports in the client policy matches the port of the server's policy, the client policy is used to configure the TLS validation context of the client\. For example, if a virtual gateway's client policy matches a virtual node's server policy, TLS negotiation will be attempted between the proxies using the settings defined in the virtual gateway's client policy\. If the client policy does not match the port of the server's policy, TLS between the proxies may or may not be negotiated, depending on the server policy's TLS settings\. 

**Without client policies**  
If the client has not configured a client policy, or the client policy does not match the port of the server, App Mesh will use the server to determine whether or not to negotiate TLS from the client, and how\. For example, if a virtual gateway has not specified a client policy, and a virtual node has not configured TLS termination, TLS will not be negotiated between the proxies\. If a client has not specified a matching client policy, and a server has been configured with TLS modes `STRICT` or `PERMISSIVE`, the proxies will be configured to negotiate TLS\. Depending on how the certificates have been provided for TLS termination, the following additional behavior applies\.
+ **ACM\-managed TLS certificates** – When a server has configured TLS termination using an ACM\-managed certificate, App Mesh automatically configures clients to negotiate TLS and validate the certificate against the root CA that the certificate chains up to\.
+ **File\-based TLS certificates** – When a server has configured TLS termination using a certificate from the proxy's local file system, App Mesh automatically configures a client to negotiate TLS, but the certificate of the server is not validated\.

## Verify encryption<a name="verify-encryption"></a>

Once you've enabled TLS, you can query the Envoy proxy to confirm that communication is encrypted\. The Envoy proxy emits statistics on resources that can help you understand if your TLS communication is working properly\. For example, the Envoy proxy records statistics on the number of successful TLS handshakes it has negotiated for a specified mesh endpoint\. Determine how many successful TLS handshakes there were for a mesh endpoint named `my-mesh-endpoint` with the following command\.

```
curl -s 'http://my-mesh-endpoint.apps.local:9901/stats' | grep ssl.handshake
```

In the following example returned output, there were three handshakes for the mesh endpoint, so communication is encrypted\.

```
listener.0.0.0.0_15000.ssl.handshake: 3
```

The Envoy proxy also emits statistics when TLS negotiation is failing\. Determine whether there were TLS errors for the mesh endpoint\.

```
curl -s 'http://my-mesh-endpoint.apps.local:9901/stats' | grep -e "ssl.*\(fail\|error\)"
```

In the example returned output, there were zero errors for several statistics, so the TLS negotiation succeeded\.

```
listener.0.0.0.0_15000.ssl.connection_error: 0
listener.0.0.0.0_15000.ssl.fail_verify_cert_hash: 0
listener.0.0.0.0_15000.ssl.fail_verify_error: 0
listener.0.0.0.0_15000.ssl.fail_verify_no_cert: 0
listener.0.0.0.0_15000.ssl.ssl.fail_verify_san: 0
```

For more information about Envoy TLS statistics, see [Envoy Listener Statistics](https://www.envoyproxy.io/docs/envoy/latest/configuration/listeners/stats)\.

## Certificate renewal<a name="certificate-renewal"></a>

**ACM PCA**  
When you renew a certificate with ACM, the renewed certificate will be automatically distributed to your connected proxies within 35 minutes of the renewal completion\. We recommend using managed renewal to automatically renew certificates nearing the end of their validity period\. For more information, see [Managed Renewal for ACM's Amazon\-Issued Certificates](https://docs.aws.amazon.com/acm/latest/userguide/managed-renewal.html) in the AWS Certificate Manager User Guide\.

**Your own certificate**  
When using a certificate from the local file system, Envoy will automatically reload the certificate in response to a move command\. For more information about why Envoy only reloads certificates on move and the symbolic link swap example for Envoy's recommended approach for changing files for a running proxy see [https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/core/config_source.proto#core-configsource](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/core/config_source.proto#core-configsource) and [Updating runtime values via symbolic link swap](https://www.envoyproxy.io/docs/envoy/latest/configuration/operations/runtime#config-runtime-symbolic-link-swap) in the Envoy documentation\.
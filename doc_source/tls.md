# Transport Layer Security \(TLS\)<a name="tls"></a>

In App Mesh, Transport Layer Security \(TLS\) encrypts communication between the Envoy proxies deployed on compute resources that are represented in App Mesh by mesh endpoints, such as [Virtual nodes](virtual_nodes.md) and [Virtual gateways](virtual_gateways.md)\. The proxy negotiates and terminates TLS\. When the proxy is deployed with an application, your application code is not responsible for negotiating a TLS session\. The proxy negotiates TLS on your application's behalf\. 

App Mesh allows you to provide the TLS certificate to the proxy in the following ways:
+ A private certificate from AWS Certificate Manager \(ACM\) that is issued by an AWS Certificate Manager Private Certificate Authority \(ACM PCA\)\.
+ A certificate stored on the local file system of a virtual node that is issued by your own certificate authority \(CA\) 
+ A certificate provided by a Secrets Discovery Service \(SDS\) endpoint over local Unix Domain Socket\.

[Envoy Proxy authorization](proxy-authorization.md) must be enabled for the deployed Envoy proxy represented by a mesh endpoint\. We recommend that when you enable proxy authorization, you restrict access to only the mesh endpoint that you're enabling encryption for\.

## Certificate requirements<a name="virtual-node-tls-prerequisites"></a>

One of the Subject Alternative Names \(SANs\) on the certificate must match specific criteria, depending on how the actual service represented by a mesh endpoint is discovered\. 
+ **DNS** – One of the certificate SANs must match the value provided in the DNS service discovery settings\. For an application with the service discovery name `mesh-endpoint.apps.local`, you can create a certificate matching that name, or a certificate with the wild card `*.apps.local`\.
+ **AWS Cloud Map** – One of the certificate SANs must match the value provided in the AWS Cloud Map service discovery settings using the format `service-name.namespace-name`\. For an application with the AWS Cloud Map service discovery settings of serviceName `mesh-endpoint` and the namespaceName `apps.local`, you can create a certificate matching the name `mesh-endpoint.apps.local`, or a certificate with the wild card `*.apps.local.`

For both discovery mechanisms, if none of the certificate SANs match the DNS service discovery settings, the connection between Envoys fails with the following error message, as seen from the client Envoy\. 

```
TLS error: 268435581:SSL routines:OPENSSL_internal:CERTIFICATE_VERIFY_FAILED
```

## TLS authentication certificates<a name="authentication-certificates"></a>

App Mesh supports multiple sources for certificates when using TLS authentication\.

**ACM PCA**  
The certificate must be stored in ACM in the same Region and AWS account as the mesh endpoint that will use the certificate\. The CA's certificate does not need to be in the same AWS account, but it does still need to be in the same Region as the mesh endpoint\. If you don't have an ACM Private CA, then you must [create one](https://docs.aws.amazon.com/acm-pca/latest/userguide/PcaCreateCa.html) before you can request a certificate from it\. For more information about requesting a certificate from an existing ACM PCA using ACM, see [Request a Private Certificate](https://docs.aws.amazon.com/acm/latest/userguide/gs-acm-request-private.html)\. The certificate cannot be a public certificate\.  
The private CAs that you use for TLS client policies must be root CAs\.  
To configure a virtual node with certificates and CAs from ACM PCA, the principal \(such as a user or role\) that you use to call App Mesh must have the following IAM permissions:   
+ For any certificates that you add to a listener's TLS configuration, the principal must have the `acm:DescribeCertificate` permission\.
+ For any CAs configured on a TLS client policy, the principal must have the `acm-pca:DescribeCertificateAuthority` permission\.
Sharing CAs with other accounts may give those accounts unintended privileges to the CA\. We recommend using resource\-based policies to restrict access to just `acm-pca:DescribeCertificateAuthority` and `acm-pca:GetCertificateAuthorityCertificate` for accounts that do not need to issue certificates from the CA\.
You can add these permissions to an existing IAM policy that is attached to a principal or create a new principal and policy and attach the policy to the principal\. For more information, see [Editing IAM Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-edit.html), [Creating IAM Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_create-console.html), and [Adding IAM Identity Permissions](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-attach-detach.html#add-policies-console)\.  
You pay a monthly fee for the operation of each ACM PCA until you delete it\. You also pay for the private certificates you issue each month and private certificates that you export\. For more information, see [AWS Certificate Manager Pricing](http://aws.amazon.com/certificate-manager/pricing/)\.
When you enable [proxy authorization](proxy-authorization.md) for the Envoy Proxy that a mesh endpoint represents, the IAM role that you use must be assigned the following IAM permissions:  
+ For any certificates configured on a virtual node’s listener, the role must have the `acm:ExportCertificate` permission\.
+ For any CAs configured on a TLS client policy, the role must have the `acm-pca:GetCertificateAuthorityCertificate` permission\.

**File System**  
You can distribute certificates to Envoy using the file system\. You can do this by making the certificate chain and the corresponding private key available on the file path\. That way, these resources are reachable from the Envoy sidecar proxy\. 

**Envoy’s Secret Discovery Service \(SDS\)**  
Envoy fetches secrets like TLS certificates from a specific endpoint through the Secrets Discovery protocol\. For more information about this protocol, see Envoy's [SDS documentation](https://www.envoyproxy.io/docs/envoy/latest/configuration/security/secret)\.  
App Mesh configures the Envoy proxy to use a Unix Domain Socket that's local to the proxy to serve as the Secret Discovery Service \(SDS\) endpoint when SDS serves as the source for your certificates and certificate chains\. You can configure the path to this endpoint by using the `APPMESH_SDS_SOCKET_PATH` environment variable\.  
Local Secrets Discovery Service using Unix Domain Socket is supported on App Mesh Envoy proxy version 1\.15\.1\.0 and later\.  
App Mesh supports V2 SDS protocol using gRPC\.

**Integrating with SPIFFE Runtime Environment \(SPIRE\)**  
You can use any sidecar implementation of the SDS API, including existing toolchains like [SPIFFE Runtime Environment \(SPIRE\)](https://github.com/spiffe/spire)\. SPIRE is designed to enable the deployment of mutual TLS authentication between multiple workloads in distributed systems\. It attests the identity of workloads at runtime\. SPIRE also delivers workload\-specific, short\-lived, and automatically rotating keys and certificates directly to workloads\.  
You should configure the SPIRE Agent as an SDS provider for Envoy\. Allow it to directly supply Envoy with the key material that it needs to provide mutual TLS authentication\. Run SPIRE Agents in sidecars next to Envoy proxies\. The Agent takes care of re\-generating the short\-lived keys and certificates as required\. The Agent attests Envoy and determines which service identities and CA certificates that it should make available to Envoy when Envoy connects to the SDS server exposed by the SPIRE Agent\.  
During this process, service identities and CA certificates are rotated, and updates are streamed back to Envoy\. Envoy immediately applies them to new connections without any interruptions or downtime and without the private keys ever touching the file system\.  
+ AWS Certificate Manager– AWS Certificate Manager Private Certificate Authority \(ACM PCA\) can be used along side SPIRE to serve as an upstream Certificate Authority\. By using a Private Certificate Authority \(PCA\), private keys for the CAs are hardware based and not stored in the memory or on the disk\. This adds an additional layer of security\. For more information, see the [SPIRE documentation](https://github.com/spiffe/spire/blob/main/doc/plugin_server_upstreamauthority_aws_pca.md)\.

## How App Mesh configures Envoys to negotiate TLS<a name="envoy-configuration-tls"></a>

App Mesh uses the mesh endpoint configuration of both the client and server when determining how to configure the communication between Envoys in a mesh\.

**With client policies**  
When a client policy is enforcing the use of TLS, and one of the ports in the client policy matches the port of the server's policy, the client policy is used to configure the TLS validation context of the client\. For example, if a virtual gateway's client policy matches a virtual node's server policy, TLS negotiation will be attempted between the proxies using the settings defined in the virtual gateway's client policy\. If the client policy does not match the port of the server's policy, TLS between the proxies may or may not be negotiated, depending on the server policy's TLS settings\.

**Without client policies**  
If the client has not configured a client policy, or the client policy does not match the port of the server, App Mesh will use the server to determine whether or not to negotiate TLS from the client, and how\. For example, if a virtual gateway has not specified a client policy, and a virtual node has not configured TLS termination, TLS will not be negotiated between the proxies\. If a client has not specified a matching client policy, and a server has been configured with TLS modes `STRICT` or `PERMISSIVE`, the proxies will be configured to negotiate TLS\. Depending on how the certificates have been provided for TLS termination, the following additional behavior applies\.  
+ **ACM\-managed TLS certificates** – When a server has configured TLS termination using an ACM\-managed certificate, App Mesh automatically configures clients to negotiate TLS and validate the certificate against the root CA that the certificate chains up to\.
+ **File\-based TLS certificates** – When a server has configured TLS termination using a certificate from the proxy's local file system, App Mesh automatically configures a client to negotiate TLS, but the certificate of the server is not validated\.

**Subject alternative names**  
You can optionally specify a list of Subject Alternative Names \(SANs\) to trust\. SANs must be in the FQDN or URI format\. If SANs are provided, Envoy verifies that the Subject Alternative Name of the presented certificate matches one of the names on this list\.  
If you don't specify SANs on the terminating mesh endpoint, the Envoy proxy for that node doesn't verify the SAN on a peer client certificate\. If you don't specify SANs on the originating mesh endpoint, the SAN on the certificate provided by the terminating endpoint must match the mesh endpoint service discovery configuration\.  
For more information, see App Mesh [TLS: Certificate requirements](https://docs.aws.amazon.com/app-mesh/latest/userguide/tls.html#virtual-node-tls-prerequisites)\.  
App Mesh doesn't support wildcard DNS SANs\. You need to provide exact names of the endpoints\.

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
When using a certificate from the local file system, Envoy will not automatically reload the certificate when it changes\. You may either restart or redeploy the Envoy process to load a new certificate\. You can also place a newer certificate at a different file path and update the virtual node or gateway configuration with that file path\.

## Configure Amazon ECS workloads to use TLS authentication with AWS App Mesh<a name="mtls-configure-ecs"></a>

You can configure your mesh to use TLS authentication\. Make sure that the certificates are available to Envoy proxy sidecars that you add to your workloads\. You can attach an EBS or EFS volume to your Envoy sidecar, or you can store and retrieve certificates from AWS Secrets Manager\.
+ If you use file\-based certificate distribution, attach an EBS or EFS volume to your Envoy sidecar\. Make sure that the path to the certificate and private key matches the one that is configured in AWS App Mesh\.
+ If you're using SDS\-based distribution, add a sidecar that implements Envoy’s SDS API with access to the certificate\.

**Note**  
SPIRE is not supported on Amazon ECS\.

## Configure Kubernetes workloads to use TLS authentication with AWS App Mesh<a name="mtls-configure-kubernetes"></a>

You can configure the AWS App Mesh Controller for Kubernetes to enable TLS authentication for virtual node and virtual gateway service backends and listeners\. Make sure that the certificates are available to the Envoy proxy sidecars that you add to your workloads\. You can see an example for each distribution type in the [walkthrough](https://docs.aws.amazon.com/app-mesh/latest/userguide/mutual-tls.html#mtls-walkthrough) section of Mutual TLS Authentication\.
+ If you use file\-based certificate distribution, attach an EBS or EFS volume to your Envoy sidecar\. Make sure that the path to the certificate and private key matches the one configured in the controller\. Alternatively, you can use a Kubernetes Secret that is mounted on the file system\.
+ If you’re using SDS\-based distribution, you should setup a node local SDS provider that implements Envoy’s SDS API\. Envoy will reach it over UDS\. To enable SDS based mTLS support in the EKS AppMesh controller, set the `enable-sds` flag to `true` and provide the local SDS provider’s UDS path to the controller via the `sds-uds-path` flag\. If you use helm, you set these as part of your controller installation: 

  ```
  --set sds.enabled=true
  ```

**Note**  
You won't be able to use SPIRE to distribute your certificates if you're using Amazon Elastic Kubernetes Service \(Amazon EKS\) in Fargate mode\.
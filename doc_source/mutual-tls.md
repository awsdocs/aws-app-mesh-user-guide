# App Mesh Preview Channel only – Mutual TLS Authentication<a name="mutual-tls"></a>

Mutual TLS \(Transport Layer Security\) authentication is an optional component of TLS that offers two\-way peer authentication\. Mutual TLS authentication adds a layer of security over TLS and allows your services to verify the client that's making the connection\.

The client in the client\-server relationship also provides an X\.509 certificate during the session negotiation process\. The server uses this certificate to identify and authenticate the client\. This process helps to verify if the certificate is issued by a trusted certificate authority \(CA\) and if the certificate is a valid certificate\. It also uses the Subject Alternative Name \(SAN\) on the certificate to identify the client\. 

You can enable mutual TLS authentication for all the protocols supported by AWS App Mesh\. They are TCP, HTTP/1\.1, HTTP/2, gRPC\.

**Note**  
Using App Mesh, you can configure mutual TLS authentication for communications between Envoy proxies from your services\. However, communications between your applications and Envoy proxies are unencrypted\.

## Mutual TLS authentication certificates<a name="mtls-certificates"></a>

AWS App Mesh supports two possible certificate sources for mutual TLS authentication:
+ **File System–** Certificates from the local file system of the Envoy proxy that's being run\. To distribute certificates to Envoy, you need to provide file paths for the certificate chain and private key to the App Mesh API\.
+ **Envoy’s Secret Discovery Service \(SDS\)–** Bring\-your\-own sidecars that implement SDS and allow certificates to be sent to Envoy\. They include the SPIFFE Runtime Environment \(SPIRE\)\. 

**Important**  
App Mesh doesn't store the certificates or private keys that are used for mutual TLS authentication\. Instead, Envoy stores them in memory\.

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
+ AWS Certificate Manager– AWS Certificate Manager Private Certificate Authority \(ACM PCA\) can be used along side SPIRE to serve as an upstream Certificate Authority\. By using a Private Certificate Authority \(PCA\), private keys for the CAs are hardware based and not stored in the memory or on the disk\. This adds an additional layer of security\. For more information, see the [SPIRE documentation](https://github.com/spiffe/spire/blob/master/doc/plugin_server_upstreamauthority_aws_pca.md)\.

## Configure mesh endpoints<a name="mtls-configure-mesh-endpoints"></a>

Configure mutual TLS authentication for your mesh endpoints, such as virtual nodes or gateways\. These endpoints provide certificates and specify trusted authorities\.

To do this, you need to provision X\.509 certificates for both the client and the server, and explicitly define trusted authority certificates in the validation context of both the TLS termination and TLS origination\.

**Subject alternative names**  
You can optionally specify a list of Subject Alternative Names \(SANs\) to trust\. SANs must be in the FQDN or URI format\. If SANs are provided, Envoy verifies that the Subject Alternative Name of the presented certificate matches one of the names on this list\.  
If you don't specify SANs on the terminating mesh endpoint, the Envoy proxy for that node doesn't verify the SAN on a peer client certificate\. If you don't specify SANs on the originating mesh endpoint, the SAN on the certificate provided by the terminating endpoint must match the mesh endpoint service discovery configuration\.  
For more information, see App Mesh [TLS: Certificate requirements](https://docs.aws.amazon.com/app-mesh/latest/userguide/tls.html#virtual-node-tls-prerequisites)\.  
App Mesh doesn't support wildcard DNS SANs\. You need to provide exact names of the endpoints\.

**Trust inside of a mesh**  
Server\-side certificates are configured in Virtual Node listeners \(TLS termination\), and client\-side certificates are configured in Virtual Nodes service backends \(TLS origination\)\. As an alternative to this configuration, you can define a default client policy for all services backends of a virtual node, and then, if required, you can override this policy for specific backends as needed\. Virtual Gateways can only be configured with a default client that applies to all of its backends\.  
You can configure trust across different meshes by enabling mutual TLS authentication for inbound traffic on the Virtual Gateways for both meshes\.

**Trust outside of a mesh**  
Specify server\-side certificates in the Virtual Gateway listener for TLS termination\. Configure the external service that communicates with your Virtual Gateway to present client\-side certificates\. The certificates should be derived from one of the same certificate authorities \(CAs\) that the server\-side certificates use on the Virtual Gateway listener for TLS origination\.

## Migrate services to mutual TLS authentication<a name="mtls-migrating-services"></a>

Follow these guidelines to maintain connectivity when migrating your existing services within App Mesh to mutual TLS authentication\.

**Migrating services communicating over plaintext**

1. Enable `PERMISSIVE` mode for the TLS configuration on the server endpoint\. This mode allows plain\-text traffic to connect to the endpoint\.

1. Configure mutual TLS authentication on your server, specifying the server certificate, trust chain, and optionally the trusted SANs\.

1. Confirm communication is happening over a TLS connection\.

1. Configure mutual TLS authentication on your clients, specifying the client certificate, trust chain, and optionally the trusted SANs\.

1. Enable `STRICT` mode for the TLS configuration on the server\.

**Migrating services communicating over TLS**

1. Configure the mutual TLS settings on your clients, specifying the client certificate and optionally the trusted SANs\. The client certificate isn't sent to its backend until after the backend server requests it\.

1. Configure the mutual TLS settings on your server, specifying the trust chain and optionally the trusted SANs\. For this, your server requests a client certificate\.

## Verifying mutual TLS authentication<a name="mtls-verification"></a>

You can refer to the [Transport Layer Security: Verify encryption](https://docs.aws.amazon.com/app-mesh/latest/userguide/tls.html#verify-encryption) documentation to see how exactly Envoy emits TLS\-related statistics\. For mutual TLS authentication, you should inspect the following statistics:
+ `ssl.handshake`
+ `ssl.no_certificate`
+ `ssl.fail_verify_no_cert`
+ `ssl.fail_verify_san`

The two following examples of statistics together show that successful TLS connections terminating to the virtual node all originated from a client that provided a certificate\.

```
listener.0.0.0.0_15000.ssl.handshake: 3
```

```
listener.0.0.0.0_15000.ssl.no_certificate: 0
```

The next example of a statistic shows that the connections from a virtual client node \(or gateway\) to a backend virtual node failed\. The Subject Alternative Name \(SAN\) that's presented in the server certificate doesn't match any of the SANs trusted by the client\.

```
cluster.cds_egress_my-mesh_my-backend-node_http_9080.ssl.fail_verify_san: 5
```

## Configure Amazon ECS workloads to use mutual TLS authentication with AWS App Mesh<a name="mtls-configure-ecs"></a>

You can configure your mesh to use mutual TLS authentication\. Make sure that the certificates are available to Envoy proxy sidecars that you add to your workloads\. You can attach an EBS or EFS volume to your Envoy sidecar, or you can store and retrieve certificates from AWS Secrets Manager\.
+ If you use file\-based certificate distribution, attach an EBS or EFS volume to your Envoy sidecar\. Make sure that the path to the certificate and private key matches the one that is configured in AWS App Mesh\.
+ If you're using SDS\-based distribution, add a sidecar that implements Envoy’s SDS API with access to the certificate\.

**Note**  
SPIRE is not supported on Amazon ECS\.

## Configure Kubernetes workloads to use mutual TLS authentication with AWS App Mesh<a name="mtls-configure-kubernetes"></a>

You can configure the AWS App Mesh Controller for Kubernetes to enable mutual TLS authentication for virtual node service backends and listeners\. Make sure that the certificates are available to the Envoy proxy sidecars that you add to your workloads\.
+ If you use file\-based certificate distribution, attach an EBS or EFS volume to your Envoy sidecar\. Make sure that the path to the certificate and private key matches the one configured in the controller\. Alternatively, you can use a Kubernetes Secret that is mounted on the file system\.
+ If you're using SDS\-based distribution, add a sidecar that implements Envoy’s SDS API with access to the certificate\.

**Note**  
You won't be able to use SPIRE to distribute your certificates if you're using Amazon Elastic Kubernetes Service \(Amazon EKS\) in Fargate mode\.

## App Mesh mutual TLS authentication walkthroughs<a name="mtls-walkthrough"></a>
+  [Mutual TLS authentication walkthrough](https://github.com/aws/aws-app-mesh-examples/tree/master/walkthroughs/howto-mutual-tls-file-provided): This walkthrough describes how you can use the App Mesh Preview CLI to build a color app with mutual TLS authentication\. 
+  [Amazon EKS mutual TLS walkthrough](https://github.com/aws/aws-app-mesh-examples/tree/master/walkthroughs/howto-k8s-mtls-sds-based): This walkthrough shows how you can use mutual TLS authentication with Amazon EKS and SPIFFE Runtime Environment \(SPIRE\)\. 
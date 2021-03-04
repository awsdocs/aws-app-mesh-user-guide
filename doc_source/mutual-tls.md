# Mutual TLS Authentication<a name="mutual-tls"></a>

Mutual TLS \(Transport Layer Security\) authentication is an optional component of TLS that offers two\-way peer authentication\. Mutual TLS authentication adds a layer of security over TLS and allows your services to verify the client that's making the connection\.

The client in the client\-server relationship also provides an X\.509 certificate during the session negotiation process\. The server uses this certificate to identify and authenticate the client\. This process helps to verify if the certificate is issued by a trusted certificate authority \(CA\) and if the certificate is a valid certificate\. It also uses the Subject Alternative Name \(SAN\) on the certificate to identify the client\. 

You can enable mutual TLS authentication for all the protocols supported by AWS App Mesh\. They are TCP, HTTP/1\.1, HTTP/2, gRPC\.

**Note**  
Using App Mesh, you can configure mutual TLS authentication for communications between Envoy proxies from your services\. However, communications between your applications and Envoy proxies are unencrypted\.

## Mutual TLS authentication certificates<a name="mtls-certificates"></a>

AWS App Mesh supports two possible certificate sources for mutual TLS authentication\. Client certificates in a TLS Client Policy and server validation in a listener TLS configuration can be sourced from:
+ **File System–** Certificates from the local file system of the Envoy proxy that's being run\. To distribute certificates to Envoy, you need to provide file paths for the certificate chain and private key to the App Mesh API\.
+ **Envoy’s Secret Discovery Service \(SDS\)–** Bring\-your\-own sidecars that implement SDS and allow certificates to be sent to Envoy\. They include the SPIFFE Runtime Environment \(SPIRE\)\. 

**Important**  
App Mesh doesn't store the certificates or private keys that are used for mutual TLS authentication\. Instead, Envoy stores them in memory\.

## Configure mesh endpoints<a name="mtls-configure-mesh-endpoints"></a>

Configure mutual TLS authentication for your mesh endpoints, such as virtual nodes or gateways\. These endpoints provide certificates and specify trusted authorities\.

To do this, you need to provision X\.509 certificates for both the client and the server, and explicitly define trusted authority certificates in the validation context of both the TLS termination and TLS origination\.

**Trust inside of a mesh**  
Server\-side certificates are configured in Virtual Node listeners \(TLS termination\), and client\-side certificates are configured in Virtual Nodes service backends \(TLS origination\)\. As an alternative to this configuration, you can define a default client policy for all services backends of a virtual node, and then, if required, you can override this policy for specific backends as needed\. Virtual Gateways can only be configured with a default client policy that applies to all of its backends\.  
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

## App Mesh mutual TLS authentication walkthroughs<a name="mtls-walkthrough"></a>
+  [Mutual TLS authentication walkthrough](https://github.com/aws/aws-app-mesh-examples/tree/master/walkthroughs/howto-mutual-tls-file-provided): This walkthrough describes how you can use the App Mesh CLI to build a color app with mutual TLS authentication\. 
+  [Amazon EKS mutual TLS SDS\-based walkthrough](https://github.com/aws/aws-app-mesh-examples/tree/master/walkthroughs/howto-k8s-mtls-sds-based): This walkthrough shows how you can use mutual TLS SDS\-based authentication with Amazon EKS and SPIFFE Runtime Environment \(SPIRE\)\. 
+  [Amazon EKS mutual TLS file\-based walkthrough](https://github.com/aws/aws-app-mesh-examples/tree/master/walkthroughs/howto-k8s-mtls-file-based): This walkthrough shows how you can use mutual TLS file\-based authentication with Amazon EKS and SPIFFE Runtime Environment \(SPIRE\)\. 
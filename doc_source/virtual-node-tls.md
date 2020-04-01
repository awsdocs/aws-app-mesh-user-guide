# Transport Layer Security \(TLS\)<a name="virtual-node-tls"></a>

In App Mesh, TLS encrypts communication between the Envoy proxies deployed with your applications that are represented as App Mesh virtual nodes\. Your application code is not responsible for negotiating a TLS session\. The proxy negotiates and terminates TLS on your application's behalf\. 

App Mesh allows you to provide the TLS certificate to the proxy in the following ways:
+ A private certificate from AWS Certificate Manager \(ACM\) that is issued by an AWS Certificate Manager Private Certificate Authority \(ACM PCA\)
+ A certificate stored on the local file system of a virtual node that is issued by your own certificate authority \(CA\) 

[Proxy authorization](proxy-authorization.md) must be enabled for the Envoy proxy deployed with the application represented by the virtual node\. We recommend that when you enable proxy authorization, you restrict access to only the virtual node that you're enabling encryption for\.

## Certificate Requirements<a name="virtual-node-tls-prerequisites"></a>

The Common Name \(CN\) or Subject Alternative Name \(SAN\) of the certificate must match specific criteria, depending on how the actual service represented by a virtual node is discovered\.
+ **DNS** – The certificate CN or one of the SANs must match the value provided in the DNS service discovery settings\. You can also use wild cards such as `*.apps.local` in your certificates\. If the certificate CN or SAN does not match the DNS service discovery settings, the connection between Envoys fails with the following error message, as seen from the downstream \(calling\) Envoy\.

  ```
  SSL error: 268435703:SSL routines:OPENSSL_internal:WRONG_VERSION_NUMBER
  ```
+ **AWS Cloud Map** – The certificate CN and SAN are not considered when negotiating TLS\. You can use any names for your certificates when using AWS Cloud Map, but we recommend that you create a name that’s significant to your virtual node, such as `virtual-node-name.apps.local`\. This name can help you identify the certificate in ACM\.

For additional requirements, select the issuer of the certificate that you're using\.

### ACM PCA<a name="certificate-pca"></a>

The certificate must be stored in ACM in the same Region and account as the virtual node that will use the certificate\. If you don't have an ACM Private CA, then you must [create one](https://docs.aws.amazon.com/acm-pca/latest/userguide/PcaCreateCa.html) before you can request a certificate from it\. For more information about requesting a certificate from an existing ACM PCA using ACM, see [Request a Private Certificate](https://docs.aws.amazon.com/acm/latest/userguide/gs-acm-request-private.html)\. The certificate cannot be a public certificate\.

Your account must have the `acm:DescribeCertificate` and `acm-pca:DescribeCertificateAuthority` IAM permissions for the certificate that you use for TLS\.

**Note**  
You pay a monthly fee for the operation of each ACM PCA until you delete it\. You also pay for the private certificates you issue each month and private certificates that you export\. For more information, see [AWS Certificate Manager Pricing](http://aws.amazon.com/certificate-manager/pricing/)\.

When you enable [proxy authorization](proxy-authorization.md) for the Envoy proxy deployed with the application that a virtual node represents, the IAM role that you use must be assigned the `acm:ExportCertificate` action\. In addition to the actions required for proxy authorization, you must add the `acm-pca:GetCertificateAuthorityCertificate` action to the policy for the CA that you use\.

For a complete, end\-to\-end walk through of deploying a mesh with a sample application using encryption with an ACM certificate, see [Configuring TLS with AWS Certificate Manager](https://github.com/aws/aws-app-mesh-examples/tree/master/walkthroughs/tls-with-acm) on GitHub\.

### Your own CA<a name="certificate-file"></a>

When using a certificate from your own CA, you'll need a public key, a private key, and a certificate chain from the CA\. The files must all be stored in a location accessible to your application code\. For a complete, end\-to\-end walk through of deploying a mesh with a sample application using encryption with local files, see [Configuring TLS with File Provided TLS Certificates](https://github.com/aws/aws-app-mesh-examples/tree/master/walkthroughs/howto-tls-file-provided) on GitHub\.

## Verify encryption<a name="verify-encryption"></a>

Once you've enabled TLS, you can query the Envoy proxy to confirm that communication is encrypted\. The Envoy proxy emits statistics on resources that can help you understand if your TLS communication is working properly\. For example, the Envoy proxy records statistics on the number of successful TLS handshakes it has negotiated for a specified virtual node\. Determine how many successful TLS handshakes there were for a virtual node named `serviceA` with the following command\.

```
curl -s 'http://servicea.apps.local:9901/stats' | grep ssl.handshake
```

In the following example returned output, there were three handshakes for the virtual node, so communication is encrypted\.

```
listener.0.0.0.0_15000.ssl.handshake: 3
```

The Envoy proxy also emits statistics when TLS negotiation is failing\. Determine whether there were TLS errors for a virtual node named `serviceb` with the following command\.

```
$ curl -s 'http://servicea.apps.local:9901/stats' | grep -e "ssl.*\(fail\|error\)"
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
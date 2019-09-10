# TLS Encryption<a name="virtual-node-tls"></a>

\([App Mesh Preview Channel](https://docs.aws.amazon.com//app-mesh/latest/userguide/preview.html) only\) Transport Layer Security \(TLS\) enables you to encrypt communication between virtual nodes\. To learn more about using features available in the Preview Channel, see [App Mesh Preview Channel](preview.md)\. You can also view the [GitHub issue](https://github.com/aws/aws-app-mesh-roadmap/issues/39) that this feature is based on\. 

**Note**  
You must enable [proxy authorization](proxy-authorization.md) for the Envoy proxy in each microservice that each virtual node represents before you can enable TLS encryption for a virtual node\. We recommend that when you enable proxy authorization, you restrict access to only the virtual node that you're enabling TLS for\. The role that you assign the IAM action to that enables proxy authorization must also be assigned the `acm:ExportCertificate` action\.

## Request a Certificate<a name="certificate"></a>

To use TLS encryption, you must have a certificate stored in AWS Certificate Manager\. The AWS Certificate Manager must exist in the same Region as the virtual node that will use the certificate\. When creating a certificate to use with TLS encryption, the Common Name \(CN\) or Subject Alternative Name \(SAN\) of the certificate must match specific criteria, depending on how the actual service represented by a virtual node is discovered\.
+ **DNS** – The certificate Common Name \(CN\) or one of the Subject Alternative Names \(SAN\) must match the value provided in the DNS service discovery settings\. You can also use wildcards, such as `*.mesh.local`, in your certificates\. If the certificate CN or SAN does not match the DNS service discovery settings, the connection between Envoys will fail with the following error, as seen from the downstream \(calling\) Envoy\.

  ```
  SSL error: 268435703:SSL routines:OPENSSL_internal:WRONG_VERSION_NUMBER
  ```
+ **AWS Cloud Map** – The certificate Common Name \(CN\) and Subject Alternative Names \(SAN\) are not considered when negotiating TLS\. You can use any names for your certificates when using AWS Cloud Map, but we recommend that you create a name that’s significant to your virtual node, such as `virtual-node-name.mesh.local`\. This will help you identify the certificate in AWS Certificate Manager\.

The certificate must be a private certificate issued from an AWS Certificate Manager Private Certificate Authority\. If you don't have an ACM Private CA, you must [create one](https://docs.aws.amazon.com/acm-pca/latest/userguide/PcaCreateCa.html) before you can request a certificate from it\. [Request](https://docs.aws.amazon.com/acm/latest/userguide/gs-acm-request-private.html) a private certificate from an existing ACM Private CA using the AWS Certificate Manager\. The certficate cannot be a public certificate\.

**Note**  
You pay a monthly fee for the operation of each AWS Certificate Manager Private Certificate Authority until you delete it\. You also pay for the private certificates you issue each month and private certificates that you export\. For more information, see [AWS Certificate Manager Pricing](https://aws.amazon.com/certificate-manager/pricing/)\.

Note the Amazon Resource Name \(ARN\) of the certificate stored in AWS Certificate Manager for use in a later step\. For additional information about how to view the ARN of an AWS Certificate Manager certificate, see [Describe ACM Certificates](https://docs.aws.amazon.com/acm/latest/userguide/gs-acm-describe.html)\.

## Enable Encryption<a name="enable-encryption"></a>

1. Download the Preview Channel service model with the following command\.

   ```
   curl -o appmesh-preview-channel-service-model.json https://raw.githubusercontent.com/aws/aws-app-mesh-roadmap/master/appmesh-preview/service-model.json
   ```

1. Add the Preview Channel service model to the AWS CLI with the following command\.

   ```
   aws configure add-model \
       --service-name appmesh-preview \
       --service-model file://appmesh-preview-channel-service-model.json
   ```

1. Create a JSON file named `create-virtual-node.json` with a virtual node configuration\. In the following example JSON file, a virtual node is created in a mesh named *app1*\. The virtual node only accepts connections from other virtual nodes that have TLS enabled\.

   ```
   {
      "meshName" : "app1",
      "spec" : {
         "listeners" : [
            {
               "portMapping" : {
                  "port" : 80,
                  "protocol" : "http"
               },
               "tls" : {
                  "mode" : "STRICT",
                  "certificate" : {
                     "acm" : {
                        "certificateArn" : "arn:aws:acm:us-west-2:123456789012:certificate/12345678-1234-1234-1234-123456789012"
                     }
                  }
               }
            }
         ],
         "serviceDiscovery" : {
            "dns" : {
               "hostname" : "serviceBv1.mesh.local"
            }
         }
      },
      "virtualNodeName" : "serviceBv1"
   }
   ```

   Valid values for `mode` are:
   + `STRICT` – Listener only accepts connections with TLS enabled\.
   + `PERMISSIVE` – Listener accepts connections with or without TLS enabled\.
   + `DISABLED` – Listener only accepts connections without TLS\.

   The ARN specified for `certificateArn` must be for a certificate stored in AWS Certificate Manager that meets the requirements specified in [Request a Certificate](#certificate)\. The certificate specified must be stored in an AWS Certificate Manager that exists in the same Region as the virtual node\.

1. Create the virtual node with the following command\.

   ```
   aws appmesh-preview create-virtual-node --cli-input-json file://create-virtual-node.json
   ```

When you have existing virtual nodes, we recommend that you create a new virtual node on which to enable TLS\. Use the previous steps to create a new virtual node that represents the same service as the existing virtual node\. Then gradually shift traffic to the new virtual node using a virtual router and route\. For more information about creating a route and adjusting weights for the transition, see [Routes](routes.md)\. If you update an existing, traffic\-serving virtual node with TLS, there is a chance that the downstream client Envoys will receive TLS validation context before the Envoy for the virtual node that you have updated receives the certificate\. This can cause TLS negotiation errors on the downstream Envoys\.

## Verify Encryption<a name="verify-encryption"></a>

Once you've enabled TLS encryption, you can query the Envoy proxy to confirm that communication is encrypted\. Envoy emits statistics on resources that can help you understand if your TLS communication is working properly\. For example, Envoy records statistics on the number of successful TLS handshakes it has negotiated for a specified virtual node\. The following example shows how to determine how many successful TLS handshakes there were for a virtual node\.

```
curl -s 'http://serviceBv1.mesh.local:9901/stats' | grep ssl.handshake
```

In the example returned output, there were three handshakes for a virtual node named `serviceBv1` in a mesh named `app1`, so communication is encrypted\.

```
listener.0.0.0.0_15000.ssl.handshake: 3
```

Envoy will also emit statistics when TLS negotiation is failing\. The following example shows how to determine whether there were TLS errors for a virtual node\.

```
$ curl -s 'http://serviceBv1.mesh.local:9901/stats' | grep -e "ssl.*\(fail\|error\)"
```

In the example returned output, there were zero errors for several stats\.

```
listener.0.0.0.0_15000.ssl.connection_error: 0
listener.0.0.0.0_15000.ssl.fail_verify_cert_hash: 0
listener.0.0.0.0_15000.ssl.fail_verify_error: 0
listener.0.0.0.0_15000.ssl.fail_verify_no_cert: 0
listener.0.0.0.0_15000.ssl.ssl.fail_verify_san: 0
```

For more information about Envoy TLS\-related statistics, see [Envoy Listener Statistics](https://www.envoyproxy.io/docs/envoy/latest/configuration/listeners/stats)\.
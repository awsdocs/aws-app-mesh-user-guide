# TLS Encryption<a name="virtual-node-tls"></a>

\([App Mesh Preview Channel](https://docs.aws.amazon.com//app-mesh/latest/userguide/preview.html) only\) In App Mesh, traffic encryption works between the Envoy proxies deployed with your applications that are represented as App Mesh virtual nodes\. Your application code is not responsible for negotiating a TLS\-encrypted session\. The proxy negotiates and terminates TLS on your application's behalf\. App Mesh allows you to provide the TLS certificate to the proxy in the following ways:
+ A private certificate from AWS Certificate Manager \(ACM\) that is issued by an AWS Certificate Manager Private Certificate Authority
+ A certificate stored on the local file system of a virtual node that is issued by your own certificate authority \(CA\) 

Your virtual node can specify one or more CAs that it trusts by default when communicating with any backend\. Your virtual node can also specify one or more CAs that it trusts for individual backends\. For example, if `virtualNodeA` attempts to communicate with `virtualNodeB`, then `virtualNodeB` must have a certificate issued by a CA that `virtualNodeA` trusts\.

To learn more about using features available in the Preview Channel, see [App Mesh Preview Channel](preview.md)\. You can also view the [GitHub issue](https://github.com/aws/aws-app-mesh-roadmap/issues/39) that this feature is based on\.

## Prerequisites<a name="virtual-node-tls-prerequisites"></a>

For each virtual node that you want to enable ecnryption for, you must have the following:
+ An existing certificate\. The Common Name \(CN\) or Subject Alternative Name \(SAN\) of the certificate must match specific criteria, depending on how the actual service represented by a virtual node is discovered\.
  + **DNS** – The certificate Common Name \(CN\) or one of the Subject Alternative Names \(SAN\) must match the value provided in the DNS service discovery settings\. You can also use wild cards, such as `*.apps.local`, in your certificates\. If the certificate CN or SAN does not match the DNS service discovery settings, the connection between Envoys fails with the following error message, as seen from the downstream \(calling\) Envoy\.

    ```
    SSL error: 268435703:SSL routines:OPENSSL_internal:WRONG_VERSION_NUMBER
    ```
  + **AWS Cloud Map** – The certificate Common Name \(CN\) and Subject Alternative Names \(SAN\) are not considered when negotiating TLS\. You can use any names for your certificates when using AWS Cloud Map, but we recommend that you create a name that’s significant to your virtual node, such as `virtual-node-name.apps.local`\. This name can help you identify the certificate in AWS Certificate Manager\.
+ [Proxy authorization](proxy-authorization.md) enabled for the Envoy proxy depoyed with the application represented by the virtual node\. We recommend that when you enable proxy authorization, you restrict access to only the virtual node that you're enabling encryption for\.

## Create a Virtual Node<a name="virtual-node-tls-create"></a>

**To create a virtual node with encryption**

1. Add the Preview Channel service model to the AWS CLI with the following command\.

   ```
   aws configure add-model \
       --service-name appmesh-preview \
       --service-model https://raw.githubusercontent.com/aws/aws-app-mesh-roadmap/master/appmesh-preview/service-model.json
   ```

1. If you don't already have a mesh to work with, then create a mesh with the following command\.

   ```
   aws appmesh-preview create-mesh --mesh-name apps --region us-west-2
   ```

1. Select the tab for the way that you'd like to provide the TLS certificate to use for encryption\.

------
#### [ ACM ]

   When you enable [proxy authorization](proxy-authorization.md) for the Envoy proxy deployed with the application that a virtual node represents, the IAM role that you use must be assigned the `acm:ExportCertificate` action\.

   You must have an existing private certificate, issued from an AWS Certificate Manager Private Certificate Authority, stored in ACM\. The certificate must be stored in the same Region and account as the virtual node that will use the certificate\. If you don't have an ACM Private CA, then you must [create one](https://docs.aws.amazon.com/acm-pca/latest/userguide/PcaCreateCa.html) before you can request a certificate from it\. For more information, see [Request](https://docs.aws.amazon.com/acm/latest/userguide/gs-acm-request-private.html) a private certificate from an existing ACM Private CA using the AWS Certificate Manager\. The certificate cannot be a public certificate\.

**Note**  
You pay a monthly fee for the operation of each AWS Certificate Manager Private Certificate Authority until you delete it\. You also pay for the private certificates you issue each month and private certificates that you export\. For more information, see [AWS Certificate Manager Pricing](https://aws.amazon.com/certificate-manager/pricing/)\.

   For a complete, end\-to\-end walk through of deploying a mesh with a sample application using encryption with an ACM certificate, see [Configuring TLS with AWS Certificate Manager](https://github.com/aws/aws-app-mesh-examples/tree/master/walkthroughs/tls-with-acm) on GitHub\.

   Create a file named `create-virtual-node.json` with the following contents\. The `clientPolicy` sections are optional\. If you specify either section, then you must specify one or more values for `certificateAuthorityArns`\.

**Note**  
Your account must have the `acm:DescribeCertificate` and `acm-pca:DescribeCertificateAuthority` IAM permissions for the certificate that you specify in the following text\.

   ```
   {
      "meshName" : "apps",
      "spec" : {
         "backendDefaults" : {
            "clientPolicy" : {
               "tls" : {
                  "validation" : {
                     "trust" : {
                        "acm" : {
                           "certificateAuthorityArns" : [
                              "arn:aws:acm:us-west-2:111122223333:certificate-authority/11d11d11-2222-3d33-44a4-5aedbEXAMPLE"
                           ]
                        }
                     }
                  }
               }
            }
         },
         "backends" : [
            {
               "virtualService" : {
                  "clientPolicy" : {
                     "tls" : {
                        "validation" : {
                           "trust" : {
                              "acm" : {
                                 "certificateAuthorityArns" : [
                                    "arn:aws:acm:us-west-2:111122223333:certificate-authority/17d7df97-2564-4d44-93a8-3aed6EXAMPLE"
                                 ]
                              }
                           }
                        }
                     }
                  },
                  "virtualServiceName" : "serviceb.apps.local"
               }
            }
         ],
         "listeners" : [
            {
               "portMapping" : {
                  "port" : 80,
                  "protocol" : "http2"
               },
               "tls" : {
                  "certificate" : {
                     "acm" : {
                        "certificateArn" : "arn:aws:acm:us-west-2:111122223333:certificate/12345678-1234-1234-1234-12345EXAMPLE"
                     }
                  },
                  "mode" : "STRICT"
               }
            }
         ],
         "serviceDiscovery" : {
            "dns" : {
               "hostname" : "servicea.apps.local"
            }
         }
      },
      "virtualNodeName" : "serviceA"
   }
   ```

------
#### [ Local files ]

   You'll need a public key, a private key, and a certificate chain from a certificate authority\. The files must all be stored in a location accessible to your application code\. For a complete, end\-to\-end walk through of deploying a mesh with a sample application using encryption with local files, see [Configuring TLS with File Provided TLS Certificates](https://github.com/aws/aws-app-mesh-examples/tree/master/walkthroughs/howto-tls-file-provided) on GitHub\.

   Create a file named `create-virtual-node.json` with the following contents\. The `clientPolicy` sections are optional\. If you specify either section, then you must specify one or more values for `certificateChain`\.

   ```
   {
      "meshName" : "apps",
      "spec" : {
         "backendDefaults" : {
            "clientPolicy" : {
               "tls" : {
                  "validation" : {
                     "trust" : {
                        "file" : {
                           "certificateChain" : "/path/to/cert-chain.pem"
                        }
                     }
                  }
               }
            }
         },
         "backends" : [
            {
               "virtualService" : {
                  "clientPolicy" : {
                     "tls" : {
                        "validation" : {
                           "trust" : {
                              "file" : {
                                 "certificateChain" : "/path/to/cert-chain2.pem"
                              }
                           }
                        }
                     }
                  },
                  "virtualServiceName" : "serviceb.apps.local"
               }
            }
         ],
         "listeners" : [
            {
               "portMapping" : {
                  "port" : 80,
                  "protocol" : "http2"
               },
               "tls" : {
                  "certificate" : {
                     "file" : {
                        "certificateChain" : "/path/to/cert-chain.pem",
                        "privateKey" : "/path/to/private-key.pem"
                     }
                  },
                  "mode" : "STRICT"
               }
            }
         ],
         "serviceDiscovery" : {
            "dns" : {
               "hostname" : "servicea.apps.local"
            }
         }
      },
      "virtualNodeName" : "serviceA"
   }
   ```

------

   Valid values for `mode` are:
   + `STRICT` – Listener only accepts connections with TLS enabled\.
   + `PERMISSIVE` – Listener accepts connections with or without TLS enabled\.
   + `DISABLED` – Listener only accepts connections without TLS\.

1. Create the virtual node with the following command\.

   ```
   aws appmesh-preview create-virtual-node --region us-west-2 --cli-input-json file://create-virtual-node.json
   ```

When you have existing virtual nodes, we recommend that you create a new virtual node on which to enable TLS\. Use the previous steps to create a new virtual node that represents the same service as the existing virtual node\. Then gradually shift traffic to the new virtual node using a virtual router and route\. For more information about creating a route and adjusting weights for the transition, see [Routes](routes.md)\. If you update an existing, traffic\-serving virtual node with TLS, there is a chance that the downstream client Envoys will receive TLS validation context before the Envoy for the virtual node that you have updated receives the certificate\. This can cause TLS negotiation errors on the downstream Envoys\.

## Verify Encryption<a name="verify-encryption"></a>

Once you've enabled TLS encryption, you can query the Envoy proxy to confirm that communication is encrypted\. Envoy emits statistics on resources that can help you understand if your TLS communication is working properly\. For example, Envoy records statistics on the number of successful TLS handshakes it has negotiated for a specified virtual node\. Determine how many successful TLS handshakes there were for a virtual node named `serviceA` with the following command\.

```
curl -s 'http://servicea.apps.local:9901/stats' | grep ssl.handshake
```

In the following example returned output, there were three handshakes for the virtual node, so communication is encrypted\.

```
listener.0.0.0.0_15000.ssl.handshake: 3
```

Envoy will also emit statistics when TLS negotiation is failing\. Determine whether there were TLS errors for a virtual node named `serviceb` with the following command\.

```
$ curl -s 'http://servicea.apps.local:9901/stats' | grep -e "ssl.*\(fail\|error\)"
```

In the example returned output, there were zero errors for several stats, so the TLS negotiation succeeded\.

```
listener.0.0.0.0_15000.ssl.connection_error: 0
listener.0.0.0.0_15000.ssl.fail_verify_cert_hash: 0
listener.0.0.0.0_15000.ssl.fail_verify_error: 0
listener.0.0.0.0_15000.ssl.fail_verify_no_cert: 0
listener.0.0.0.0_15000.ssl.ssl.fail_verify_san: 0
```

For more information about Envoy TLS\-related statistics, see [Envoy Listener Statistics](https://www.envoyproxy.io/docs/envoy/latest/configuration/listeners/stats)\.
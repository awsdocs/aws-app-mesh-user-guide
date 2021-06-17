# Infrastructure security in App Mesh<a name="infrastructure-security"></a>

As a managed service, App Mesh is protected by the AWS global network security procedures that are described in the [Amazon Web Services: Overview of Security Processes](https://d0.awsstatic.com/whitepapers/Security/AWS_Security_Whitepaper.pdf) paper\.

You use AWS published API calls to access App Mesh through the network\. Clients must support Transport Layer Security \(TLS\) 1\.0 or later\. We recommend TLS 1\.2 or later\. Clients must also support cipher suites with perfect forward secrecy \(PFS\) such as Ephemeral Diffie\-Hellman \(DHE\) or Elliptic Curve Ephemeral Diffie\-Hellman \(ECDHE\)\. Most modern systems such as Java 7 and later support these modes\.

Additionally, requests must be signed by using an access key ID and a secret access key that is associated with an IAM principal\. Or you can use the [AWS Security Token Service](https://docs.aws.amazon.com/STS/latest/APIReference/Welcome.html) \(AWS STS\) to generate temporary security credentials to sign requests\.

The Envoy proxy is deployed with a microservice application that is running on an AWS compute service\. Each of the compute services are deployed within an Amazon VPC\.

You can improve the security posture of your VPC by configuring App Mesh to use an interface VPC endpoint\. For more information, see [App Mesh Interface VPC Endpoints \(AWS PrivateLink\)](vpc-endpoints.md)\.
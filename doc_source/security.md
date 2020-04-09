# Security in AWS App Mesh<a name="security"></a>

Cloud security at AWS is the highest priority\. As an AWS customer, you benefit from a data center and network architecture that is built to meet the requirements of the most security\-sensitive organizations\.

Security is a shared responsibility between AWS and you\. The [shared responsibility model](http://aws.amazon.com/compliance/shared-responsibility-model/) describes this as security *of* the cloud and security *in* the cloud:
+ **Security of the cloud** – AWS is responsible for protecting the infrastructure that runs AWS services in the AWS Cloud\. AWS also provides you with services that you can use securely\. Third\-party auditors regularly test and verify the effectiveness of our security as part of the [AWS compliance programs](http://aws.amazon.com/compliance/programs/)\. To learn about the compliance programs that apply to AWS App Mesh, see [AWS Services in Scope by Compliance Program](http://aws.amazon.com/compliance/services-in-scope/)\. App Mesh is responsible for securely delivering configuration to local proxies, including secrets such as TLS certificate private keys\. 
+ **Security in the cloud** – Your responsibility is determined by the AWS service that you use\. You are also responsible for other factors including:
  + The sensitivity of your data, your company’s requirements, and applicable laws and regulations\.
  + The security configuration of the App Mesh data plane, including the configuration of the security groups that allow traffic to pass between services within your VPC\.
  + The configuration of your compute resources associated with App Mesh\.
  + The IAM policies associated with your compute resources and what configuration they are allowed to retrieve from the App Mesh control plane\.

This documentation helps you understand how to apply the shared responsibility model when using App Mesh\. The following topics show you how to configure App Mesh to meet your security and compliance objectives\. You also learn how to use other AWS services that help you to monitor and secure your App Mesh resources\. 

**Topics**
+ [Data protection in AWS App Mesh](data-protection.md)
+ [Identity and Access Management for AWS App Mesh](security-iam.md)
+ [Logging App Mesh API calls with AWS CloudTrail](logging-using-cloudtrail.md)
+ [Compliance validation for AWS App Mesh](compliance.md)
+ [Resilience in AWS App Mesh](disaster-recovery-resiliency.md)
+ [Infrastructure security in App Mesh](infrastructure-security.md)
+ [Configuration and vulnerability analysis in AWS App Mesh](configuration-vulnerability-analysis.md)
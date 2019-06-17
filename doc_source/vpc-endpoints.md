# App Mesh Interface VPC Endpoints \(AWS PrivateLink\)<a name="vpc-endpoints"></a>

You can improve the security posture of your Amazon VPC by configuring App Mesh to use an interface VPC endpoint\. Interface endpoints are powered by AWS PrivateLink, a technology that enables you to privately access App Mesh APIs by using private IP addresses\. PrivateLink restricts all network traffic between your Amazon VPC and App Mesh to the Amazon network\.

You're not required to configure PrivateLink, but we recommend it\. For more information about PrivateLink and interface VPC endpoints, see [Accessing Services Through AWS PrivateLink](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html#what-is-privatelink)\.

## Considerations for App Mesh Interface VPC Endpoints<a name="app-mesh-vpc-endpoint-considerations"></a>

Before you set up interface VPC endpoints for App Mesh, be aware of the following considerations:
+ If your Amazon VPC doesn't have an internet gateway and your tasks use the `awslogs` log driver to send log information to CloudWatch Logs, you must create an interface VPC endpoint for CloudWatch Logs\. For more information, see [Using CloudWatch Logs with Interface VPC Endpoints](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/cloudwatch-logs-and-interface-VPC.html) in the *Amazon CloudWatch Logs User Guide*\.
+ VPC endpoints don't support AWS cross\-Region requests\. Ensure that you create your endpoint in the same Region where you plan to issue your API calls to App Mesh\.
+ VPC endpoints only support Amazon\-provided DNS through Amazon Route 53\. If you want to use your own DNS, you can use conditional DNS forwarding\. For more information, see [DHCP Options Sets](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_DHCP_Options.html) in the *Amazon VPC User Guide*\.
+ The security group attached to the VPC endpoint must allow incoming connections on port 443 from the private subnet of the Amazon VPC\.
+ Controlling access to App Mesh by attaching an endpoint policy to the VPC endpoint isn't supported\. By default, full access to the service will be allowed through the endpoint\. For more information, see [Controlling Access to Services with VPC Endpoints](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-endpoints-access.html) in the *Amazon VPC User Guide*\.

For additional considerations and limitations, see [Interface Endpoint Availability Zone Considerations](https://docs.aws.amazon.com/vpc/latest/userguide/vpce-interface.html#vpce-interface-availability-zones) and [Interface Endpoint Properties and Limitations](https://docs.aws.amazon.com/vpc/latest/userguide/vpce-interface.html#vpce-interface-limitations)\.

## Create the Interface VPC Endpoint for App Mesh<a name="app-mesh-setting-up-vpc-create"></a>

To create the interface VPC endpoint for the App Mesh service, use the [Creating an Interface Endpoint](https://docs.aws.amazon.com/vpc/latest/userguide/vpce-interface.html#create-interface-endpoint) procedure in the *Amazon VPC User Guide*\. Specify `com.amazonaws.region.appmesh-envoy-management` for the service name\.

**Note**  
*region* represents the Region identifier for an AWS Region supported by App Mesh, such as `us-east-2` for the US East \(Ohio\) Region\.

Though you can define an interface VPC endpoint for App Mesh in any Region where App Mesh is supported, you may not be able to define an endpoint for all Availability Zones in each Region\. To find out which Availability Zones are supported with interface VPC endpoints in a Region, use the [describe\-vpc\-endpoint\-services ](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-vpc-endpoint-services.html)command or use the AWS Management Console\. For example, the following command returns the availability zones to which you can deploy an App Mesh interface VPC endpoint within the US East \(Ohio\) Region: 

```
aws --region us-east-2 ec2 describe-vpc-endpoint-services --query 'ServiceDetails[?ServiceName==`com.amazonaws.us-east-2.appmesh-envoy-management`].AvailabilityZones[]'
```
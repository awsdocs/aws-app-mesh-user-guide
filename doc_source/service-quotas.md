# App Mesh Service Quotas<a name="service-quotas"></a>

AWS App Mesh has integrated with Service Quotas, an AWS service that enables you to view and manage your quotas from a central location\. Service quotas are also referred to as limits\. For more information, see [What Is Service Quotas?](https://docs.aws.amazon.com/servicequotas/latest/userguide/intro.html) in the *Service Quotas User Guide*\.

## View AWS App Mesh service quotas<a name="view-service-quotas"></a>
To view App Mesh service quotas in the AWS Management Console:

1. Open the Service Quotas console at [https://console\.aws\.amazon\.com/servicequotas/](https://console.aws.amazon.com/servicequotas/)\.

1. In the navigation pane, choose **AWS services**\.

1. From the **AWS services** list, search for and select **AWS App Mesh**\.

   In the **Service quotas** list, you can see the service quota name, applied value \(if it is available\), AWS default quota, and whether the quota value is adjustable\.

1. To view additional information about a service quota, such as the description, choose the quota name\.

## Request an AWS App Mesh service quota increase<a name="increase-service-quota"></a>
To request a quota increase, see [Requesting a Quota Increase](https://docs.aws.amazon.com/servicequotas/latest/userguide/request-increase.html) in the *Service Quotas User Guide*\.

## Default AWS App Mesh service quotas<a name="default-service-quotas"></a>

The following table provides the default AWS App Mesh quotas for an AWS account that can be changed\.

| Resource | Default Quota | 
| --- | --- | 
| Maximum number of meshes \(per region, per account\) | 15 | 
| Maximum number of virtual services per mesh | 200 | 
| Maximum number of virtual nodes per mesh | 200 | 
| Maximum number of backends per virtual node | 50 | 
| Maximum number of connected Envoys per virtual node | 10 | 
| Maximum number of virtual routers per mesh | 200 | 
| Maximum number of routes per virtual router | 50 | 

The following table provides quotas for AWS App Mesh that cannot be changed\.

| Resource | Default Quota | 
| --- | --- | 
| Maximum number of weighted targets per route | 10 | 

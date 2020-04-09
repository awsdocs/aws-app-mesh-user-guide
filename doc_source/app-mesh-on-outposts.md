# App Mesh on AWS Outposts<a name="app-mesh-on-outposts"></a>

AWS Outposts enables native AWS services, infrastructure, and operating models in on\-premises facilities\. In AWS Outposts environments, you can use the same AWS APIs, tools, and infrastructure that you use in the AWS Cloud\. App Mesh on AWS Outposts is ideal for low\-latency workloads that need to be run in close proximity to on\-premises data and applications\. For more information about AWS Outposts, see the [AWS Outposts User Guide](https://docs.aws.amazon.com/outposts/latest/userguide/)\.

## Prerequisites<a name="app-mesh-outposts-prereq"></a>

 The following are the prerequisites for using App Mesh on AWS Outposts:
+ You must have installed and configured an Outpost in your on\-premises data center\.
+ You must have a reliable network connection between your Outpost and its AWS Region\.
+ The AWS Region for the Outpost must support AWS App Mesh\. For a list of supported Regions, see [AWS App Mesh Endpoints and Quotas](https://docs.aws.amazon.com/general/latest/gr/appmesh.html) in the *AWS General Reference*\.

## Limitations<a name="app-mesh-outposts-limit"></a>

The following are the limitations of using App Mesh on AWS Outposts:
+ AWS Identity and Access Management, Application Load Balancer, Network Load Balancer, Classic Load Balancer, and Amazon Route 53 run in the AWS Region, not on Outposts\. This will increase latencies between these services and the containers\.

## Network connectivity considerations<a name="app-mesh-outposts-considerations"></a>

The following are network connectivity considerations for Amazon EKS AWS Outposts:
+ If network connectivity between your Outpost and its AWS Region is lost, the App Mesh Envoy proxies will continue to run\. However you will not be able to modify your service mesh until connectivity is restored\.
+ We recommend that you provide reliable, highly available, and low\-latency connectivity between your Outpost and its AWS Region\.

## Creating an App Mesh Envoy proxy on an Outpost<a name="app-mesh-outposts-create"></a>

An Outpost is an extension of an AWS Region, and you can extend an Amazon VPC in an account to span multiple Availability Zones and any associated Outpost locations\. When you configure your Outpost, you associate a subnet with it to extend your Regional VPC environment to your on\-premises facility\. Instances on an Outpost appear as part of your Regional VPC, similar to an Availability Zone with associated subnets\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/app-mesh/latest/userguide/images/network-components.png)

 To create an App Mesh Envoy proxy on an Outpost, add the App Mesh Envoy container image to the Amazon ECS task or Amazon EKS pod running on an Outpost\. For more information, see [Amazon Elastic Container Service on AWS Outposts](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-on-outposts.html) in the *Amazon Elastic Container Service Developer Guide*and [Amazon Elastic Kubernetes Service on AWS Outposts](https://docs.aws.amazon.com/eks/latest/userguide/eks-on-outposts.html) in the **Amazon EKS User Guide**\.
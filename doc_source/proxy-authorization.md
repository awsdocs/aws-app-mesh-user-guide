# Proxy Authorization<a name="proxy-authorization"></a>

Proxy authorization authorizes the [Envoy](envoy.md) proxy to read the configuration of one or more virtual nodes from the App Mesh Envoy Management Service\. An example of a configuration that the proxy can read from a virtual node is the private certificate of a virtual node that has [Transport Layer Security \(TLS\)](virtual-node-tls.md) enabled\. Proxy authorization requires that the `appmesh:StreamAggregatedResources` permission is specified in an IAM policy\. The policy must be attached to an IAM role, and that IAM role must be attached to the appropriate compute service resource that you host your microservice on\.

**Note**  
Effective September 3, 2019, the Envoy proxy will be unable to read the configuration of any virtual node in a mesh unless the `appmesh:StreamAggregatedResources` permission is attached to the appropriate resource in the compute service that you host your microservice on\.

## Create IAM Policy<a name="create-iam-policy"></a>

If you want all virtual nodes in a service mesh to be able to read the configuration for all virtual nodes, skip to [Create IAM role](#create-iam-role)\. If you want to limit the virtual nodes that configuration can be read from by an Envoy proxy, then you need to create one or more IAM policies\. Limiting the virtual nodes that configuration can be read from is recommended\. Add the `appmesh:StreamAggregatedResources` permission to the IAM policy and limit the virtual nodes for which configuration can be read\.

The following example policy allows the configuration of the virtual nodes named `serviceBv1` and `serviceBv2` to be read in a service mesh\. Configuration can't be read for any other virtual nodes defined in the service mesh\. You might name this example policy `mesh-read-vn-serviceB`\. For more information about creating or editing an IAM policy, see [Creating IAM Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_create.html) and [Edit IAM Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-edit.html)\.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "appmesh:StreamAggregatedResources",
            "Resource": [
                "arn:aws:appmesh:us-east-1:123456789012:mesh/app1/virtualNode/serviceBv1",
                "arn:aws:appmesh:us-east-1:123456789012:mesh/app1/virtualNode/serviceBv2"
            ]
        }
    ]
}
```

If you want to create a custom policy to restrict access to specific virtual nodes, you can create multiple custom policies, each restricting access to different virtual nodes, and attach each IAM policy to a different IAM role\. 

## Create IAM role<a name="create-iam-role"></a>

Create a service IAM role for the compute service that you host your microservice on:
+ **Amazon EKS** – If you deployed your worker nodes using either of the Amazon EKS [Getting Started](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html) guides, you don't need to create a role, since an IAM role for the Amazon EC2 service was automatically created during deployment\. If you deployed Amazon EKS with `eksctl`, the role name contains `eksctl`\. If you deployed with the AWS CloudFormation template, the role name contains `NodeInstanceRole`\.
+ **Amazon ECS** – Select the **Elastic Container Service Task** use case when creating your IAM role\. 
+ **Amazon EC2** – Select the **EC2** use case when creating your IAM role\. This applies whether you host your microservices directly on an Amazon EC2 instance or on Kubernetes running on an instance\.

For more information about how to create a service IAM role, see [Creating a Role for an AWS Service](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-service.html#roles-creatingrole-service-console)\.

## Attach IAM Policy<a name="attach-iam-policy"></a>

Attach the custom policy that you created in a previous step or the `[AWSAppMeshEnvoyAccess](https://console.aws.amazon.com/iam/home#/policies/arn:aws:iam::aws:policy/AWSAppMeshEnvoyAccess%24jsonEditor)` managed IAM policy to the IAM role for the resources that you host your microservice on\. For more information about assigning a custom or managed IAM policy to an IAM role, see [Adding IAM Identity Permissions](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-attach-detach.html#add-policies-console)\. 

## Attach IAM Role<a name="attach-role"></a>

Attach the IAM role to the appropriate resource for the compute service that you host your microservice on:
+ **Amazon EKS** – The Envoy proxy runs in a pod\. An IAM role can't be attached to a pod natively\. You attach an IAM role to each worker node in the cluster\. If you deployed an Amazon EKS cluster using either the AWS CloudFormation template or `eksctl`, an IAM EC2 service role was created and attached to each worker node in a nodegroup for you\.
**Note**  
Since the same IAM role is attached to all worker nodes in a nodegroup, the Envoy proxy in any pod on any worker node is able to read the configuration for all virtual nodes in the service mesh\.
+ **Amazon ECS** – Assign an Amazon ECS Task Role to the task definition that includes the Envoy proxy\. The task can be deployed with the EC2 or Fargate launch type\. For more information about how to create an Amazon ECS Task Role and assign it to a task, see [Specifying an IAM Role for your Tasks](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-iam-roles.html#create_task_iam_policy_and_role)\.
+ **Amazon EC2** – The role must be attached to the EC2 instance that hosts the Envoy proxy\. For more information about how to assign a role to an Amazon EC2 instance, see [I’ve created an IAM role, and now I want to assign it to an EC2 instance](https://aws.amazon.com/premiumsupport/knowledge-center/assign-iam-role-ec2-instance/)\.

## Confirm permission<a name="confirm-permission"></a>

Confirm that the `appmesh:StreamAggregatedResources` permission is assigned to the compute service resource that you host your microservice on by selecting one of the compute service names\.

------
#### [ Amazon EKS ]

1. From the Amazon EC2 console, select **Instances** in the left navigation\.

1. Select one of your worker node instances\.

1. In the **Description** tab, select the link of the IAM role name that is to the right of **IAM role**\. If an IAM role isn't listed, then you need to [create an IAM role](#create-iam-role)\.

1. In the **Summary** page, on the **Permissions** tab, confirm that either the custom policy you created previously, or the `[AWSAppMeshEnvoyAccess](https://console.aws.amazon.com/iam/home#/policies/arn:aws:iam::aws:policy/AWSAppMeshEnvoyAccess%24jsonEditor)` managed policy is listed\. If neither policy is attached, [attach the IAM policy](#attach-iam-policy) to the IAM role\. If you want to attach a custom IAM policy but don't have one, then you need to [create the custom IAM policy](#create-iam-policy)\.

1. If a custom IAM policy is attached, select the policy and confirm that it contains `"Action": "appmesh:StreamAggregatedResources"`\. If it does not, then you need to add that permission to your custom IAM policy\. You can also confirm that the appropriate Amazon Resource Name \(ARN\) for a specific virtual node is listed\. If no ARNs are listed, then you can edit the policy to add, remove, or change the listed ARNs\. For more information, see [Edit IAM Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-edit.html) and [Create IAM Policy](#create-iam-policy)\.

1. Repeat the previous steps for each of your worker nodes\.

------
#### [ Amazon ECS ]

1. From the Amazon ECS console, choose **Task Definitions**\.

1. Select your Amazon ECS task\.

1. On the **Task Definition Name** page, select your task definition\.

1. On the `Task Definition` page, select the link of the IAM role name that is to the right of **Task Role**\. If an IAM role isn't listed, then you need to [create an IAM role](#create-iam-role) and attach it to your task by [updating your task definition\.](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/update-task-definition.html)

1. In the **Summary** page, on the **Permissions** tab, confirm that either the custom policy you created previously, or the `[AWSAppMeshEnvoyAccess](https://console.aws.amazon.com/iam/home#/policies/arn:aws:iam::aws:policy/AWSAppMeshEnvoyAccess%24jsonEditor)` managed policy is listed\. If neither policy is attached, [attach the IAM policy](#attach-iam-policy) to the IAM role\. If you want to attach a custom IAM policy but don't have one, then you need to [create the custom IAM policy](#create-iam-policy)\.

1. If a custom IAM policy is attached, select the policy and confirm that it contains `"Action": "appmesh:StreamAggregatedResources"`\. If it does not, then you need to add that permission to your custom IAM policy\. You can also confirm that the appropriate Amazon Resource Name \(ARN\) for a specific virtual node is listed\. If no ARNs are listed, then you can edit the policy to add, remove, or change the listed ARNs\. For more information, see [Edit IAM Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-edit.html) and [Create IAM Policy](#create-iam-policy)\.

1. Repeat the previous steps for each task definition that contains the Envoy proxy\.

------
#### [ Amazon EC2 ]

1. From the Amazon EC2 console, select **Instances** in the left navigation\.

1. Select one of your instances that hosts the Envoy proxy\.

1. In the **Description** tab, select the link of the IAM role name that is to the right of **IAM role**\. If an IAM role isn't listed, then you need to [create an IAM role](#create-iam-role)\.

1. In the **Summary** page, on the **Permissions** tab, confirm that either the custom policy you created previously, or the `[AWSAppMeshEnvoyAccess](https://console.aws.amazon.com/iam/home#/policies/arn:aws:iam::aws:policy/AWSAppMeshEnvoyAccess%24jsonEditor)` managed policy is listed\. If neither policy is attached, [attach the IAM policy](#attach-iam-policy) to the IAM role\. If you want to attach a custom IAM policy but don't have one, then you need to [create the custom IAM policy](#create-iam-policy)\.

1. If a custom IAM policy is attached, select the policy and confirm that it contains `"Action": "appmesh:StreamAggregatedResources"`\. If it does not, then you need to add that permission to your custom IAM policy\. You can also confirm that the appropriate Amazon Resource Name \(ARN\) for a specific virtual node is listed\. If no ARNs are listed, then you can edit the policy to add, remove, or change the listed ARNs\. For more information, see [Edit IAM Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-edit.html) and [Create IAM Policy](#create-iam-policy)\.

1. Repeat the previous steps for each instance that you host the Envoy proxy on\.

------
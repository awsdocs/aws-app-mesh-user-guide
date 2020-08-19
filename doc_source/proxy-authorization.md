# Proxy authorization<a name="proxy-authorization"></a>

Proxy authorization authorizes the [Envoy](envoy.md) proxy running within an Amazon ECS task, in a Kubernetes pod running on Amazon EKS, or running on an Amazon EC2 instance to read the configuration of one or more mesh endpoints from the App Mesh Envoy Management Service\. Proxy authorization is required for virtual nodes that use [Transport Layer Security \(TLS\)](tls.md) and for virtual gateways \(with or without TLS\), but will eventually be required for all App Mesh capabilities\. Until proxy authorization is required for all App Mesh capabilities, it is only required if specifically mentioned in the documentation for the capability\. It is recommended to enable proxy authorization for all virtual nodes, even if they don't use TLS, to have a secure and consistent experience using IAM for authorization to specific resources\. Proxy authorization requires that the `appmesh:StreamAggregatedResources` permission is specified in an IAM policy\. The policy must be attached to an IAM role, and that IAM role must be attached to the compute resource that you host the proxy on\.

## Create IAM policy<a name="create-iam-policy"></a>

If you want all mesh endpoints in a service mesh to be able to read the configuration for all mesh endpoints, then skip to [Create IAM role](#create-iam-role)\. If you want to limit the mesh endpoints that configuration can be read from by individual mesh endpoints, then you need to create one or more IAM policies\. Limiting the mesh endpoints that configuration can be read from to only the Envoy proxy running on specific compute resources is recommended\. Create an IAM policy and add the `appmesh:StreamAggregatedResources` permission to the policy\. The following example policy allows the configuration of the virtual nodes named `serviceBv1` and `serviceBv2` to be read in a service mesh\. Configuration can't be read for any other virtual nodes defined in the service mesh\. For more information about creating or editing an IAM policy, see [Creating IAM Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_create.html) and [Edit IAM Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-edit.html)\.

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

You can create multiple policies, with each policy restricting access to different mesh endpoints\. 

## Create IAM role<a name="create-iam-role"></a>

If you want all mesh endpoints in a service mesh to be able to read the configuration for all mesh endpoints, then you only need to create one IAM role\. If you want to limit the mesh endpoints that configuration can be read from by individual mesh endpoints, then you need to create a role for each policy that you created in the previous step\. Complete the instructions for the compute resource that the proxy runs on\.
+ **Amazon EKS** – If you want to use a singe role, then you can use the existing role that was created and assigned to the worker nodes when you created your cluster\. To use multiple roles, your cluster must meet the requirements defined in [Enabling IAM Roles for Service Accounts on your Cluster](https://docs.aws.amazon.com/eks/latest/userguide/enable-iam-roles-for-service-accounts.html)\. Create the IAM roles and associate the roles with Kubernetes service accounts\. For more information, see [Creating an IAM Role and Policy for your Service Account](https://docs.aws.amazon.com/eks/latest/userguide/create-service-account-iam-policy-and-role.html) and [Specifying an IAM Role for your Service Account](https://docs.aws.amazon.com/eks/latest/userguide/specify-service-account-role.html)\.
+ **Amazon ECS** – Select **AWS service,** select **Elastic Container Service**, and then select the **Elastic Container Service Task** use case when creating your IAM role\.
+ **Amazon EC2** – Select **AWS service,** select **EC2**, and then select the **EC2** use case when creating your IAM role\. This applies whether you host the proxy directly on an Amazon EC2 instance or on Kubernetes running on an instance\.

For more information about how to create an IAM role, see [Creating a Role for an AWS Service](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-service.html#roles-creatingrole-service-console)\.

## Attach IAM policy<a name="attach-iam-policy"></a>

If you want all mesh endpoints in a service mesh to be able to read the configuration for all mesh endpoints, then attach the `[AWSAppMeshEnvoyAccess](https://console.aws.amazon.com/iam/home#/policies/arn:aws:iam::aws:policy/AWSAppMeshEnvoyAccess%24jsonEditor)` managed IAM policy to the IAM role that you created in a previous step\. If you want to limit the mesh endpoints that configuration can be read from by individual mesh endpoints, then attach each policy that you created to each role that you created\. For more information about attaching a custom or managed IAM policy to an IAM role, see [Adding IAM Identity Permissions](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-attach-detach.html#add-policies-console)\. 

## Attach IAM role<a name="attach-role"></a>

Attach each IAM role to the appropriate compute resource:
+ **Amazon EKS** – If you attached the policy to the role attached to your worker nodes, you can skip this step\. If you created separate roles, then assign each role to a separate Kubernetes service account, and assign each service account to an individual Kubernetes pod deployment spec that includes the Envoy proxy\. For more information, see [Specifying an IAM Role for your Service Account](https://docs.aws.amazon.com/eks/latest/userguide/specify-service-account-role.html) in the *Amazon EKS User Guide* and [Configure Service Accounts for Pods](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/) in the Kubernetes documentation\.
+ **Amazon ECS** – Attach an Amazon ECS Task Role to the task definition that includes the Envoy proxy\. The task can be deployed with the EC2 or Fargate launch type\. For more information about how to create an Amazon ECS Task Role and attach it to a task, see [Specifying an IAM Role for your Tasks](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-iam-roles.html#create_task_iam_policy_and_role)\.
+ **Amazon EC2** – The IAM role must be attached to the Amazon EC2 instance that hosts the Envoy proxy\. For more information about how to attach a role to an Amazon EC2 instance, see [I’ve created an IAM role, and now I want to assign it to an EC2 instance](http://aws.amazon.com/premiumsupport/knowledge-center/assign-iam-role-ec2-instance)\.

## Confirm permission<a name="confirm-permission"></a>

Confirm that the `appmesh:StreamAggregatedResources` permission is assigned to the compute resource that you host the proxy on by selecting one of the compute service names\.

------
#### [ Amazon EKS ]

A custom policy may be assigned to the role assigned to the worker nodes, to individual pods, or both\. It's recommended however, that you assign the policy only at individual pods, so that you can restrict access of individual pods to individual mesh endpoints\. If the policy is attached to the role assigned to the worker nodes, select the **Amazon EC2** tab, and complete the steps found there for your worker node instances\. To determine which IAM role is assigned to a Kubernetes pod, complete the following steps\.

1. View the details of a Kubernetes deployment that includes the pod that you want to confirm that a Kubernetes service account is assigned to\. The following command views the details for a deployment named *my\-deployment*\.

   ```
   kubectl describe deployment my-deployment
   ```

   In the returned output note the value to the right of `Service Account:`\. If a line that starts with `Service Account:` doesn't exist, then a custom Kubernetes service account isn't currently assigned to the deployment\. You'll need to assign one\. For more information, see [Configure Service Accounts for Pods](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/) in the Kubernetes documentation\.

1. View the details of the service account returned in the previous step\. The following command views the details of a service account named *my\-service\-account*\.

   ```
   kubectl describe serviceaccount my-service-account
   ```

   Provided the Kubernetes service account is associated to an AWS Identity and Access Management role, one of the lines returned will look similar to the following example\.

   ```
   Annotations:         eks.amazonaws.com/role-arn=arn:aws:iam::123456789012:role/my-deployment
   ```

   In the previous example `my-deployment` is the name of the IAM role that the service account is associated with\. If the service account output doesn't contain a line similar to the example above, then the Kubernetes service account isn't associated to an AWS Identity and Access Management account and you need to associate it to one\. For more information, see [Specifying an IAM Role for your Service Account](https://docs.aws.amazon.com/eks/latest/userguide/specify-service-account-role.html)\. 

1. Sign in to the AWS Management Console and open the IAM console at [https://console\.aws\.amazon\.com/iam/](https://console.aws.amazon.com/iam/)\.

1. In the left navigation, select **Roles**\. Select the name of the IAM role that you noted in a previous step\.

1. Confirm that either the custom policy you created previously, or the `[AWSAppMeshEnvoyAccess](https://console.aws.amazon.com/iam/home#/policies/arn:aws:iam::aws:policy/AWSAppMeshEnvoyAccess%24jsonEditor)` managed policy is listed\. If neither policy is attached, [attach an IAM policy](#attach-iam-policy) to the IAM role\. If you want to attach a custom IAM policy but don't have one, then you need to [create a custom IAM policy](#create-iam-policy) with the required permissions\. If a custom IAM policy is attached, select the policy and confirm that it contains `"Action": "appmesh:StreamAggregatedResources"`\. If it does not, then you need to add that permission to your custom IAM policy\. You can also confirm that the appropriate Amazon Resource Name \(ARN\) for a specific mesh endpoint is listed\. If no ARNs are listed, then you can edit the policy to add, remove, or change the listed ARNs\. For more information, see [Edit IAM Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-edit.html) and [Create IAM policy](#create-iam-policy)\.

1. Repeat the previous steps for each Kubernetes pod that contains the Envoy proxy\.

------
#### [ Amazon ECS ]

1. From the Amazon ECS console, choose **Task Definitions**\.

1. Select your Amazon ECS task\.

1. On the **Task Definition Name** page, select your task definition\.

1. On the **Task Definition** page, select the link of the IAM role name that is to the right of **Task Role**\. If an IAM role isn't listed, then you need to [create an IAM role](#create-iam-role) and attach it to your task by [updating your task definition\.](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/update-task-definition.html)

1. In the **Summary** page, on the **Permissions** tab, confirm that either the custom policy you created previously, or the `[AWSAppMeshEnvoyAccess](https://console.aws.amazon.com/iam/home#/policies/arn:aws:iam::aws:policy/AWSAppMeshEnvoyAccess%24jsonEditor)` managed policy is listed\. If neither policy is attached, [attach an IAM policy](#attach-iam-policy) to the IAM role\. If you want to attach a custom IAM policy but don't have one, then you need to [create the custom IAM policy](#create-iam-policy)\. If a custom IAM policy is attached, select the policy and confirm that it contains `"Action": "appmesh:StreamAggregatedResources"`\. If it does not, then you need to add that permission to your custom IAM policy\. You can also confirm that the appropriate Amazon Resource Name \(ARN\) for a specific mesh endpoints is listed\. If no ARNs are listed, then you can edit the policy to add, remove, or change the listed ARNs\. For more information, see [Edit IAM Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-edit.html) and [Create IAM policy](#create-iam-policy)\.

1. Repeat the previous steps for each task definition that contains the Envoy proxy\.

------
#### [ Amazon EC2 ]

1. From the Amazon EC2 console, select **Instances** in the left navigation\.

1. Select one of your instances that hosts the Envoy proxy\.

1. In the **Description** tab, select the link of the IAM role name that is to the right of **IAM role**\. If an IAM role isn't listed, then you need to [create an IAM role](#create-iam-role)\.

1. In the **Summary** page, on the **Permissions** tab, confirm that either the custom policy you created previously, or the `[AWSAppMeshEnvoyAccess](https://console.aws.amazon.com/iam/home#/policies/arn:aws:iam::aws:policy/AWSAppMeshEnvoyAccess%24jsonEditor)` managed policy is listed\. If neither policy is attached, [attach the IAM policy](#attach-iam-policy) to the IAM role\. If you want to attach a custom IAM policy but don't have one, then you need to [create the custom IAM policy](#create-iam-policy)\. If a custom IAM policy is attached, select the policy and confirm that it contains `"Action": "appmesh:StreamAggregatedResources"`\. If it does not, then you need to add that permission to your custom IAM policy\. You can also confirm that the appropriate Amazon Resource Name \(ARN\) for a specific mesh endpoints is listed\. If no ARNs are listed, then you can edit the policy to add, remove, or change the listed ARNs\. For more information, see [Edit IAM Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-edit.html) and [Create IAM policy](#create-iam-policy)\.

1. Repeat the previous steps for each instance that you host the Envoy proxy on\.

------
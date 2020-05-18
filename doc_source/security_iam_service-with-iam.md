# How AWS App Mesh works with IAM<a name="security_iam_service-with-iam"></a>

Before you use IAM to manage access to App Mesh, you should understand what IAM features are available to use with App Mesh\. To get a high\-level view of how App Mesh and other AWS services work with IAM, see [AWS Services That Work with IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_aws-services-that-work-with-iam.html) in the *IAM User Guide*\.

**Topics**
+ [App Mesh identity\-based policies](#security_iam_service-with-iam-id-based-policies)
+ [App Mesh resource\-based policies](#security_iam_service-with-iam-resource-based-policies)
+ [Authorization based on App Mesh tags](#security_iam_service-with-iam-tags)
+ [App Mesh IAM roles](#security_iam_service-with-iam-roles)

## App Mesh identity\-based policies<a name="security_iam_service-with-iam-id-based-policies"></a>

With IAM identity\-based policies, you can specify allowed or denied actions and resources as well as the conditions under which actions are allowed or denied\. App Mesh supports specific actions, resources, and condition keys\. To learn about all of the elements that you use in a JSON policy, see [IAM JSON Policy Elements Reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements.html) in the *IAM User Guide*\.

### Actions<a name="security_iam_service-with-iam-id-based-policies-actions"></a>

The `Action` element of an IAM identity\-based policy describes the specific action or actions that will be allowed or denied by the policy\. Policy actions usually have the same name as the associated AWS API operation\. The action is used in a policy to grant permissions to perform the associated operation\. 

Policy actions in App Mesh use the following prefix before the action: `appmesh:`\. For example, to grant someone permission to list meshes in an account with the `appmesh:ListMeshes` API operation, you include the `appmesh:ListMeshes` action in their policy\. Policy statements must include either an `Action` or `NotAction` element\.

To specify multiple actions in a single statement, separate them with commas as follows\.

```
"Action": [
      "appmesh:ListMeshes",
      "appmesh:ListVirtualNodes"
]
```

You can specify multiple actions using wildcards \(\*\)\. For example, to specify all actions that begin with the word `Describe`, include the following action\.

```
"Action": "appmesh:Describe*"
```



To see a list of App Mesh actions, see [Actions Defined by AWS App Mesh](https://docs.aws.amazon.com/IAM/latest/UserGuide/list_awsappmesh.html#awsappmesh-actions-as-permissions) in the *IAM User Guide*\.

### Resources<a name="security_iam_service-with-iam-id-based-policies-resources"></a>

The `Resource` element specifies the object or objects to which the action applies\. Statements must include either a `Resource` or a `NotResource` element\. You specify a resource using an ARN or using the wildcard \(\*\) to indicate that the statement applies to all resources\.



The App Mesh `mesh` resource has the following ARN\.

```
arn:${Partition}:appmesh:${Region}:${Account}:mesh/${MeshName}
```

For more information about the format of ARNs, see [Amazon Resource Names \(ARNs\) and AWS Service Namespaces](https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html)\.

For example, to specify the mesh named *apps* in the *region\-code* Region in your statement, use the following ARN\.

```
arn:aws:appmesh:region-code:111122223333:mesh/apps
```

To specify all instances that belong to a specific account, use the wildcard \(\*\)\.

```
"Resource": "arn:aws:appmesh:region-code:111122223333:mesh/*"
```

Some App Mesh actions, such as those for creating resources, cannot be performed on a specific resource\. In those cases, you must use the wildcard \(\*\)\.

```
"Resource": "*"
```

Many App Mesh API actions involve multiple resources\. For example, `CreateRoute` creates a route with a virtual node target, so an IAM user must have permissions to use the route and the virtual node\. To specify multiple resources in a single statement, separate the ARNs with commas\. 

```
"Resource": [
      "arn:aws:appmesh:region-code:111122223333:mesh/apps/virtualRouter/serviceB/route/*",
      "arn:aws:appmesh:region-code:111122223333:mesh/apps/virtualNode/serviceB"
]
```

To see a list of App Mesh resource types and their ARNs, see [Resources Defined by AWS App Mesh](https://docs.aws.amazon.com/IAM/latest/UserGuide/list_awsappmesh.html#awsappmesh-resources-for-iam-policies) in the *IAM User Guide*\. To learn with which actions you can specify the ARN of each resource, see [Actions Defined by AWS App Mesh](https://docs.aws.amazon.com/IAM/latest/UserGuide/list_awsappmesh.html#awsappmesh-actions-as-permissions)\.

### Condition keys<a name="security_iam_service-with-iam-id-based-policies-conditionkeys"></a>

App Mesh supports using some global condition keys\. To see all AWS global condition keys, see [AWS Global Condition Context Keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html) in the *IAM User Guide*\. To see a list of the global condition keys that App Mesh supports, see [Condition Keys for AWS App Mesh](https://docs.aws.amazon.com/IAM/latest/UserGuide/list_awsappmesh.html#awsappmesh-policy-keys) in the *IAM User Guide*\. To learn with which actions and resources you can use with a condition key, see [Actions Defined by AWS App Mesh](https://docs.aws.amazon.com/IAM/latest/UserGuide/list_awsappmesh.html#awsappmesh-actions-as-permissions)\.

### Examples<a name="security_iam_service-with-iam-id-based-policies-examples"></a>



To view examples of App Mesh identity\-based policies, see [AWS App Mesh identity\-based policy examples](security_iam_id-based-policy-examples.md)\.

## App Mesh resource\-based policies<a name="security_iam_service-with-iam-resource-based-policies"></a>

App Mesh does not support resource\-based policies\.

## Authorization based on App Mesh tags<a name="security_iam_service-with-iam-tags"></a>

You can attach tags to App Mesh resources or pass tags in a request to App Mesh\. To control access based on tags, you provide tag information in the [condition element](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition.html) of a policy using the `appmesh:ResourceTag/key-name`, `aws:RequestTag/key-name`, or `aws:TagKeys` condition keys\. For more information about tagging App Mesh resources, see [Tagging AWS Resources](https://docs.aws.amazon.com/general/latest/gr/aws_tagging.html)\.

To view an example identity\-based policy for limiting access to a resource based on the tags on that resource, see [Creating App Mesh meshes with restricted tags](security_iam_id-based-policy-examples.md#security_iam_id-based-policy-examples-view-widget-tags)\.

## App Mesh IAM roles<a name="security_iam_service-with-iam-roles"></a>

An [IAM role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html) is an entity within your AWS account that has specific permissions\.

### Using temporary credentials with App Mesh<a name="security_iam_service-with-iam-roles-tempcreds"></a>

You can use temporary credentials to sign in with federation, assume an IAM role, or to assume a cross\-account role\. You obtain temporary security credentials by calling AWS STS API operations such as [AssumeRole](https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRole.html) or [GetFederationToken](https://docs.aws.amazon.com/STS/latest/APIReference/API_GetFederationToken.html)\. 

App Mesh supports using temporary credentials\. 

### Service\-linked roles<a name="security_iam_service-with-iam-roles-service-linked"></a>

[Service\-linked roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_terms-and-concepts.html#iam-term-service-linked-role) allow AWS services to access resources in other services to complete an action on your behalf\. Service\-linked roles appear in your IAM account and are owned by the service\. An IAM administrator can view but not edit the permissions for service\-linked roles\.

App Mesh supports service\-linked roles\. For details about creating or managing App Mesh service\-linked roles, see [Using Service\-Linked Roles for App Mesh](using-service-linked-roles.md)\.

### Service roles<a name="security_iam_service-with-iam-roles-service"></a>

This feature allows a service to assume a [service role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_terms-and-concepts.html#iam-term-service-role) on your behalf\. This role allows the service to access resources in other services to complete an action on your behalf\. Service roles appear in your IAM account and are owned by the account\. This means that an IAM administrator can change the permissions for this role\. However, doing so might break the functionality of the service\.

App Mesh does not support service roles\.
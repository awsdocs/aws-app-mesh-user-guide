# AWS App Mesh IAM Policies, Roles, and Permissions<a name="IAM_policies"></a>

By default, IAM users don't have permission to create or modify AWS App Mesh resources or perform tasks using the App Mesh API\. \(This means that they also can't do so using the AWS CLI\.\) To allow IAM users to create or modify service meshes, you must create IAM policies that grant IAM users permissions to use the specific resources and API actions that they need, and then attach those policies to the IAM users or groups that require those permissions\.

When you attach a policy to a user or group of users, it allows or denies the users permission to perform the specified tasks on the specified resources\. For more information, see [Permissions and Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/PermissionsAndPolicies.html) in the *IAM User Guide*\. For more information about managing and creating custom IAM policies, see [Managing IAM Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/ManagingPolicies.html)\.

**Getting Started**

An IAM policy must grant or deny permissions to use one or more App Mesh actions\.

**Topics**
+ [Policy Structure](iam-policy-structure.md)
+ [Creating App Mesh IAM Policies](MESH_IAM_user_policies.md)
+ [Using Service\-Linked Roles for App Mesh](using-service-linked-roles.md)
+ [Proxy Authorization](proxy-authorization.md)
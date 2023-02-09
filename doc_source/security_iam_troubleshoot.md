# Troubleshooting AWS App Mesh identity and access<a name="security_iam_troubleshoot"></a>

Use the following information to help you diagnose and fix common issues that you might encounter when working with App Mesh and IAM\.

**Topics**
+ [I am not authorized to perform an action in App Mesh](#security_iam_troubleshoot-no-permissions)
+ [I want to allow people outside of my AWS account to access my App Mesh resources](#security_iam_troubleshoot-cross-account-access)

## I am not authorized to perform an action in App Mesh<a name="security_iam_troubleshoot-no-permissions"></a>

If the AWS Management Console tells you that you're not authorized to perform an action, then you must contact your administrator for assistance\. Your administrator is the person that provided you with your sign\-in credentials\.

The following error occurs when the `mateojackson` tries to use the console to create a virtual node named *my\-virtual\-node* in the mesh named *my\-mesh* but does not have the `appmesh:CreateVirtualNode` permission\.

```
User: arn:aws:iam::123456789012:user/mateojackson is not authorized to perform: appmesh:CreateVirtualNode on resource: arn:aws:appmesh:us-east-1:123456789012:mesh/my-mesh/virtualNode/my-virtual-node
```

In this case, Mateo asks his administrator to update his policies to allow him to create a virtual node using the `appmesh:CreateVirtualNode` action\.

**Note**  
Since a virtual node is created within a mesh, Mateo's account also requires the `appmesh:DescribeMesh` and `appmesh:ListMeshes` actions to create the virtual node in the console\.

## I want to allow people outside of my AWS account to access my App Mesh resources<a name="security_iam_troubleshoot-cross-account-access"></a>

You can create a role that users in other accounts or people outside of your organization can use to access your resources\. You can specify who is trusted to assume the role\. For services that support resource\-based policies or access control lists \(ACLs\), you can use those policies to grant people access to your resources\.

To learn more, consult the following:
+ To learn whether App Mesh supports these features, see [How AWS App Mesh works with IAM](security_iam_service-with-iam.md)\.
+ To learn how to provide access to your resources across AWS accounts that you own, see [Providing access to an IAM user in another AWS account that you own](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_common-scenarios_aws-accounts.html) in the *IAM User Guide*\.
+ To learn how to provide access to your resources to third\-party AWS accounts, see [Providing access to AWS accounts owned by third parties](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_common-scenarios_third-party.html) in the *IAM User Guide*\.
+ To learn how to provide access through identity federation, see [Providing access to externally authenticated users \(identity federation\)](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_common-scenarios_federated-users.html) in the *IAM User Guide*\.
+ To learn the difference between using roles and resource\-based policies for cross\-account access, see [How IAM roles differ from resource\-based policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_compare-resource-policies.html) in the *IAM User Guide*\.
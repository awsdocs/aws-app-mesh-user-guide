# Troubleshooting AWS App Mesh identity and access<a name="security_iam_troubleshoot"></a>

Use the following information to help you diagnose and fix common issues that you might encounter when working with App Mesh and IAM\.

**Topics**
+ [I am not authorized to perform an action in App Mesh](#security_iam_troubleshoot-no-permissions)
+ [I want to view my access keys](#security_iam_troubleshoot-access-keys)
+ [I'm an administrator and want to allow others to access App Mesh](#security_iam_troubleshoot-admin-delegate)
+ [I want to allow people outside of my AWS account to access my App Mesh resources](#security_iam_troubleshoot-cross-account-access)

## I am not authorized to perform an action in App Mesh<a name="security_iam_troubleshoot-no-permissions"></a>

If the AWS Management Console tells you that you're not authorized to perform an action, then you must contact your administrator for assistance\. Your administrator is the person that provided you with your user name and password\.

The following error occurs when the `mateojackson` IAM user tries to use the console to create a virtual node named *my\-virtual\-node* in the mesh named *my\-mesh* but does not have the `appmesh:CreateVirtualNode` permission\.

```
User: arn:aws:iam::123456789012:user/mateojackson is not authorized to perform: appmesh:CreateVirtualNode on resource: arn:aws:appmesh:us-east-1:123456789012:mesh/my-mesh/virtualNode/my-virtual-node
```

In this case, Mateo asks his administrator to update his policies to allow him to create a virtual node using the `appmesh:CreateVirtualNode` action\.

**Note**  
Since a virtual node is created within a mesh, Mateo's account also requires the `appmesh:DescribeMesh` and `appmesh:ListMeshes` actions to create the virtual node in the console\.

## I want to view my access keys<a name="security_iam_troubleshoot-access-keys"></a>

After you create your IAM user access keys, you can view your access key ID at any time\. However, you can't view your secret access key again\. If you lose your secret key, you must create a new access key pair\. 

Access keys consist of two parts: an access key ID \(for example, `AKIAIOSFODNN7EXAMPLE`\) and a secret access key \(for example, `wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY`\)\. Like a user name and password, you must use both the access key ID and secret access key together to authenticate your requests\. Manage your access keys as securely as you do your user name and password\.

**Important**  
 Do not provide your access keys to a third party, even to help [find your canonical user ID](https://docs.aws.amazon.com/general/latest/gr/acct-identifiers.html#FindingCanonicalId)\. By doing this, you might give someone permanent access to your account\. 

When you create an access key pair, you are prompted to save the access key ID and secret access key in a secure location\. The secret access key is available only at the time you create it\. If you lose your secret access key, you must add new access keys to your IAM user\. You can have a maximum of two access keys\. If you already have two, you must delete one key pair before creating a new one\. To view instructions, see [Managing Access Keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html#Using_CreateAccessKey) in the *IAM User Guide*\.

## I'm an administrator and want to allow others to access App Mesh<a name="security_iam_troubleshoot-admin-delegate"></a>

To allow others to access App Mesh, you must create an IAM entity \(user or role\) for the person or application that needs access\. They will use the credentials for that entity to access AWS\. You must then attach a policy to the entity that grants them the correct permissions in App Mesh\.

To get started right away, see [Creating Your First IAM Delegated User and Group](https://docs.aws.amazon.com/IAM/latest/UserGuide/getting-started_create-delegated-user.html) in the *IAM User Guide*\.

## I want to allow people outside of my AWS account to access my App Mesh resources<a name="security_iam_troubleshoot-cross-account-access"></a>

You can create a role that users in other accounts or people outside of your organization can use to access your resources\. You can specify who is trusted to assume the role\. For services that support resource\-based policies or access control lists \(ACLs\), you can use those policies to grant people access to your resources\.

To learn more, consult the following:
+ To learn whether App Mesh supports these features, see [How AWS App Mesh works with IAM](security_iam_service-with-iam.md)\.
+ To learn how to provide access to your resources across AWS accounts that you own, see [Providing Access to an IAM User in Another AWS Account That You Own](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_common-scenarios_aws-accounts.html) in the *IAM User Guide*\.
+ To learn how to provide access to your resources to third\-party AWS accounts, see [Providing Access to AWS Accounts Owned by Third Parties](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_common-scenarios_third-party.html) in the *IAM User Guide*\.
+ To learn how to provide access through identity federation, see [Providing Access to Externally Authenticated Users \(Identity Federation\)](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_common-scenarios_federated-users.html) in the *IAM User Guide*\.
+ To learn the difference between using roles and resource\-based policies for cross\-account access, see [How IAM Roles Differ from Resource\-based Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_compare-resource-policies.html) in the *IAM User Guide*\.
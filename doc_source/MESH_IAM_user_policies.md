# Creating App Mesh IAM Policies<a name="MESH_IAM_user_policies"></a>

You can create specific IAM policies to restrict the calls and resources that users in your account have access to, and then attach those policies to IAM users\.

When you attach a policy to a user or group of users, it allows or denies the users permission to perform the specified tasks on the specified resources\. For more information, see [Permissions and Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/PermissionsAndPolicies.html) in the *IAM User Guide*\. For more information about managing and creating custom IAM policies, see [Managing IAM Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/ManagingPolicies.html)\.

If your IAM user doesn't have administrative privileges, you must explicitly add permissions for that user to call the App Mesh API operations\.

**To create an IAM policy for an App Mesh admin user**

1. Open the IAM console at [https://console\.aws\.amazon\.com/iam/](https://console.aws.amazon.com/iam/)\.

1. In the navigation pane, choose **Policies** and then **Create policy**\. 

1. On the **JSON** tab, paste the following policy\.

   ```
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Effect": "Allow",
               "Action": [
                   "appmesh:*"
               ],
               "Resource": "*"
           }
       ]
   }
   ```

1. Choose **Review policy**\.

1. For **Name**, enter your own unique name, such as `AWSAppMeshAdminPolicy`\.

1. Choose **Create Policy**\. 

**To attach an IAM policy to a user**

1. Open the IAM console at [https://console\.aws\.amazon\.com/iam/](https://console.aws.amazon.com/iam/)\.

1. In the navigation pane, choose **Users** and then choose the user to attach the policy to\. 

1. Choose **Permissions** and then **Add permissions**\.

1. Under **Grant permissions**, choose **Attach existing policies directly**\.

1. Select the custom policy that you created in the previous procedure and choose **Next: Review**\.

1. Review your details and choose **Add permissions**\.
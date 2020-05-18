# Logging App Mesh API calls with AWS CloudTrail<a name="logging-using-cloudtrail"></a>

AWS App Mesh works with AWS CloudTrail, a service that provides a record of actions taken by a user, a role, or an AWS service in App Mesh\. CloudTrail captures all API calls for App Mesh as events\. The calls captured include calls from the App Mesh console and code calls to the App Mesh API operations\. If you create a trail, you can enable continuous delivery of CloudTrail events to an Amazon S3 bucket, including events for App Mesh\. If you don't configure a trail, you can still view the most recent events in the CloudTrail console in **Event history**\. Using the information collected by CloudTrail, you can determine the request that was made to App Mesh, the IP address from which the request was made, which user or account made the request, when it was made, and additional details\. 

To learn more about CloudTrail, see the [AWS CloudTrail User Guide](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/)\.

## App Mesh information in CloudTrail<a name="service-name-info-in-cloudtrail"></a>

CloudTrail is enabled on your AWS account when you create the account\. When activity occurs in App Mesh, that activity is recorded in a CloudTrail event along with other AWS service events in **Event history**\. You can view, search, and download recent events in your AWS account\. For more information, see [Viewing Events with CloudTrail Event History](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/view-cloudtrail-events.html)\. 

For an ongoing record of events in your AWS account, including events for App Mesh, create a trail\. A *trail* enables CloudTrail to deliver log files to an Amazon S3 bucket\. By default, when you create a trail in the console, the trail applies to all AWS Regions\. The trail logs events from all Regions in the AWS partition and delivers the log files to the Amazon S3 bucket that you specify\. Additionally, you can configure other AWS services to further analyze and act upon the event data collected in CloudTrail logs\. For more information, see the following: 
+ [Overview for Creating a Trail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-create-and-update-a-trail.html)
+ [CloudTrail Supported Services and Integrations](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-aws-service-specific-topics.html#cloudtrail-aws-service-specific-topics-integrations)
+ [Configuring Amazon SNS Notifications for CloudTrail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/getting_notifications_top_level.html)
+ [Receiving CloudTrail Log Files from Multiple Regions](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/receive-cloudtrail-log-files-from-multiple-regions.html) and [Receiving CloudTrail Log Files from Multiple Accounts](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-receive-logs-from-multiple-accounts.html)

All App Mesh actions are logged by CloudTrail and are documented in [API Actions](https://docs.aws.amazon.com/app-mesh/latest/APIReference/API_Operations.html)\. For example, calls to the [https://docs.aws.amazon.com/app-mesh/latest/APIReference/API_CreateMesh.html](https://docs.aws.amazon.com/app-mesh/latest/APIReference/API_CreateMesh.html), [https://docs.aws.amazon.com/app-mesh/latest/APIReference/API_DescribeMesh.html](https://docs.aws.amazon.com/app-mesh/latest/APIReference/API_DescribeMesh.html), and [https://docs.aws.amazon.com/app-mesh/latest/APIReference/API_DeleteMesh.html](https://docs.aws.amazon.com/app-mesh/latest/APIReference/API_DeleteMesh.html) actions generate entries in the CloudTrail log files\. Actions that App Mesh takes on your behalf, such as creating a service\-linked role when you create a mesh, are also logged\.

Every event or log entry contains information about who generated the request\. The identity information helps you determine the following: 
+ Whether the request was made with root or AWS Identity and Access Management \(IAM\) user credentials
+ Whether the request was made with temporary security credentials for a role or federated user
+ Whether the request was made by another AWS service

For more information, see the [CloudTrail userIdentity Element](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-event-reference-user-identity.html)\.

## Understanding App Mesh log file entries<a name="understanding-service-name-entries"></a>

A trail is a configuration that enables delivery of events as log files to an Amazon S3 bucket that you specify\. CloudTrail log files contain one or more log entries\. An event represents a single request from any source and includes information about the requested action, the date and time of the action, request parameters, and so on\. CloudTrail log files aren't an ordered stack trace of the public API calls, so they don't appear in any specific order\. 

The following example shows a CloudTrail log entry that demonstrates the [https://docs.aws.amazon.com/app-mesh/latest/APIReference/API_CreateMesh.html](https://docs.aws.amazon.com/app-mesh/latest/APIReference/API_CreateMesh.html) action\.

```
{
    "eventVersion": "1.05",
    "userIdentity": {
        "type": "IAMUser",
        "principalId": "AKIAIOSFODNN7EXAMPLE",
        "arn": "arn:aws:iam::123456789012:user/Mary_Major",
        "accountId": "123456789012",
        "accessKeyId": "AKIAIOSFODNN7EXAMPLE",
        "userName": "Mary_Major",
        "sessionContext": {
            "sessionIssuer": {},
            "webIdFederationData": {},
            "attributes": {
                "mfaAuthenticated": "false",
                "creationDate": "2019-10-18T14:56:49Z"
            }
        },
        "invokedBy": "signin.amazonaws.com"
    },
    "eventTime": "2019-10-18T15:00:49Z",
    "eventSource": "appmesh.amazonaws.com",
    "eventName": "CreateMesh",
    "awsRegion": "us-east-2",
    "sourceIPAddress": "205.251.233.178",
    "userAgent": "signin.amazonaws.com",
    "requestParameters": {
        "meshName": "my-mesh",
        "clientToken": "00000000-0000-0000-0000-0000000000",
        "spec": {
            "egressFilter": {
                "type": "DROP_ALL"
            }
        }
    },
    "responseElements": {
        "mesh": {
            "meshName": "my-mesh",
            "status": {
                "status": "ACTIVE"
            },
            "metadata": {
                "version": 1,
                "lastUpdatedAt": "Oct 18, 2019 3:00:49 PM",
                "uid": "00000000-0000-0000-0000-000000000000",
                "createdAt": "Oct 18, 2019 3:00:49 PM",
                "arn": "arn:aws:iam::123456789012::mesh/my-mesh"
            },
            "spec": {
                "egressFilter": {
                    "type": "DROP_ALL"
                }
            }
        }
    },
    "requestID": "cb8c167e-EXAMPLE",
    "eventID": "e3c6f4ce-EXAMPLE",
    "readOnly": false,
    "eventType": "AwsApiCall",
    "apiVersion": "2019-01-25",
    "recipientAccountId": 123456789012"
}
```
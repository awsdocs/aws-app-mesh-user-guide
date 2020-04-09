# App Mesh Preview Channel<a name="preview"></a>

The App Mesh Preview Channel is a distinct variant of the App Mesh service provided in the `us-west-2` Region\. The Preview Channel exposes upcoming features for you to try as they are developed\. As you use features in the Preview Channel, you can provide feedback via GitHub to shape how the features behave\. Once a feature has complete functionality in the Preview Channel, with all of the necessary integrations and checks completed, it will graduate to the production App Mesh service\.

The AWS App Mesh Preview Channel is a *Beta Service* and all features are *previews*, as those terms are defined in the [AWS Service Terms](http://aws.amazon.com/service-terms)\. Your participation in the Preview Channel is governed by your Agreement with AWS and the AWS Service Terms, in particular, the Universal and Beta Service Participation terms, and is confidential\. 

The following questions are frequently asked about the Preview Channel\.

## What is the Preview Channel?<a name="what-is"></a>

The Preview Channel is a public service endpoint that allows you to try out and provide feedback on new service features before they are generally available\. The service endpoint for the Preview Channel is separate from the standard production endpoint\. You interact with the endpoint by using the AWS CLI, a service model file for the Preview Channel, and command input files for the AWS CLI\. The Preview Channel, allows you to try new features without impacting your current production infrastructure\. You're encouraged to [provide feedback](#provide-feedback) to the App Mesh team to help ensure that App Mesh meets customers' most important requirements\. Your feedback on features while they're in the Preview Channel can help shape the features of App Mesh so that we can deliver the best possible service\.

## How is the Preview Channel different from production App Mesh?<a name="differences"></a>

The following table lists aspects of the App Mesh service that are different from the Preview Channel\.


| 
| 
| Aspect | App Mesh production service | App Mesh Preview Channel service  | 
| --- |--- |--- |
| Frontend endpoint | appmesh\.us\-west\-2\.amazonaws\.com | appmesh\-preview\.us\-west\-2\.amazonaws\.com | 
| Envoy management service endpoint | appmesh\-envoy\-management\.us\-west\-2\.amazonaws\.com | appmesh\-preview\-envoy\-management\.us\-west\-2\.amazonaws\.com | 
| CLI | aws appmesh list\-meshes | aws appmesh\-preview list\-meshes \(only available after adding the Preview Channel service model\) | 
| Signing name | appmesh | appmesh\-preview | 
| Service principal | appmesh\.amazonaws\.com | appmesh\-preview\.amazonaws\.com | 

**Note**  
Though the example in the table for the App Mesh production service lists the `us-west-2` Region, the production service is available in most Regions\. For a list of all Regions that the App Mesh production service is available in, see [AWS App Mesh Endpoints and Quotas](https://docs.aws.amazon.com/general/latest/gr/appmesh.html)\. However, the App Mesh Preview Channel service is available only in the `us-west-2` Region\. 

## How can I use features in the Preview Channel?<a name="try-out"></a>

1. Add the Preview Channel service model that includes the Preview Channel feature to the AWS CLI with the following command\.

   ```
   aws configure add-model \
       --service-name appmesh-preview \
       --service-model https://raw.githubusercontent.com/aws/aws-app-mesh-roadmap/master/appmesh-preview/service-model.json
   ```

1. Create a JSON file that includes the feature, based on the JSON example and instructions provided in the [AWS App Mesh User Guide](https://docs.aws.amazon.com/app-mesh/latest/userguide/) for the feature\.

1. Implement the feature with the appropriate AWS CLI command and command input file\. For example, the following command creates a route with Preview Channel features using the *route\.json* file\.

   ```
   aws appmesh-preview create-route --cli-input-json file://route.json
   ```

1. Add `APPMESH_PREVIEW = 1` as a configuration variable for the Envoy container when adding it to your Amazon ECS task definitions, Kubernetes Pod specifications, or Amazon EC2 instances\. This variable enables the Envoy container to communicate with the Preview Channel endpoints\. For more information about adding configuration variables, see [Updating services in Amazon ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/appmesh-getting-started.html#update-services), [Updating services in Kubernetes](https://docs.aws.amazon.com/eks/latest/userguide/appmesh-getting-started.html#update-services), and [Updating services on Amazon EC2](https://docs.aws.amazon.com/app-mesh/latest/userguide/appmesh-getting-started.html#update-services)\.

## How do I provide feedback?<a name="provide-feedback"></a>

You can provide feedback directly on the [App Mesh roadmap GitHub repo](https://github.com/aws/aws-app-mesh-roadmap/issues) issue that is linked from the documentation about the feature\.

## How long do I have to provide feedback on a feature in the Preview Channel?<a name="feedback-duration"></a>

The feedback period will vary depending on the size and complexity of the feature being introduced\. We intend to give a comment period of 14 days between release of a feature to the preview endpoint and release of the feature to production\. The App Mesh team may extend the feedback period for specific features\.

## What level of support is provided for the Preview Channel?<a name="support"></a>

While we encourage you to provide feedback and bug reports directly on the App Mesh [GitHub roadmap issue](https://github.com/aws/aws-app-mesh-roadmap/issues), we understand that you may have sensitive data to share, or you may find an issue that you do not feel is safe to disclose publicly\. For these issues, you can contact AWS Support or email the App Mesh team directly at [ aws\-appmesh\-feedback@amazon\.com](mailto:aws-appmesh-feedback@amazon.com)\.

## Is my data secure on the Preview Channel endpoint?<a name="data-security"></a>

Yes\. The Preview Channel is given the same level of security as the standard production endpoint\.

## How long will my configuration be available?<a name="data-durability"></a>

You can work with a mesh in the Preview Channel for thirty days\. Thirty days after a mesh is created, you can only list, read, or delete the mesh\. If you attempt to create or update resources after thirty days, you'll receive a `BadRequest` exception explaining that the mesh is archived\. 

## What tools can I use to work with the Preview Channel?<a name="tools"></a>

You can use the AWS CLI with a Preview Channel service model file and command input files\. For more information about how to work with features, see [How can I use features in the Preview Channel?](#try-out)\. You cannot use AWS CLI command options, the AWS Management Console, SDKs, or AWS CloudFormation to work with Preview Channel features\. You can use all tools however, once a feature is released to the production service\.

## Will there be forward compatibility of Preview Channel APIs?<a name="forward-compatibility"></a>

No\. APIs may change based on feedback\.

## Are Preview Channel features complete?<a name="feature-completeness"></a>

No\. New API objects may not be fully integrated into the AWS Management Console, AWS CloudFormation, or AWS CloudTrail\. As features solidify in the Preview Channel and near general availability, the integrations will eventually become available\.

## Is documentation available for Preview Channel features?<a name="documentation"></a>

Yes\. Documentation for Preview Channel features is included in the production documentation\. For example, if features for the route resource are released to the Preview Channel, information about how to use the features would be in the existing [route](routes.md) resource document\. Preview Channel features are labeled as only available in the Preview Channel\.

## How will I know when new features are available in the Preview Channel?<a name="new-features"></a>

When new features are introduced in the Preview Channel, an entry is added to the [App Mesh Document History](https://docs.aws.amazon.com/app-mesh/latest/userguide/doc-history.html)\. You can review the page regularly or subscribe to the [App Mesh Document History RSS feed](https://docs.aws.amazon.com/app-mesh/latest/userguide/app-mesh-ug.rss)\. Additionally, you can review the [issues](https://github.com/aws/aws-app-mesh-roadmap/issues) for the aws\-app\-mesh\-roadmap GitHub repo\. A download link for the Preview Channel service model JSON file is added to an issue when it releases to the Preview Channel\. For more information about how to use the model and feature, see [How can I use features in the Preview Channel?](#try-out)\.

## How will I know when a feature has graduated to the production service?<a name="feature-status"></a>

The text in the App Mesh documentation noting that the feature is available only in the Preview Channel is removed, and an entry is added to the [App Mesh Document History](https://docs.aws.amazon.com/app-mesh/latest/userguide/doc-history.html)\. You can review the page regularly or subscribe to the [App Mesh Document History RSS feed](https://docs.aws.amazon.com/app-mesh/latest/userguide/app-mesh-ug.rss)\.
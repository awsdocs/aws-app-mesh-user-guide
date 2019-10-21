# Virtual Routers<a name="virtual_routers"></a>

Virtual routers handle traffic for one or more virtual services within your mesh\. After you create a virtual router, you can create and associate routes for your virtual router that direct incoming requests to different virtual nodes\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/app-mesh/latest/userguide/images/virtual_router.png)

Any inbound traffic that your virtual router expects should be specified as a *listener*\.

## Creating a Virtual Router<a name="create-virtual-router"></a>

To create a virtual router, select the tool that you want to use to create it\.

### AWS Management Console<a name="console"></a>

1. Open the App Mesh console at [https://console\.aws\.amazon\.com/appmesh/](https://console.aws.amazon.com/appmesh/)\.

1. Choose the mesh that you want to create the virtual router in\.

1. Choose **Virtual routers** in the left navigation\.

1. Choose **Create virtual router**\.

1. For **Virtual router name**, specify a name for your virtual router\. Up to 255 letters, numbers, hyphens, and underscores are allowed\.

1. For **Listener**, specify a **Port** and **Protocol** for your virtual router\.

1. Choose **Create virtual router** to finish\.

### AWS CLI<a name="cli"></a>

To create a virtual router with released features using the AWS CLI version 1\.16\.235 or higher, see the example in the AWS CLI reference for the [create\-virtual\-router](https://docs.aws.amazon.com/cli/latest/reference/appmesh/create-virtual-router.html) command\.

 \([App Mesh Preview Channel](https://docs.aws.amazon.com//app-mesh/latest/userguide/preview.html) only\) To create a virtual router with an HTTP2 or GRPC listener\.

1. Download the Preview Channel service model with the following command\.

   ```
   curl -o appmesh-preview-channel-service-model.json https://raw.githubusercontent.com/aws/aws-app-mesh-roadmap/master/appmesh-preview/service-model.json
   ```

1. Add the Preview Channel service model to the AWS CLI with the following command\.

   ```
   aws configure add-model \
       --service-name appmesh-preview \
       --service-model file://appmesh-preview-channel-service-model.json
   ```

1. Create a JSON file named `create-virtual-router.json` with a virtual router configuration\. In the following example JSON file, a virtual router with an HTTP2 listener is created in a mesh named *app1*\. Alternately, for `protocol` you can replace *http2* with `grpc`\.

   ```
   {
       "meshName": "app1",
       "spec": {
           "listeners": [
               {
                   "portMapping": {
                       "port": 80,
                       "protocol": "http2"
                   }
               }
           ]
       },
       "virtualRouterName": "serviceB-http2"
   }
   ```

1. Create the virtual router with the following command\.

   ```
   aws appmesh-preview create-virtual-router --cli-input-json file://create-virtual-router.json
   ```
# Virtual Routers<a name="virtual_routers"></a>

Virtual routers handle traffic for one or more virtual services within your mesh\. After you create a virtual router, you can create and associate routes for your virtual router that direct incoming requests to different virtual nodes\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/app-mesh/latest/userguide/images/virtual_router.png)

Any inbound traffic that your virtual router expects should be specified as a *listener*\.

## Creating a Virtual Router<a name="create-virtual-router"></a>

This topic helps you to create a virtual router in your service mesh\.

**Creating a virtual router in the AWS Management Console\.**

1. Open the App Mesh console at [https://console\.aws\.amazon\.com/appmesh/](https://console.aws.amazon.com/appmesh/)\.

1. Choose the mesh that you want to create the virtual router in\.

1. Choose **Virtual routers** in the left navigation\.

1. Choose **Create virtual router**\.

1. For **Virtual router name**, specify a name for your virtual router\. Up to 255 letters, numbers, hyphens, and underscores are allowed\.

1. For **Listener**, specify a **Port** and **Protocol** for your virtual router\.

1. Choose **Create virtual router** to finish\.

**Creating a virtual router in the AWS CLI\.**
+ The following JSON represents a virtual router named `serviceB` that listens for HTTP traffic on port 80\.

  ```
  {
    "meshName": "simpleapp",
    "spec": {
          "listeners": [
              {
                  "portMapping": {
                      "port": 80,
                      "protocol": "http"
                  }
              }
          ]
      },
    "virtualRouterName": "serviceB"
  }
  ```

  If you save the preceding JSON as a file, you can create the virtual router with the following AWS CLI command\.

  ```
  aws appmesh create-virtual-router --cli-input-json file://serviceB-router.json
  ```
**Note**  
You must use at least version 1\.16\.133 of the AWS CLI with App Mesh\. To install the latest version of the AWS CLI, see [Installing the AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/installing.html) in the *AWS Command Line Interface User Guide*\.

  For more information about creating virtual routers with the AWS CLI, see [create\-virtual\-router](https://docs.aws.amazon.com/cli/latest/reference/appmesh/create-virtual-router.html) in the *AWS Command Line Interface User Guide*\.
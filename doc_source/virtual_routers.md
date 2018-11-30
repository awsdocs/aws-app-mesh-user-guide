# Virtual Routers<a name="virtual_routers"></a>

Virtual routers handle traffic for one or more service names within your mesh\. After you create a virtual router, you can create and associate routes for your virtual router that direct incoming requests to different virtual nodes\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/app-mesh/latest/userguide/images/virtual_router.png)

Each service name within the mesh must be fronted by a virtual router, and the service name you specify for the virtual router must be a real DNS service name within your VPC \(in most cases you should just use the same service name that you specified for your virtual nodes\)\.

The following JSON represents a virtual router called `serviceB`, for the service name `serviceB.simpleapp.local`\.

```
{
  "meshName": "simpleapp",
  "spec": {
    "serviceNames": [
      "serviceB.simpleapp.local"
    ]
  },
  "virtualRouterName": "serviceB"
}
```

If the above JSON is saved as a file, you can create the virtual node with the following AWS CLI command\.

```
aws appmesh create-virtual-router --cli-input-json file://serviceB-router.json
```

**Note**  
You must use at least version 1\.16\.65 of the AWS CLI with App Mesh\. To install the latest version of the AWS CLI, see [Installing the AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/installing.html) in the *AWS Command Line Interface User Guide*\.

For more information about creating virtual routers with the AWS CLI, see [create\-virtual\-router](https://docs.aws.amazon.com/cli/latest/reference/appmesh/create-virtual-router.html) in the *AWS Command Line Interface User Guide*\.
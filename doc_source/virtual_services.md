# Virtual Services<a name="virtual_services"></a>

A virtual service is an abstraction of a real service that is either provided by a virtual node directly, or indirectly by means of a virtual router\. Dependent services call your virtual service by its `virtualServiceName`, and those requests are routed to the virtual node or virtual router that is specified as the provider for the virtual service\.

The following JSON represents a virtual service called `serviceB` that is provided by the virtual router called `serviceB`\.

```
{
    "meshName": "simpleapp",
    "spec": {
        "provider": {
            "virtualRouter": {
                "virtualRouterName": "serviceB"
            }
        }
    },
    "virtualServiceName": "serviceB"
}
```

If the above JSON is saved as a file, you can create the virtual service with the following AWS CLI command\.

```
aws appmesh create-virtual-router --cli-input-json file://serviceB-virtual-service.json
```

**Note**  
You must use at least version 1\.16\.120 of the AWS CLI with App Mesh\. To install the latest version of the AWS CLI, see [Installing the AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/installing.html) in the *AWS Command Line Interface User Guide*\.

For more information about creating virtual routers with the AWS CLI, see [create\-virtual\-router](https://docs.aws.amazon.com/cli/latest/reference/appmesh/create-virtual-router.html) in the *AWS Command Line Interface User Guide*\.
# Envoy image<a name="envoy"></a>

AWS App Mesh is a service mesh based on the [Envoy](https://www.envoyproxy.io/) proxy\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/app-mesh/latest/userguide/images/proxy.png)

You must add an Envoy proxy to the Amazon ECS task, Kubernetes pod, or Amazon EC2 instance represented by your App Mesh endpoint, such as a virtual node or virtual gateway\. App Mesh vends an Envoy proxy Docker container image and validate that this container image is patched with the latest vulnerability and performance patches\. App Mesh tests a new Envoy proxy release against the App Mesh feature set before making a new container image available to you\.

You can choose either a Regional image from the list below or an image from our [public repository](https://gallery.ecr.aws/appmesh/aws-appmesh-envoy) named `aws-appmesh-envoy`\.

**Important**  
Version `1.17` was a significant update to Envoy\. See [Updating/migrating to Envoy 1\.17](https://docs.aws.amazon.com/app-mesh/latest/userguide/1.17-migration.html) for more details\.
Version `1.20.0.1` or later is `ARM64` compatible\.
For `IPv6` support, Envoy version `1.13` or later is required\.
+ All [supported](https://docs.aws.amazon.com/general/latest/gr/appmesh.html) Regions other than `me-south-1` and `ap-east-1`, `eu-south-1`, `af-south-1`\. You can replace *Region\-code* with any Region other than `me-south-1` and `ap-east-1`, `eu-south-1`, `af-south-1`\. 

  ```
  840364872350.dkr.ecr.region-code.amazonaws.com/aws-appmesh-envoy:v1.22.0.0-prod
  ```
+ `me-south-1` Region:

  ```
  772975370895.dkr.ecr.me-south-1.amazonaws.com/aws-appmesh-envoy:v1.22.0.0-prod
  ```
+ `ap-east-1` Region:

  ```
  856666278305.dkr.ecr.ap-east-1.amazonaws.com/aws-appmesh-envoy:v1.22.0.0-prod
  ```
+ `eu-south-1` Region:

  ```
  422531588944.dkr.ecr.eu-south-1.amazonaws.com/aws-appmesh-envoy:v1.22.0.0-prod
  ```
+ `af-south-1` Region:

  ```
  924023996002.dkr.ecr.af-south-1.amazonaws.com/aws-appmesh-envoy:v1.22.0.0-prod
  ```
+ `Public repository`

  ```
  public.ecr.aws/appmesh/aws-appmesh-envoy:v1.22.0.0-prod
  ```

**Important**  
Only version v1\.9\.0\.0\-prod or later is supported for use with App Mesh\.

**Note**  
We recommend allocating 512 CPU units and at least 64 MiB of memory to the Envoy container\. On Fargate the lowest amount of memory that you can set is 1024 MiB of memory\.

**Note**  
All `aws-appmesh-envoy` image release versions starting from `v1.22.0.0` are built as a distroless Docker image\. We made this change so that we could reduce the image size and reduce our vulnerability exposure in unused packages present in the image\. If you are building on top of aws\-appmesh\-envoy image and are relying on some of the AL2 base packages \(e\.g\. yum\) and functionalities, then we suggest you copy the binaries from inside an `aws-appmesh-envoy` image to build a new Docker image with AL2 base\.  
Run this script to generate a custom docker image with the tag `aws-appmesh-envoy:v1.22.0.0-prod-al2:`  

```
cat << EOF > Dockerfile
FROM public.ecr.aws/appmesh/aws-appmesh-envoy:v1.22.0.0-prod as envoy

FROM public.ecr.aws/amazonlinux/amazonlinux:2
RUN yum -y update && \
    yum clean all && \
    rm -rf /var/cache/yum

COPY —from=envoy /usr/bin/envoy /usr/bin/envoy
COPY —from=envoy /usr/bin/agent /usr/bin/agent
COPY —from=envoy /aws_appmesh_aggregate_stats.wasm /aws_appmesh_aggregate_stats.wasm

CMD [ "/usr/bin/agent" ]
EOF

docker build -f Dockerfile -t aws-appmesh-envoy:v1.22.0.0-prod-al2 .
```

Access to this container image in Amazon ECR is controlled by AWS Identity and Access Management \(IAM\)\. As a result, you must use IAM to verify that you have read access to Amazon ECR\. For example, when using Amazon ECS, you can assign an appropriate task execution role to an Amazon ECS task\. If you use IAM policies that limit access to specific Amazon ECR resources, make sure to verify that you allow access to the Region specific Amazon Resource Name \(ARN\) that identifies the `aws-appmesh-envoy` repository\. For example, in the `us-west-2` Region, you allow access to the following resource: `arn:aws:ecr:us-west-2:840364872350:repository/aws-appmesh-envoy`\. For more information, see [Amazon ECR Managed Policies](https://docs.aws.amazon.com/AmazonECR/latest/userguide/ecr_managed_policies.html)\. If you're using Docker on an Amazon EC2 instance, then authenticate Docker to the repository\. For more information, see [Registry authentication](https://docs.aws.amazon.com/AmazonECR/latest/userguide/Registries.html#registry_auth)\.

We occasionally release new App Mesh features that depend on Envoy changes that have not been merged to the upstream Envoy images yet\. To use these new App Mesh features before the Envoy changes are merged upstream, you must use the App Mesh\-vended Envoy container image\. For a list of changes, see the [App Mesh GitHub roadmap issues](https://github.com/aws/aws-app-mesh-roadmap/labels/Envoy%20Upstream) with the `Envoy Upstream` label\. We recommend that you use the App Mesh Envoy container image as the best supported option\.
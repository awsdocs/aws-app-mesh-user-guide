# Getting Started with AWS App Mesh and Amazon ECS<a name="mesh-getting-started-ecs"></a>

This topic helps you to use AWS App Mesh with an existing set of microservice applications running on Amazon ECS\.

## Prerequisites<a name="mesh-gs-ecs-prerequisites"></a>

App Mesh supports microservice applications that use service discovery naming for their components\. For more information about service discovery on Amazon ECS, see [Service Discovery](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-discovery.html) in the *Amazon Elastic Container Service Developer Guide*\.

To use this getting started guide, you must have a microservice application running on Amazon ECS\. You must also have the following App Mesh resources, that you can create by completing the steps in the [Getting Started with AWS App Mesh](getting_started.md) guide:
+ A service mesh
+ Virtual nodes for each microservice in your application
+ Virtual routers and routes for each microservice in your application, except for virtual services that are provided by a virtual node directly
+ Virtual services for each microservice in your application

## Update Your Microservice Task Definitions<a name="mesh-gs-ecs-update-microservices"></a>

App Mesh is a service mesh based on the [Envoy](https://www.envoyproxy.io/) proxy\. After you create your service mesh, virtual nodes, virtual routers, routes, and virtual services, you must update the Amazon ECS task definitions for your microservices to be compatible with App Mesh\. Complete the steps in the following sections to update your services' task definitions to work with App Mesh\. Alternately, you can update your task definitions using the AWS Management Console\. For more information, see [Updating a Task Definition](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/update-task-definition.html)\.

### Proxy Configuration<a name="mesh-gs-ecs-proxyconfig"></a>

To configure your Amazon ECS service to use App Mesh, your service's task definition must have the following proxy configuration section\. All of the properties in the following example are required\. Some of the property values are also required, but some are *replaceable*\. Set the proxy configuration `type` to `APPMESH` and the `containerName` to `envoy`\. Set the following property values accordingly\.

`IgnoredUID`  
Envoy doesn't proxy traffic from processes that use this user ID\. You can choose any user ID that you want for this, but this ID must be the same as the `user` ID for the Envoy container in your task definition\. This matching allows Envoy to ignore its own traffic without using the proxy\. Our examples use `1337` for historical purposes\.

`ProxyIngressPort`  
This is the ingress port for the Envoy proxy container\. Set this value to `15000`\.

`ProxyEgressPort`  
This is the egress port for the Envoy proxy container\. Set this value to `15001`\.

`AppPorts`  
Specify any ingress ports that your application containers listen on\. In this example, the application container listens on port `9080`\. The port that you specify must match the port configured on the virtual node listener\.

`EgressIgnoredIPs`  
Envoy doesn't proxy traffic to these IP addresses\. Set this value to `169.254.170.2,169.254.169.254`, which ignores the Amazon EC2 metadata server and the Amazon ECS task metadata endpoint\. The metadata endpoint provides IAM roles for tasks credentials\. You can add additional addresses\.

```
"proxyConfiguration": {
	"type": "APPMESH",
	"containerName": "envoy",
	"properties": [{
			"name": "IgnoredUID",
			"value": "1337"
		},
		{
			"name": "ProxyIngressPort",
			"value": "15000"
		},
		{
			"name": "ProxyEgressPort",
			"value": "15001"
		},
		{
			"name": "AppPorts",
			"value": "9080"
		},
		{
			"name": "EgressIgnoredIPs",
			"value": "169.254.170.2,169.254.169.254"
		}
	]
}
```

### Application Container Envoy Dependency<a name="mesh-gs-ecs-envoy-dep"></a>

The application containers in your task definitions must wait for the Envoy proxy to bootstrap and start before they can start\. To ensure that this happens, you set a `dependsOn` section in each application container definition to wait for the Envoy container to report as `HEALTHY`\. The following code block shows an application container definition example with this dependency\. All of the properties in the following example are required\. Some of the property values are also required, but some are *replaceable*\.

```
{
	"name": "appName",
	"image": "appImage",
	"portMappings": [{
		"containerPort": 9080,
		"hostPort": 9080,
		"protocol": "tcp"
	}],
	"essential": true,
	"dependsOn": [{
		"containerName": "envoy",
		"condition": "HEALTHY"
	}]
}
```

### Envoy Container Definition<a name="mesh-gs-ecs-envoy"></a>

Your Amazon ECS task definitions or Kubernetes pod specs must contain the [App Mesh Envoy container image](envoy.md):
+ `111345817488.dkr.ecr.us-west-2.amazonaws.com/aws-appmesh-envoy:v1.11.1.0-prod`

All of the properties in the following example are required\. Some of the property values are also required, but some are *replaceable*\. The Envoy container definition must be marked as `essential`\. The virtual node name for the Amazon ECS service must be set to the value of the `APPMESH_VIRTUAL_NODE_NAME` property\. The value for the `user` setting must match the `IgnoredUID` value from the task definition proxy configuration\. In this example, we use `1337`\. The health check shown here waits for the Envoy container to bootstrap properly before reporting to Amazon ECS that the Envoy container is healthy and ready for the application containers to start\.

The following code block shows an Envoy container definition example\.

```
{
	"name": "envoy",
	"image": "111345817488.dkr.ecr.us-west-2.amazonaws.com/aws-appmesh-envoy:v1.11.1.0-prod",
	"essential": true,
	"environment": [{
		"name": "APPMESH_VIRTUAL_NODE_NAME",
		"value": "mesh/meshName/virtualNode/virtualNodeName"
	}],
	"healthCheck": {
		"command": [
			"CMD-SHELL",
			"curl -s http://localhost:9901/server_info | grep state | grep -q LIVE"
		],
		"startPeriod": 10,
		"interval": 5,
		"timeout": 2,
		"retries": 3
	},
	"user": "1337"
}
```

### Credentials<a name="credentials"></a>

The Envoy container requires AWS Identity and Access Management credentials for signing requests that are sent to the App Mesh service\. For Amazon ECS tasks deployed with the Amazon EC2 launch type, the credentials can come from the [instance IAM role ](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/instance_IAM_role.html)or from a [task IAM role](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-iam-roles.html)\. Amazon ECS tasks deployed with the Fargate launch type do not have access to the Amazon EC2 metadata server that supplies instance IAM profile credentials\. To supply the credentials, you must attach an IAM task role to any tasks deployed with the Fargate launch type\. The role doesn't need to have a policy attached to it, but for a task to work properly with App Mesh, the role must be attached to each task deployed with the Fargate launch type\. If a task is deployed with the Amazon EC2 launch type and access is blocked to the Amazon EC2 metadata server, as described in the *Important* annotation in [IAM Roles for Tasks](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-iam-roles.html), then a task IAM role must also be attached to the task\. 

### Example Task Definitions<a name="mesh-gs-ecs-task-def"></a>

The following example Amazon ECS task definitions show, in context, the snippets that you can merge with your existing task groups\. Substitute your mesh name and virtual node name for the `APPMESH_VIRTUAL_NODE_NAME` value and a list of ports that your application listens on for the proxy configuration `AppPorts` value\. All of the properties in the following examples are required\. Some of the property values are also required, but some are *replaceable*\. 

If you're running an Amazon ECS task as described in [Credentials](#credentials), you need an existing [task IAM role](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-iam-roles.html), as defined in the examples\. Select your Amazon ECS task launch type\.

------
#### [ Fargate launch type ]

**Example JSON for Amazon ECS task definition**  

```
{
   
   "family" : "appmesh-gateway",
   "memory" : "1024",
   "cpu" : "0.5 vCPU",
   "proxyConfiguration" : {
      "containerName" : "envoy",
      "properties" : [
         {
            "name" : "ProxyIngressPort",
            "value" : "15000"
         },
         {
            "name" : "AppPorts",
            "value" : "9080"
         },
         {
            "name" : "EgressIgnoredIPs",
            "value" : "169.254.170.2,169.254.169.254"
         },
         {
            "name" : "IgnoredUID",
            "value" : "1337"
         },
         {
            "name" : "ProxyEgressPort",
            "value" : "15001"
         }
      ],
      "type" : "APPMESH"
   },
   "containerDefinitions" : [
      {
         "name" : "appName",
         "image" : "appImage",
         "portMappings" : [
            {
               "containerPort" : 9080,
               "protocol" : "tcp"
            }
         ],
         "essential" : true,
         "dependsOn" : [
            {
               "containerName" : "envoy",
               "condition" : "HEALTHY"
            }
         ]
      },
      {         
         "name" : "envoy",
         "image" : "111345817488.dkr.ecr.us-west-2.amazonaws.com/aws-appmesh-envoy:v1.11.1.0-prod",
         "essential" : true,
         "environment" : [
            {
               "name" : "APPMESH_VIRTUAL_NODE_NAME",
               "value" : "mesh/meshName/virtualNode/virtualNodeName"
            }
         ],
         "healthCheck" : {
            "command" : [
               "CMD-SHELL",
               "curl -s http://localhost:9901/server_info | grep state | grep -q LIVE"
            ],
            "interval" : 5,
            "retries" : 3,
            "startPeriod" : 10,
            "timeout" : 2
         },
         "memory" : "500",
         "user" : "1337"
      }
   ],
   "requiresCompatibilities" : [ "FARGATE" ],
   "taskRoleArn" : "arn:aws:iam::123456789012:role/ecsTaskRole",
   "executionRoleArn" : "arn:aws:iam::123456789012:role/ecsTaskExecutionRole",
   "networkMode" : "awsvpc"
}
```

**Example JSON for Amazon ECS task definition with AWS X\-Ray**  
X\-Ray allows you to collect data about requests that an application serves and provides tools that you can use to visualize traffic flow\. Using the X\-Ray driver for Envoy enables Envoy to report tracing information to X\-Ray\. You can enable X\-Ray tracing using the [Envoy configuration](envoy.md)\. Based on the configuration, Envoy sends tracing data to the X\-Ray daemon running as a [sidecar](https://docs.aws.amazon.com/xray/latest/devguide/xray-daemon-ecs.html) container and the daemon forwards the traces to the X\-Ray service\. Once the traces are published to X\-Ray, you can use the X\-Ray console to visualize the service call graph and request trace details\. The following JSON represents a task definition to enable X\-Ray integration\.  

```
{
   
   
   "family" : "appmesh-gateway",
   "memory" : "1024",
   "cpu" : "0.5 vCPU",
   "proxyConfiguration" : {
      "containerName" : "envoy",
      "properties" : [
         {
            "name" : "ProxyIngressPort",
            "value" : "15000"
         },
         {
            "name" : "AppPorts",
            "value" : "9080"
         },
         {
            "name" : "EgressIgnoredIPs",
            "value" : "169.254.170.2,169.254.169.254"
         },
         {
            "name" : "IgnoredUID",
            "value" : "1337"
         },
         {
            "name" : "ProxyEgressPort",
            "value" : "15001"
         }
      ],
      "type" : "APPMESH"
   },
   "containerDefinitions" : [
      {
         "name" : "appName",
         "image" : "appImage",
         "portMappings" : [
            {
               "containerPort" : 9080,
               "protocol" : "tcp"
            }
         ],
         "essential" : true,
         "dependsOn" : [
            {
               "containerName" : "envoy",
               "condition" : "HEALTHY"
            }
         ]
      },
      {
         
         "name" : "envoy",
         "image" : "111345817488.dkr.ecr.us-west-2.amazonaws.com/aws-appmesh-envoy:v1.11.1.0-prod",
         "essential" : true,
         "environment" : [
            {
               "name" : "APPMESH_VIRTUAL_NODE_NAME",
               "value" : "mesh/meshName/virtualNode/virtualNodeName"
            },
            {
               "name": "ENABLE_ENVOY_XRAY_TRACING",
               "value": "1"
            }
         ],
         "healthCheck" : {
            "command" : [
               "CMD-SHELL",
               "curl -s http://localhost:9901/server_info | grep state | grep -q LIVE"
            ],
            "interval" : 5,
            "retries" : 3,
            "startPeriod" : 10,
            "timeout" : 2
         },
         "memory" : "500",
         "user" : "1337"
      },
      {
         "name" : "xray-daemon",
         "image" : "amazon/aws-xray-daemon",
         "user" : "1337",
         "essential" : true,
         "cpu" : "32",
         "memoryReservation" : "256",
         "portMappings" : [
            {
               "containerPort" : 2000,
               "protocol" : "udp"
            }
         ]
      }
   ],
   "requiresCompatibilities" : [ "FARGATE" ],
   "taskRoleArn" : "arn:aws:iam::123456789012:role/ecsTaskRole",
   "executionRoleArn" : "arn:aws:iam::123456789012:role/ecsTaskExecutionRole",
   "networkMode" : "awsvpc"
}
```

------
#### [ EC2 launch type ]

**Example JSON for Amazon ECS task definition**  

```
{
  "family": "appmesh-gateway",
  "memory": "256",
  "proxyConfiguration": {
    "type": "APPMESH",
    "containerName": "envoy",
    "properties": [
      {
        "name": "IgnoredUID",
        "value": "1337"
      },
      {
        "name": "ProxyIngressPort",
        "value": "15000"
      },
      {
        "name": "ProxyEgressPort",
        "value": "15001"
      },
      {
        "name": "AppPorts",
        "value": "9080"
      },
      {
        "name": "EgressIgnoredIPs",
        "value": "169.254.170.2,169.254.169.254"
      }
    ]
  },
  "containerDefinitions": [
    {
      "name": "appName",
      "image": "appImage",
      "portMappings": [
        {
          "containerPort": 9080,
          "hostPort": 9080,
          "protocol": "tcp"
        }
      ],
      "essential": true,
      "dependsOn": [
        {
          "containerName": "envoy",
          "condition": "HEALTHY"
        }
      ]
    },
    {
      "name": "envoy",
      "image": "111345817488.dkr.ecr.us-west-2.amazonaws.com/aws-appmesh-envoy:v1.11.1.0-prod",
      "essential": true,
      "environment": [
        {
          "name": "APPMESH_VIRTUAL_NODE_NAME",
          "value": "mesh/meshName/virtualNode/virtualNodeName"
        }
      ],
      "healthCheck": {
        "command": [
          "CMD-SHELL",
          "curl -s http://localhost:9901/server_info | grep state | grep -q LIVE"
        ],
        "startPeriod": 10,
        "interval": 5,
        "timeout": 2,
        "retries": 3
      },
      "user": "1337"
    }
  ],
  "requiresCompatibilities" : [ "EC2" ],
  "taskRoleArn" : "arn:aws:iam::123456789012:role/ecsTaskRole",
  "executionRoleArn" : "arn:aws:iam::123456789012:role/ecsTaskExecutionRole",
  "networkMode": "awsvpc"
}
```

**Example JSON for Amazon ECS task definition with AWS X\-Ray**  
X\-Ray allows you to collect data about requests that an application serves and provides tools that you can use to visualize traffic flow\. Using the X\-Ray driver for Envoy enables Envoy to report tracing information to X\-Ray\. You can enable X\-Ray tracing using the [Envoy configuration](envoy.md)\. Based on the configuration, Envoy sends tracing data to the X\-Ray daemon running as a [sidecar](https://docs.aws.amazon.com/xray/latest/devguide/xray-daemon-ecs.html) container and the daemon forwards the traces to the X\-Ray service\. Once the traces are published to X\-Ray, you can use the X\-Ray console to visualize the service call graph and request trace details\. The following JSON represents a task definition to enable X\-Ray integration\.  

```
{
  "family": "appmesh-gateway",
  "memory": "256",
   "cpu" : "1024",
  "proxyConfiguration": {
    "type": "APPMESH",
    "containerName": "envoy",
    "properties": [
      {
        "name": "IgnoredUID",
        "value": "1337"
      },
      {
        "name": "ProxyIngressPort",
        "value": "15000"
      },
      {
        "name": "ProxyEgressPort",
        "value": "15001"
      },
      {
        "name": "AppPorts",
        "value": "9080"
      },
      {
        "name": "EgressIgnoredIPs",
        "value": "169.254.170.2,169.254.169.254"
      }
    ]
  },
  "containerDefinitions": [
    {
      "name": "appName",
      "image": "appImage",
      "portMappings": [
        {
          "containerPort": 9080,
          "hostPort": 9080,
          "protocol": "tcp"
        }
      ],
      "essential": true,
      "dependsOn": [
        {
          "containerName": "envoy",
          "condition": "HEALTHY"
        }
      ]
    },
    {
      "name": "envoy",
      "image": "111345817488.dkr.ecr.us-west-2.amazonaws.com/aws-appmesh-envoy:v1.11.1.0-prod",
      "essential": true,
      "environment": [
        {
          "name": "APPMESH_VIRTUAL_NODE_NAME",
          "value": "mesh/meshName/virtualNode/virtualNodeName"
        },
        {
         "name": "ENABLE_ENVOY_XRAY_TRACING",
         "value": "1"
        }
      ],
      "healthCheck": {
        "command": [
          "CMD-SHELL",
          "curl -s http://localhost:9901/server_info | grep state | grep -q LIVE"
        ],
        "startPeriod": 10,
        "interval": 5,
        "timeout": 2,
        "retries": 3
      },
      "user": "1337"
    },
    {
      "name": "xray-daemon",
      "image": "amazon/aws-xray-daemon",
      "user": "1337",
      "essential": true,
      "cpu": 32,
      "memoryReservation": 256,
      "portMappings": [
        {
          "containerPort": 2000,
          "protocol": "udp"
        }
      ]
    }
  ],
  "requiresCompatibilities" : [ "EC2" ],
  "taskRoleArn" : "arn:aws:iam::123456789012:role/ecsTaskRole",
  "executionRoleArn" : "arn:aws:iam::123456789012:role/ecsTaskExecutionRole",
  "networkMode": "awsvpc"
}
```

------
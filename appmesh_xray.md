TASKDEFINITION
```
{
  "containerDefinitions": [
      {
          "name": "skills-product-ctn",
          "image": "148010479988.dkr.ecr.ap-northeast-2.amazonaws.com/skills-ecr-product:latest",
          "cpu": 512,
          "memory": 1024,
          "dependsOn": [
              {
                  "containerName": "envoy",
                  "condition": "HEALTHY"
              }
          ],
          "portMappings": [
              {
                  "name": "skills-product-port",
                  "containerPort": 8080,
                  "hostPort": 8080,
                  "protocol": "tcp",
                  "appProtocol": "http"
              }
          ],
          "essential": true,
          "logConfiguration": {
              "logDriver": "awslogs",
              "options": {
                  "awslogs-create-group": "true",
                  "awslogs-group": "/ecs/skills-product-td",
                  "awslogs-region": "ap-northeast-2",
                  "awslogs-stream-prefix": "ecs"
              }
          },
          "healthCheck": {
              "command": [
                  "CMD-SHELL",
                  "curl -fLs http://localhost:8080/health > /dev/null || exit 1"
              ],
              "interval": 5,
              "timeout": 2,
              "retries": 1,
              "startPeriod": 0
          }
      },
      {
          "name": "envoy",
          "image": "840364872350.dkr.ecr.ap-northeast-2.amazonaws.com/aws-appmesh-envoy:v1.12.3.0-prod",
          "user": "1337",
          "memoryReservation": 128,
          "essential": true,
          "environment": [
            {
              "name": "APPMESH_VIRTUAL_NODE_NAME",
              "value": "mesh/skills-mesh/virtualNode/product-service-vn"
            },
            {
              "name": "ENABLE_ENVOY_XRAY_TRACING",
              "value": "1"
            },
            {
              "name": "ENABLE_ENVOY_STATS_TAGS",
              "value": "1"
            }
          ],
          "logConfiguration": {
            "logDriver": "awslogs",
            "options": {
              "awslogs-group": "/ecs/microservice-demo",
              "awslogs-region": "ap-northeast-2",
              "awslogs-stream-prefix": "skills-product/envoy"
            }
          },
          "healthCheck": {
            "command": [
              "CMD-SHELL",
              "curl -s http://localhost:9901/server_info | grep state | grep -q LIVE"
            ],
            "interval": 5,
            "timeout": 2,
            "retries": 1,
            "startPeriod": 0
          }
      },
      {
          "logConfiguration": {
              "logDriver": "awslogs",
              "options": {
                  "awslogs-create-group": "true",
                  "awslogs-group": "/ecs/skills-product-td",
                  "awslogs-region": "ap-northeast-2",
                  "awslogs-stream-prefix": "skills-product/xray"
              }
          },
          "portMappings": [
              {
                  "containerPort": 2000,
                  "protocol": "udp"
              }
          ],
          "name": "xray-daemon",
          "image": "amazon/aws-xray-daemon",
          "cpu": 32,
          "memoryReservation": 256
      }
  ],
  "family": "skills-product-td",
  "taskRoleArn": "arn:aws:iam::148010479988:role/ecsTaskExecutionRole",
  "executionRoleArn": "arn:aws:iam::148010479988:role/ecsTaskExecutionRole",
  "networkMode": "awsvpc",
  "requiresCompatibilities": [
      "FARGATE"
  ],
  "cpu": "1024",
  "memory": "2048",
  "proxyConfiguration":{
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
          "value": "8080"
        },
        {
          "name": "EgressIgnoredIPs",
          "value": "169.254.170.2,169.254.169.254"
        }
      ]
  }
}
```
taskdef.yml
```
{
    "containerDefinitions": [
        {
            "name": "skills-product-ctn",
            "image": "<IMAGE>",
            "cpu": 512,
            "memory": 1024,
            "portMappings": [
                {
                    "name": "skills-product-port",
                    "containerPort": 8080,
                    "hostPort": 8080,
                    "protocol": "tcp",
                    "appProtocol": "http"
                }
            ],
            "essential": true,
            "dependsOn": [
                {
                    "containerName": "envoy",
                    "condition": "HEALTHY"
                }
            ],
            "logConfiguration": {
                "logDriver": "awslogs",
                "options": {
                    "awslogs-create-group": "true",
                    "awslogs-group": "/ecs/skills-product-td",
                    "awslogs-region": "ap-northeast-2",
                    "awslogs-stream-prefix": "ecs"
                }
            },
            "healthCheck": {
                "command": [
                    "CMD-SHELL",
                    "curl -fLs http://localhost:8080/health > /dev/null || exit 1"
                ],
                "interval": 5,
                "timeout": 2,
                "retries": 1,
                "startPeriod": 0
            }
        },
        {
            "name": "envoy",
            "image": "840364872350.dkr.ecr.ap-northeast-2.amazonaws.com/aws-appmesh-envoy:v1.12.3.0-prod",
            "cpu": 0,
            "memoryReservation": 128,
            "essential": true,
            "environment": [
                {
                    "name": "APPMESH_VIRTUAL_NODE_NAME",
                    "value": "mesh/skills-mesh/virtualNode/product-service-vn"
                },
                {
                    "name": "ENABLE_ENVOY_XRAY_TRACING",
                    "value": "1"
                },
                {
                    "name": "ENABLE_ENVOY_STATS_TAGS",
                    "value": "1"
                }
            ],
            "user": "1337",
            "logConfiguration": {
                "logDriver": "awslogs",
                "options": {
                    "awslogs-group": "/ecs/microservice-demo",
                    "awslogs-region": "ap-northeast-2",
                    "awslogs-stream-prefix": "skills-product/envoy"
                }
            },
            "healthCheck": {
                "command": [
                    "CMD-SHELL",
                    "curl -s http://localhost:9901/server_info | grep state | grep -q LIVE"
                ],
                "interval": 5,
                "timeout": 2,
                "retries": 1,
                "startPeriod": 0
            }
        },
        {
            "name": "xray-daemon",
            "image": "amazon/aws-xray-daemon",
            "cpu": 32,
            "memoryReservation": 256,
            "portMappings": [
                {
                    "containerPort": 2000,
                    "hostPort": 2000,
                    "protocol": "udp"
                }
            ],
            "essential": true,
            "logConfiguration": {
                "logDriver": "awslogs",
                "options": {
                    "awslogs-create-group": "true",
                    "awslogs-group": "/ecs/skills-product-td",
                    "awslogs-region": "ap-northeast-2",
                    "awslogs-stream-prefix": "skills-product/xray"
                }
            }
        }
    ],
    "family": "skills-product-td",
    "taskRoleArn": "arn:aws:iam::148010479988:role/ecsTaskExecutionRole",
    "executionRoleArn": "arn:aws:iam::148010479988:role/ecsTaskExecutionRole",
    "networkMode": "awsvpc",
    "status": "ACTIVE",
    "requiresAttributes": [
        {
            "name": "ecs.capability.execution-role-awslogs"
        },
        {
            "name": "com.amazonaws.ecs.capability.ecr-auth"
        },
        {
            "name": "com.amazonaws.ecs.capability.docker-remote-api.1.17"
        },
        {
            "name": "com.amazonaws.ecs.capability.docker-remote-api.1.21"
        },
        {
            "name": "com.amazonaws.ecs.capability.task-iam-role"
        },
        {
            "name": "ecs.capability.aws-appmesh"
        },
        {
            "name": "ecs.capability.container-health-check"
        },
        {
            "name": "ecs.capability.execution-role-ecr-pull"
        },
        {
            "name": "com.amazonaws.ecs.capability.docker-remote-api.1.18"
        },
        {
            "name": "ecs.capability.task-eni"
        },
        {
            "name": "com.amazonaws.ecs.capability.docker-remote-api.1.29"
        },
        {
            "name": "com.amazonaws.ecs.capability.logging-driver.awslogs"
        },
        {
            "name": "com.amazonaws.ecs.capability.docker-remote-api.1.19"
        },
        {
            "name": "ecs.capability.container-ordering"
        }
    ],
    "compatibilities": [
        "EC2",
        "FARGATE"
    ],
    "requiresCompatibilities": [
        "FARGATE"
    ],
    "cpu": "1024",
    "memory": "2048",
    "proxyConfiguration": {
        "type": "APPMESH",
        "containerName": "envoy",
        "properties": [
            {
                "name": "ProxyIngressPort",
                "value": "15000"
            },
            {
                "name": "AppPorts",
                "value": "8080"
            },
            {
                "name": "EgressIgnoredIPs",
                "value": "169.254.170.2,169.254.169.254"
            },
            {
                "name": "IgnoredUID",
                "value": "1337"
            },
            {
                "name": "ProxyEgressPort",
                "value": "15001"
            }
        ]
    },
    "registeredBy": "arn:aws:iam::148010479988:root"
}
```

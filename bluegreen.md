# taskdef.json
```
{
  "containerDefinitions": [
      {
          "name": "stress-ctn",
          "image": "<IMAGE>",
          "portMappings": [
              {
                  "name": "stress-ctn-8080-tcp",
                  "containerPort": 8080,
                  "hostPort": 8080,
                  "protocol": "tcp",
                  "appProtocol": "http"
              }
          ],
          "essential": true,
          "healthCheck": {
              "command": [
                  "CMD-SHELL",
                  "curl -f http://localhost:8080/health || exit 1"
              ],
              "interval": 30,
              "timeout": 5,
              "retries": 3
          }
      }
  ],
  "family": "skills-stress-td",
  "executionRoleArn": "arn:aws:iam::749692678017:role/ecsTaskExecutionRole",
  "networkMode": "awsvpc",
  "revision": 22,
  "status": "ACTIVE",
  "requiresAttributes": [
      {
          "name": "com.amazonaws.ecs.capability.logging-driver.awslogs"
      },
      {
          "name": "com.amazonaws.ecs.capability.docker-remote-api.1.24"
      },
      {
          "name": "ecs.capability.execution-role-awslogs"
      },
      {
          "name": "com.amazonaws.ecs.capability.ecr-auth"
      },
      {
          "name": "com.amazonaws.ecs.capability.docker-remote-api.1.19"
      },
      {
          "name": "com.amazonaws.ecs.capability.task-iam-role"
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
      }
  ],
  "placementConstraints": [],
  "compatibilities": [
      "EC2",
      "FARGATE"
  ],
  "requiresCompatibilities": [
      "FARGATE"
  ],
  "cpu": "1024",
  "memory": "3072",
  "runtimePlatform": {
      "cpuArchitecture": "X86_64",
      "operatingSystemFamily": "LINUX"
  },
  "registeredAt": "2023-10-10T09:42:46.925Z",
  "registeredBy": "arn:aws:iam::749692678017:root"
}
```
# buildspec.yml
```
version: 0.2
phases:
  pre_build:
    commands:
      - aws ecr get-login-password --region ap-northeast-2 | docker login --username AWS --password-stdin 749692678017.dkr.ecr.ap-northeast-2.amazonaws.com
      - STRESS_URI=749692678017.dkr.ecr.ap-northeast-2.amazonaws.com/stress-ecr
      - COMMIT_ID=$(echo $CODEBUILD_RESOLVED_SOURCE_VERSION | cut -c 1-7)
      - echo $COMMIT_ID
      - LATEST=${COMMIT_ID:=latest}
  build:
    commands:
      - docker build -t $STRESS_URI:$LATEST ./stress/
  post_build:
    commands:
      - docker push $STRESS_URI:$LATEST
      - echo "$(jq .containerDefinitions[].image=\"$STRESS_URI:$LATEST\" ./stress/taskdef.json)" > taskdef.json
      - mv stress/appspec.yml ./
artifacts:
  files:
    - appspec.yml
    - taskdef.json
```
# appspec.yml
```
version: 0.0

Resources:
  - TargetService:
      Type: AWS::ECS::Service
      Properties:
        TaskDefinition: "<TASK_DEFINITION>"
        LoadBalancerInfo:
          ContainerName: "stress-ctn"
          ContainerPort: 8080
```

# 코드코밋 커밋id를 ecr태그로 사용하기
buildspec.yml
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
```
# ECS 롤링업데이트에 필요한 imagedefinitions. buildspec.yml에 추가.
추가할 시 빌드에서 수정한 컨테이너로 ecs의 task의 컨테이너를 교체한다
```
      - printf '[{"name":"stress-ctn","imageUri":"%s"}]' $STRESS_URI:$LATEST > imagedefinitions.json
artifacts:
  files: imagedefinitions.json
```

---
layout: post
title: Github Actions로 CICD 파이프라인 만들기(Feat. 람다&ECR)
subtitle: 백엔드 파이프라인 만들기
tags: [AWS, Github_Actions, CICD]
comments: true
---

## Intro 
이번에 이직한 회사에 가자마자 기존의 아키텍처를 먼저 확인해보았다. 
아직 아키텍처도, CICD도, 제대로 배포도 한 번 안(못) 해온게 눈에 보였다. 거기다가 실행코드와 인프라가 뒤섞인 레포도 있었다. 
물론 기존에 계시던 분들이 열심히 배우고, 연구해서 만들어 놓은 것이었지만, 조금 더 편리한 방법이 있을 것 같다는 생각이 들었다. 

---

## 구조와 Task 파악

전체 구조를 파악하니 
- AWS 클라우드 기반의 Lambda함수와 ECS를 기반으로 백엔드가 구성되어 있고, 
- 프론트가 Amplify로 되어 있는 것을 확인할 수 있었다. 

그래서 결국 컨테이너 내의 실행코드를 어떻게 모으고, 관리하고 배포버전을 선택할 것인가를 관건이라고 생각해서 어떤 방식으로 배포를 하도록 되어 있는지 살펴보니, 기존은 코드와 인프라 IaC가 하나의 레포에서 관리되어 dev의 코드가 완성되면, prod 레포를 덮어쓰는 형태였다. 

그런데 이러한 방법 때문에 아래와 같은 몇가지 문제점이 생겼다. 
1. 레포를 단순히 덮어쓰는 방법으로 코드의 버전관리가 되지 않는다. 
2. 실행코드만 바뀌었을 뿐인데, 인프라를 포함한 모든 코드가 전부 prod 레포를 다 덮어써야 한다. 
3. dev 환경에서 prod와 동일한 인프라를 가지고 있어서, dev에서 인프라 변경과 같은 시도를 하기 어렵다. 

등등.. 

## 결정

따라서 이 인프라 IaC와 실행 코드를 분리하여, CICD 파이프라인을 통해 관리하기로 결정했다. 

### 고려해야 할 부분
이와 함께 고려해야 할 부분은 2가지가 있었는데, 첫번째는 **많은 컨테이너들을 어떻게 하면 효율적으로 관리할 것인가,** 두번째는 **백엔드 개발자는 인프라가 아닌 코드만을 고민했으면 좋겠다** 라는 점이었다. 

따라서 백엔드 개발자가 git push를 하면 바로 인프라로 적용되었으면 좋겠다고 생각했고, 언제든 컨테이너(lambd함수)를 생성, 롤백 할 수 있어야 한다고 생각해, <u>lambda함수를 이미지 기반으로 관리</u>하기로 결정했다. 그리고 람다함수에서 사용할 <u>이미지는 Amazon ECR 서비스를 통해 관리</u>하기로 했다.

<br />
<img src="https://github.com/yrkimyy/yrkimyy.github.io/blob/main/assets/img/lambdafunction.png?raw=true" width="90%" />

<br />


더불어 회사에서 Github을 사용해서 레포를 관리하고 있었기 때문에 편리성을 위해 Github Actions를 CI툴로 사용해, 다음과 같은 구조를 만들었다. 

```yaml
      Github Repo - docker build - docker push - ECR - Lambda Function
```

이러한 구조는 Github Actions을 통해 쉽게 작성할 수 있었는데, 아래와 같이 ECS actions 템플릿을 약간 변형하여 작성하였다. 

<img src="https://github.com/yrkimyy/yrkimyy.github.io/blob/main/assets/img/actions.png?raw=true" width="100%" />


물론 전체 Actions 템플릿은 총 4개의 파트로 만들어져 있지만, ECR까지 이미지를 빌드해서 푸시하는 과정은 1개의 단계만 사용하면 되었다. 

```yaml
 This workflow will build and push a new container image to Amazon ECR,
# and then will deploy a new task definition to Amazon ECS(여긴 진행하지 않음)
when there is a push to the "main" branch.

 1. Create an ECR repository to store your images.
    For example: `aws ecr create-repository --repository-name my-ecr-repo --region us-east-2`.
    Replace the value of the `ECR_REPOSITORY` environment variable in the workflow below with your repository's name.
    Replace the value of the `AWS_REGION` environment variable in the workflow below with your repository's region.
```
<br />

따라서 아래와 같이 템플릿을 수정하여 브랜치에 코드 변경사항을 푸시 할 때마다 ecr에 이미지를 빌드하여 푸시 할 수 있는 파이프라인을 만들 수 있었다. 

```yaml
name: Deploy to Amazon ECS

on:
  push:
    branches: [ "main" ]

env:
  AWS_REGION: MY_AWS_REGION                   # set this to your preferred AWS region, e.g. us-west-1
  ECR_REPOSITORY: MY_ECR_REPOSITORY           # set this to your Amazon ECR repository name

permissions:
  contents: read

jobs:
  ...
    - name: Login to Amazon ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v1

    - name: Build, tag, and push image to Amazon ECR
      id: build-image
      env:
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        IMAGE_TAG: latest
      run: |
        # Build a docker container and
        # push it to ECR so that it can
        # be deployed to ECS.
        docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
        echo "image=$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG" >> $GITHUB_OUTPUT
```
<br />


## 이슈 발생
위의 과정을 통해 코드를 변경할 때마다, ECR에 제대로 이미지가 들어가는 것을 확인하였다. 그러나 문제는 lambda 함수에서 ECR URI를 `이미지 주소:latest`로 지정하였고, 새로운 이미지에 latest 태그도 잘 붙는 것을 확인하였으나, lambda 함수에는 적용되지 않는 것을 발견하였다. 즉, lambda 함수에 이미지를 매번 새로 넣어줘야 한다는 이슈가 생겼다. 


## 이슈 해결
결국 ECR 이미지 레포에 올라온 새로운 이미지를 Lambda 함수에 배포할 방법이 있을 거라고 생각했다. 그리고 ECS 템플릿에서도 ECR -> ECS까지 배포가 가능하니, 이와 비슷한 방법을 도입했다. jobs의 마지막이 docker push 였으므로, docker push가 끝나고 나서 받는 새로운 ECR 이미지 주소를 lambda 함수에 넣어주는 커맨드를 찾았고, 이를 actions yaml에 추가해서 문제를 해결했다. 

```sh
aws lambda update-function-code --function-name $FUNCTION_NAME \
--image-uri $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
```
<br />

---

## 마무리
데브옵스 엔지니어는 프론트엔드/백엔드 개발자들이 일을 조금 더 효율적으로 할 수 있게 만들어주는 측면이 분명 있다. 그것이 프로덕트를 잘 전달하기 위함도 있고, 최대한 휴먼에러를 줄이기 위함도 있다. 하지만 이 파이프라인을 구축하면서 그것을 몸소 느끼게 되었다. ("와 진짜 편해졌어요" 라는 말을 들었을 땐 얼마나 뿌듯하던지)

이전에는 백엔드 개발자가 lambda 함수를 직접 만지면서 배포를 진행했기 때문에 분명 사람이 만들어 내는 에러에 대한 리스크가 존재했다. 그렇지만 이제는 코드를 작성하고 git push 외에는 개발자가 신경써야 할 부분이 줄었고 자동화로 인해 효율성이 증가 했다. 또한 굉장히 많은 lambda 함수가 있었는데, 함수마다의 실행코드를 이미지로 관리하니 롤백과 업데이트도 조금 더 수월해졌다.

<br />



---
layout: post
title: Lambda 함수 사용해서 슬랙 봇 만들기
subtitle: slack, github API 및 웹훅 사용기
# cover-img: /assets/img/path2.jpg
tags: [AWS, Integration]
---


## 개요
이번 해는 유독 람다 함수와 slack, discord, github 웹훅/API를 이용한 작은 기능을 만들어야 할 때가 많았다. 
그래서 SAM을 이용해서 Lambda 함수 사용하는 방법에 대해서 이야기 해보려고 한다. 매우 짧고 쉬울 예정.

<br />

## SAM으로 Lambda 함수 만들기
SAM은 AWS Serverless Application Model의 줄임말로, 서버리스 애플리케이션을 빌드할 수 있는 프레임워크다. 이전에는 Serverless를 써봤는데, Serverless와 SAM은 비슷하고 레거시에는 Serverless가 더 많을 거지만, 나는 SAM이 편하게 느껴져서, SAM을 선호한다. 

이를 시작하기 위해서는 SAM CLI가 필요하니, [설치하자](https://docs.aws.amazon.com/ko_kr/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html)


설치를 하고 나서 터미널에서 다음과 같이 `sam init`을 치면 템플릿을 선택할 수 있다. 

![](https://woolen-sandwich-ed7.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F2cc55a47-f355-48d8-af49-62a71a15d560%2FScreen_Shot_2022-09-10_at_20.32.15.png?table=block&id=5ea240a4-c1e1-40d6-a83f-a86dc006adaa&spaceId=85d16850-aaa6-4355-8ad2-a4fc90c8d02a&width=2000&userId=&cache=v2)


내가 주로 쓰는 템플릿은 1,3,5 정도 였는데, 나머지는 모르지만 무엇을 선택하든 CloudFormation에서 스택을 생성하고, 애플리케이션 내에 람다 함수를 생성한다. 

Lambda 함수는 기본적으로 컨테이너를 실행하는 것과 같은 원리를 가지며, 여기에 다른 리소스들을 트리거나 이벤트 등의 형식으로 부착해서 사용할 수 있다. 이는 template.yaml에 작성해서 빌드를 해도 되고, 람다 함수에서 직접 콘솔을 이용해서 작업을 해도 되지만, 기본적으로 IaC가 더 관리하기 편하다고 생각한다. 

그리고 처음 `sam init` 을 통해 템플릿을 선택하고 나면, 다음과 같은 코드를 볼 수 있다. 

```js
exports.lambdaHandler = async (event, context) => {

// 여기에 코드를 작성하세요. 

}

```

여기서 중요한 것은 event라는 파라미터로 전달되는 객체인데, (템플릿에 따라 이 파라미터가 없을 수도 있다.) 이 lambda 함수가 받게 되는 트리거가 되는 어떤 상태나 액션을 의미한다. 

그래서 만약 여러분이 github의 푸시 이벤트를 트리거 삼아서 이 람다 함수를 실행한다면, 이벤트 객에체는 github의 푸시 이벤트가 들어올 거다. 

따라서 여러분이 어떤 다른 애플리케이션 혹은 서비스의 상태변경이나 액션을 이 람다함수의 트리거로 쓰고 싶다면, 그 트리거에 대한 정보는 이 이벤트 객체에 담기게 된다. 


필요하다면, event 객체를 콘솔에 찍어 테스트를 해보면 좋다. 

<br />

나의 경우에는 주로 github에서 웹훅으로 정보를 받아서, slack으로 쏘는 형태의 기능을 많이 제작했다.

```
github webhook - lambda - slack
```

따라서 lambda에 꼭 API Gateway를 붙여서 해당 주소를 아래와 같이 github webhook에 등록해서 사용했다. 이러면 람다 함수가 이벤트 트리거에 따라 실행된다. 

![](https://blog.kakaocdn.net/dn/pMYKx/btrLPnPhXnz/ixEBGqr1sbzjWZYGqGZINk/img.png)

그러면 이 이벤트마다 무엇을 하고 싶은지를 람다 함수 안에 코드로 작성하면 된다. 나는 주로 슬랙 메세지를 전송하는 일이었기 때문에, 슬랙 API를 사용하여 슬랙 봇을 통해 전송하는 일을 했다. 

슬랙봇을 생성하면, 보통 아래와 같이 슬랙봇의 webhook 주소를 생성할 수 있다. 이는 채널마다 생성할 수 있기 때문에, 필요한 채널에 맞춰 작업하면 된다. 그리고 샘플인 curl 요청처럼 단순 post 요청을 하면 된다. 

![](https://blog.kakaocdn.net/dn/bpeoLg/btrLQx5cka5/6U4zhr2QKbKTXHtRzVGOF1/img.png)


생각보다 구조와 원리를 알면 그렇게 어렵지 않다. 물론 필요한 코드작업이 어떠냐에 따라 조금 다를 수 있지만 이정도의 통합작업은 어렵지 않다. 

다만 얼마나 람다의 실행속도를 빠르게 할 것이냐는 또 다른 문제이다. 어떤 경보를 람다와 슬랙을 통해 받아야 한다면, 실행 속도가 관건일 수 있다. 비용 또한 실행 시간에 따라서 비용이 책정되기 때문에 이 또한 마찬가지다. 

하지만 자피어가 할 수 있는 것 이상의 기능이지만 간단한 봇이 필요한거라면, 이정도로 충분하다고 생각한다. 


<br />
<br />


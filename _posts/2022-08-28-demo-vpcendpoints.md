---
layout: post
title: 프라이빗 서브넷의 ECS와 ECR, NAT gateway 없이 연결하기
subtitle: VPC Endpoint 사용하면 비용이 77% 줄어든다고?!?!
cover-img: /assets/img/path2.jpg
tags: [AWS]
---

### 개요
프라이빗 서브넷은 외부(인터넷)에서 직접적으로 접근할 수 없는 네트워크 영역으로, 데이터 보안을 위해 RDS와 같은 리소스를 위치 시키기도 합니다. 하지만, 해당 서브넷에 인터넷 연결이 필요할 때가 있습니다. 그럴 때, NAT Gateway를 통해 VPC 밖의 영역과 통신할 수 있도록 설계할 수 있습니다. 

ECR과 ECS를 통해 배포하기 위한 아키텍처를 구성할 때, 컴퓨팅 관련 리소스(fargate)를 프라이빗 서브넷에 위치시키라는 요구사항이 있었습니다. 

이에 따라 아키텍처는 아래와 같은 형태와 비슷하게 그려져야 했습니다. -- [ 출처 ](https://aws.amazon.com/ko/blogs/korea/setting-up-aws-privatelink-for-amazon-ecs-and-amazon-ecr/)

![](https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2019/01/24/Picture1-1.png)

기본적으로 VPC 마법사를 사용하면, NAT Gateway와 Internet Gateway가 디폴트로 생성되기 때문에, 어렵지 않게 구성할 수 있었습니다. 

<br />

하지만 이 [블로그](https://www.1strategy.com/blog/2021/07/13/cut-aws-costs-with-vpc-endpoints/)를 보니, 아주 혹하는 문구가 있었습니다. 

> The engineers found that 77TB of NAT gateway bandwidth had been used, which cost $3,562.02. Had they provisioned an Interface Endpoint for ECR, this traffic could have skipped the NAT gateway, which is 4.5x more expensive. The cost for using the Interface Endpoint instead would be $794.08, which is more than 77% less expensive. It’s a painless architectural change with the potential for big savings

NAT Gateway보다 VPC Endpoint를 쓰면 77%의 비용을 줄일 수 있다니! 

어떤 상황에서 진행한 결과인지 알지 못했지만, AWS 네트워크 망을 사용하는 리소스(혹은 서버리스 리소스)와 프라이빗 서브넷 사이에서 다량의 데이터가 오가는 구조에서는 의미 있는 비용절감을 가져다 줄 수 있을 것이라고 생각했습니다.

따라서 VPC endpoint를 직접 사용해보기 위해서 아키텍처 구현을 시작했습니다. 

<br />

### 구현 시작
구현을 위해서 이 [문서](https://aws.amazon.com/ko/blogs/korea/setting-up-aws-privatelink-for-amazon-ecs-and-amazon-ecr/)를 참고했습니다. 


우선 문서를 따라서 api, dkr, s3에 대한 vpc endpoint를 생성해보았으나, 아키텍처는 예상했던 대로 작동하지 않았습니다. 

1) 로컬에서는 이미지 빌드와 컨테이너 실행에 문제가 없었으나, 
2) fargate에는 어떤 컨테이너 실행 로그를 확인 할 수 없었고, 
3) pulling image 시도를 한 후, draining 되었습니다. 

이를 통해 ecr에서 이미지가 제대로 불러와지지 않는 것이 문제임을 확인했습니다. 
여기서 몇가지 의문에 대해서 답을 확실히 하고 싶었습니다. 

#### ecr.api, ecr.dkr, s3에 대한 vpc endpoint가 왜 필요할까?
확인해보니 ecr.api는 AWS 내부에서 ecr 관련 api콜을 위해서 사용되고, ecr.dkr은 Docker 클라이언트 명령에 사용하는 엔드포인트이며, ecr은 이미지를 S3에 저장하기 때문에 필요했다는 것을 알 수 있었다. 

즉, 다시 말하면 ecr에서 이미지를 불러올 때 필요한 리소스 각각마다 vpc 엔드포인트가 필요함을 확인했습니다. 이에 따라 awslogs, auth(ssm)의 vpc endpoint를 추가적으로 생성하였습니다. 

여기서 또다시 궁금한 점이 생겼습니다. 
#### endpoint를 만드는 것만으로 어떻게 네트워크가 연결될까? 
라우트 테이블이 있는 것도 아니고, endpoint는 일종의 말단인 셈인데 어떻게 네트워크망이 만들어지고 트래픽이 오고 가는지 궁금해졌습니다. 힌트는 
[참고문서](https://docs.aws.amazon.com/ko_kr/AmazonECR/latest/userguide/vpc-endpoints.html#ecr-vpc-endpoint-considerations)를 통해서 얻을 수 있었습니다. 

> VPC 엔드포인트는 프라이빗 IP 주소를 통해 Amazon ECR APIs에 비공개로 액세스할 수 있는 기술인 AWS PrivateLink로 구동됩니다.

<br />

### 그렇다면 실제로 비용이 줄어드나?

그리고 이렇게 구현에 성공한 뒤, 실제로 77%의 비용을 줄일 수 있는지 궁금했습니다. 단, 실제 트래픽이 없는 상태에서 비용 실험은 하기 어려워 AWS Calculator를 기준으로 확인해보았습니다.

총 5개의 VPC endpoint와 NAT Gateway의 비용(가용영역 1개 기준)은 월 **10TB**의 처리데이터 기준으로 VPC endpoint 사용 시 대략 70% 정도의 비용을 줄일 수 있는 것으로 확인 했습니다. **1TB**에는 50%를 절약할 수 있었으나, **100GB** 단위에서는 비용 차이가 거의 발생하지 않았습니다. 

다시 말해, ECR의 이미지를 가용영역 1개 단위 기준, ECS의 컴퓨팅 머신에서 비용을 절감하며 사용하기 위해서 VPC endpoint를 사용한다면 최소 월 100GB 이상의 이미지를 사용해야 의미가 있다고 볼 수 있습니다.

그리고 다시 혹하게 만들었던 블로그를 보니, 해당 시나리오에서도 image가 계속 pulling 되어 77TB의 데이터를 기준으로 이야기 한 것을 보아 분명 사용하는 네트워크 트래픽이 클 수록 비용 절감에 VPC endpoint 사용이 유리하다는 것을 알 수 있었습니다. 


<br />

### 마무리
여기에는 적지 않았지만 VPC endpoint를 이해하기 위해서 정말 많은 문서를 읽었고, 새로운 정보(AWS Hyperplane, PrivateLink, Bastion Host 등)를 알아가는 기쁨과 함께 VPC endpoint와 NAT Gateway의 비교를 해볼 수 있는 좋은 기회 였다고 생각합니다. 

비용 절감은 실제 트래픽으로 확인해보진 못했지만, Calculator 계산 상으로 확인 해볼 수 있었고, VPC endpoint를 통해 ECR로부터 이미지 불러오는데 성공했습니다. 

그리고 마지막으로 VPC endpoint 생성하는 시도를 성공한 후, 다시 NAT Gateway로 구현을 시작해서, 단순한 서버이긴 했지만 서버가 원활하게 돌아가는 중에 VPC endpoint로 아키텍처를 변경하는 경험을 해보았던 것이 좋았습니다. 


<br />

위와 같은 경험이 다른 분들께도 도움이 되기를 바라면서,
혹시 제 우당탕탕 블로그 글에 **오류나 잘못된 부분이 있다면 꼭 알려주시기 바랍니다.**




---
layout: post
title: Understanding VPC links in Amazon API Gateway private integrations
subtitle: VPC links, API gateway, Private integrations 이해하기
cover-img: /assets/img/path2.jpg
share-img: /assets/img/path2.jpg
tags: [AWS]
---

이 포스트는 Jose Eduardo Montilla Lugo, Security Consultant, AWS가 작성한 내용입니다.

<br />
VPC link는 VPC 내의 프라이빗 리소스에 API 라우트를 연결하도록 하는 아마존 API Gateway의 리소스 입니다. VPC link는 API를 위한 통합 엔드포인트처럼 작동하고, 네트워킹 리소스의 가장 윗단 인 추상 레이어입니다. 이는 프라이빗 통합 설정을 간단하게 하는데 도움을 줍니다. 

이 포스트는 VPC links를 가능하게 만드는 로우레벨의 기술에 대해서 이야기 합니다. 더 나아가, VPC link가 REST APIs, HTTP APIs를 위해 만들어지면 표면 아래에서 어떤 일이 벌어지는지 설명합니다. 

이런 디테일을 이해하면, 각 유형에서 제공하는 이점과 기능을 더 잘 평가하는데 도움이 됩니다. 또한 이는 API Gateway의 APIs를 디자인할 때, 더 좋은 아키텍처적 결정을 만드는데 도움이 됩니다. 

이 아티클은 APT Gateway를 통해 API를 만드어본 경험이 있음을 가정하고 작성되었습니다. 가장 주된 목적은 프라이빗 통합을 가능하게 하는 기술에 대한 깊은 설명을 제공하는데 있습니다. 프라이빗 통합을 위한 API gateway APIs를 생성하는데 필요한 더 많은 정보는 [Amazon API Gateway documentation](https://docs.aws.amazon.com/apigateway/index.html)을 참조하세요. 

<br />

## Overview 
### AWS Hyperplane and AWS PrivateLink

VPC links에는 2종류의 타입이 존재합니다. 이는 REST APIs를 위한 VPC links와 HTTP APIs를 위한 VPC links 입니다. 

둘 다 VPC내의 리소스에 접근하는 기능을 제공합니다. 그리고 이 기능들은 내부 AWS 서비스인 [AWS Hyperplane](https://www.youtube.com/watch?t=2065&v=8gc2DgBqo9U&feature=youtu.be) 위에 만들어집니다. Hyperplane은 내부 네트워크 가상화 플랙폼이며, inter-VPC connectivity, 즉 VPC간 연결과 VPC간의 라우팅을 지원합니다. 

내부적으로 Hyperplane은 AWS 서비스가 고객 VPC의 리소스에 연결하는 데 사용하는 다중 네트워크 구성을 지원합니다.

그러한 구성 중 하나가 AWS PrivateLink이며, API Gateway에서 프라이빗 APIs 및 프라이빗 통합을 지원하는데 사용됩니다. 

[AWS PrivateLink](https://aws.amazon.com/ko/privatelink/?privatelink-blogs.sort-by=item.additionalFields.createdDate&privatelink-blogs.sort-order=desc)를 사용하면, AWS 내에서의 네트워크 트래픽은 그대로 유지하면서, 다른 고객에 의해서 호스팅 된 서비스와 AWS 서비스에 접근할 수 있도록 합니다. 서비스는 프라이빗 IP주소로 노출 되기 때문에, 모든 통신은 사실상 로컬 혹은 프라이빗한 상태로 진행됩니다. 이렇게 하면 퍼블릿 인터넷으로 데이터가 노출되는 일이 줄어듭니다. 


[AWS PrivateLink](https://docs.aws.amazon.com/vpc/latest/privatelink/privatelink-share-your-services.html)에서, VPC endpoint 서비스는 다른 AWS 계정이 자신의 VPC에서 노출된 서비스에 접근할 수 있도록 하는 서비스 공급자 측의 네트워킹 리소스 입니다. VPC endpoint 서비스는 소비자 측의 VPC내의 Elastic Network Interface(ENI, 탄력적 네트워크 인터페이스)를 통해 가상의 연결을 확장함으로써 공급자의 VPC내에 위치한 특정 서비스를 공유할 수 있습니다.

[Interface VPC endpoint](https://docs.aws.amazon.com/vpc/latest/privatelink/create-interface-endpoint.html)는 서비스 소비자 안의 네트워킹 리소스로, 하나 혹은 그 이상의 elastic network interfaces의 집합입니다. 이것은 AWS PrivateLnk에서 제공하는 서비스에 연결할 수 있는 진입점입니다. 

<br />

### Comparing private APIs and private integrations (프라이빗 APIs와 프라이빗 통합 비교하기)

프라이빗 APIs는 프라이빗 통합과는 다릅니다. 둘 다 AWS PrivateLink를 사용하지만 서로 다른 방식으로 사용됩니다. 

프라이빗 API는 오직 VPC를 통해서만 API 엔드포인트에 연결 할 수 있습니다. 프라이빗 API는 VPC내의 클라이언트, 혹은 VPC에 네트워크 연결이 되어 있는 클라이언트에서만 접근 가능합니다. 

예를 들어 AWS Direct Connect를 통한 온프레미스 클라이언트에서, 프라이빗 API를 활성하기 위해서는 AWS PrivateLink 연결이 고객의 VPC와 API Gateway의 VPC에 설정되어 있어야 합니다. 

클라이언트는 API Gateway에 비공개로 요청을 라우팅하는 VPC endpoint 인터페이스를 통해 프라이빗 APIs에 연결합니다. 그 트래픽은 고객측의 VPC로부터 시작되어, AWS PrivateLink를 통해, API Gateway의 AWS account로 흐릅니다.

![Consumer connected to provider throught PrivateLink](https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2021/08/10/Screen-Shot-2021-08-10-at-1.33.13-PM.png)

<br />
API Gateway를 위한 VPC endpoint가 활성화 되면, VPC 내부에서 만들어진 API Gateway API에 대한 모든 요청을 VPC endpoint을 통하게 됩니다. 이것은 프라이빗 API와 퍼블릭 API경우 모두 해당합니다. 

퍼블릭 API는 여전히 인터넷에서 접근할 수 있고, 프라이빗 API는 오직 VPC endpoint 인터페이스에서만 액세스 할 수 있습니다. 현재(2021년 8월 10일 기준) REST API는 프라이빗으로만 설정할 수 있습니다.

프라이빗 통합은 VPC 내의 백엔드 엔드포인트가 존재하고, 공개적으로 접근할 수 없음을 의미합니다. 프라이빗 통합으로 API Gateway는 퍼블릭 인터넷에 리소스를 노출 시키지 않고 VPC 내부의 백엔드 엔드포인트에 접근할 수 있습니다. 

프라이빗 통합은 API Gateway와 타켓 VPC 리소스 간의 연결을 캡슐화 하기 위해서 VPC Links 사용합니다. 

VPC links는 고급 네트워크 구성을 할 필요 없이 VPC 내의 HTTP/HTTPS 리소스에 접근할 수 있게 합니다. REST API와 HTTP API 모두 프라이빗 통합을 지원하지만, REST API를 위한 VPC link만 내부적으로 AWS PrivateLink를 사용합니다. 


<br />

## VPC links for REST APIs
만약 REST API를 위한 VPC link를 생성한다면, VPC 엔드포인트 역시 생성되며, AWS 계정을 서비스 공급자로 만듭니다. 이 경우, 서비스 소비자는 API Gateway의 계정입니다. 

API Gateway 서비스는 VPC link가 생성되는 리전의 계정에 인터페이스 VPC endpoint를 생성합니다. 이는 API Gateway VPC로부터 여러분의 VPC로 AWS PrivateLink를 설정합니다. VPC 엔드포인트와 VPC link의 대상은 타겟 엔드포인트로 요청을 전달하는 네트워크 로드밸런서(NLB)입니다. 

![VPC Link for REST APIs](https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2021/08/10/Screen-Shot-2021-08-10-at-1.35.25-PM.png)

<br />

AWS PrivateLink 연결을 설정하기 전에, 서비스 공급자는 연결 요청을 반드시 승인해야 합니다. API Gateway 계정으로부터의 요청들은 VPC link 생성 프로세스 내에서 자동으로 승인됩니다. 그 이유는 각 리전에 API Gateway를 제공하는 AWS 계정들이 VPC 엔드포인트 서비스의 허용 목록에 속해 있기 때문입니다. 

네트워크 로드밸런서(NLB)가 엔드포인터 서비스와 연결되면, 타겟(대상)으로 향하는 트래픽은 NLB를 출처(소스)로 삼습니다. 타겟(대상)은 서비스 소비자의 IP 주소가 아니라, NLB의 프라이빗 IP주소를 받습니다.

이것은 두 가지 이유로 NLB 뒤에 존재하는 인스턴스들의 보안그룹을 설정할 때 도움이 됩니다. 

첫번째, 여러분은 서비스에 연결하는 VPC의 IP 주소들을 알 수 없습니다. 두번째로, NLB의 탄력적 네트워크 인터페이스(ENI)에는 어떤 보안그룹도 연결되어 있지 않습니다. 이는 타겟의 보안그룹에서 서비스 소비자의 IP주소를 소스로 사용할 수 없음을 의미합니다. 더 학습하기 위해서는 [NLB에 할당된 내부 IP주소를 알아내는 방법](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/target-group-register-targets.html)에 대해서 읽어보세요. 


<br />

프라이빗 통합으로 프라이빗 API를 생성하기 위해서, 두 개의 AWS PrivateLink 연결을 설정해야 합니다. 

먼저 고객 VPC에서 API Gateway의 VPC로 이동하여, VPC내의 클라이언트가 API Gateway 서비스 엔드포인트에 도달할 수 있도록 연결해야 합니다. 두번째로는 API Gateway의 VPC에서 고객 VPC로 이동하여 API Gateway가 백엔드 엔드포인트에 접근할 수 있도록 연결해야 합니다. 여기 예시 아키텍처가 있습니다.

![Private API with private integrations](https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2021/08/10/Screen-Shot-2021-08-10-at-1.39.25-PM.png)

<br />

## VPC links for HTTP APIs
HTTP APIs는 API Gateway의 최신 타입으로, REST APIs보다 빠르고, 저렴합니다. 

[VPC Links for HTTP APIs](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-vpc-links.html)는 VPC 엔드포인트 서비스의 생성을 필요로 하지 않습니다. 그래서 네트워크 로드밸런서(NLB)가 필수적이지 않습니다. 

HTTP APIs용 VPC Links를 통해 이제 ALB 또는 AWS Cloud Map 서비스로 프라이빗 리소스를 타겟(대상)으로 지정할 수 있습니다. 이는 양쪽에 필요한 구성에서 더 많은 유연성과 확장성을 줄 수 있습니다. 

HTTP API용 VPC Links를 사용하면 여러 통합 대상을 구성하는 것도 더 쉽게 할 수 있습니다. 

예를 들어 REST API용 VPC Links는 오직 단일 NLB와 결합할 수 있습니다. 여러 백엔드 엔드포인트를 설정하기 위해서는 NLB에서 여러 대상 그룹과 연결된 여러 리스너를 사용하는 것과 같은 해결방법이 필요합니다. 

반면에, HTTP APIs를 위한 단일 VPC link는 별도의 부가적인 설정 없이도 여러 백엔드 엔드포인트와 연결 할 수 있습니다. 또한, 새로운 VPC link를 통해 컨테이너화 된 애플리케이션을 사용하는 고객은 NLB 대신 ALB를 사용하고 7 레이어의 로드밸런싱 기능과 인증 및 권한 부여와 같은 다른 기능을 활용할 수 있습니다.  

AWS Hyperplane은 AWS PrivateLink를 포함한 여러 타입의 네트워크 가상화 구성을 지원합니다. REST API용의 VPC links는 AWS PrivateLink에 의존합니다. 하지만 HTTP API용 VPC links는 더 상위 레벨의 추상화를 제공하는 VPC-to-VPC NAT을 사용합니다. 

새 구성은 개념적으로는 양 VPC간의 터널을 만드는 것과 비슷합니다. 이는 AWS Hyperplane에서 모두 관리하는 공급자와 소비자 측 말단에 탄력적 네트워크 인터페이스(ENI) 연결을 통해 생성됩니다. 이 터널은 공급자의 VPC(여기서는 API Gateway)에서 호스팅 되는 서비스가 소비자의 VPC에 있는 리소스에 대한 통신을 시작할 수 있게 합니다. 

API Gateway는 탄력적 네트워크 인터페이스(ENI)에 직접 연결하고, 자체 VPC에서 직접 VPC에 있는 리소스에 도달할 수 있습니다. 고객 측의 탄력적 네트워크 인터페이스(ENI)에 부착된 보안 그룹의 구성에 따라서 연결이 허용됩니다. 

비록 이것이 AWS PrivateLink와 같은 기능을 제공하는 것처럼 보일 수 있지만, 이러한 구성은 구현 세부사항이 다릅니다.  

PrivateLink의 서비스 엔드포인트는 NLB와 같은 단일 엔드포인트에 대한 다중 연결을 허용하는 반면에, 새로운 접근 방식은 소스가 되는 VPC가 여러 대상 엔드포인트에 연결될 수 있도록 합니다. 결과적으로 단일 VPC link는 고객 측에 위치한 여러 애플리케이션 로드밸런서, 네트워크 로드밸런서, 혹은 AWS Cloud Map 서비스에 등록된 리소스와 통합될 수 있습니다. 

![VPC Links for HTTP APIs](https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2021/08/10/Screen-Shot-2021-08-10-at-1.42.02-PM.png)

이러한 접근은 [람다가 고객 VPC안의 리소스에 접근하는 것](https://aws.amazon.com/ko/blogs/compute/announcing-improved-vpc-networking-for-aws-lambda-functions/)과 같은 방식과 비슷합니다. 

<br />

## Conclusion
이 포스트는 VPC Links가 어떻게 프라이빗 통합으로 API Gateway APIs를 설정할 수 있는지 탐구합니다. REST API용 VPC Links는 API Gateway의 VPC와 고객 VPC 사이의 연결을 구성하여 프라이빗 백엔드 엔드포인트로 접근하기 위해서, 인터페이스 VPC 엔드포인트 및 VPC 엔드포인트 서비스와 같은 AWS PrivateLink 리소스를 캡슐화 합니다.  

HTTP APIs를 위한 VPC Links는 AWS Hyperplane 서비스 안에서 다른 구성을 사용하여, VPC 프라이빗 리소스에 API Gateway와 직접 네트워크 접근을 제공합니다. 

여러분의 API 아키텍처 디자인의 한 부분으로 프라이빗 통합을 추가 할 때, 이러한 두 기능의 차이점을 이해하는 것은 매우 중요합니다. 


<br />

위의 내용 중 [원문](https://aws.amazon.com/ko/blogs/compute/understanding-vpc-links-in-amazon-api-gateway-private-integrations/)과 다른 점이 있거나, 오류가 있다면 언제든 아래 메일로 말씀해주시기 바랍니다. 감사합니다. 
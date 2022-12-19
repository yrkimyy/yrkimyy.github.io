---
layout: post
title: Terraform Associate 준비하기 02
subtitle: 오답노트 02
tags: [Terraform]
comments: true
---


## 오답노트 02

### 01

Peter는 VPC, 서브넷, 보안 그룹, 라우팅 테이블 및 nat-gateway를 생성하는 모듈을 작성하고 있습니다. 이러한 각 리소스에는 공통 팀별 태그가 있습니다. 따라서 구성에서 동일한 태그를 여러 번 정의하는 대신. 향후 구성을 더 읽기 쉽고 쉽게 업데이트하려면 Peter가 어떤 접근 방식을 사용해야 합니까?

정답 : Use Local Values

Local Value은 구성에서 동일한 값이나 표현식을 여러 번 반복하는 것을 방지하는 데 도움이 될 수 있지만 과도하게 사용되면 사용된 실제 값을 숨겨 미래의 유지 관리자가 구성을 읽기 어렵게 만들 수도 있습니다. 단일 값이나 결과가 여러 곳에서 사용되고 해당 값이 향후 변경될 가능성이 있는 상황에서 Local Value만 적당히 사용하십시오. 중앙 위치에서 값을 쉽게 변경할 수 있는 기능은 Local Value의 주요 이점입니다.

<br />

### 02

**How will you share state files between team members so that each team member can access the same Terraform state files to be able to use Terraform to update infrastructure?**

팀 멤버들간에 어떻게 상태 파일을 공유할 수 있습니까? 그래서 각 멤버가 같은 테라폼 상태 파일에 접근해 인프라를 업데이트 하기 위해서 테라폼을 사용하려면 어떻게 해야 합니까?

해설 : 

Remote State By default, Terraform stores state locally in a file named terraform.tfstate. When working with Terraform in a team, the use of a local file makes Terraform usage complicated because each user must make sure they always have the latest state data before running Terraform and make sure that nobody else runs Terraform at the same time. With a remote state, Terraform writes the state data to a remote data store, which can then be shared between all members of a team. Terraform supports storing state in Terraform Cloud, HashiCorp Consul, Amazon S3, Alibaba Cloud OSS, and more.

원격 상태 기본적으로 Terraform은 terraform.tfstate라는 파일에 로컬로 상태를 저장합니다. 팀에서 Terraform으로 작업할 때 로컬 파일을 사용하면 각 사용자가 Terraform을 실행하기 전에 항상 최신 상태 데이터를 가지고 있는지 확인하고 다른 사람이 동시에 Terraform을 실행하지 않도록 해야 하므로 Terraform 사용이 복잡해집니다. **원격 상태에서 Terraform은 원격 데이터 저장소에 상태 데이터를 기록하고 팀의 모든 구성원 간에 공유할 수 있습니다. Terraform은 Terraform Cloud, HashiCorp Consul, Amazon S3, Alibaba Cloud OSS 등에서 상태 저장을 지원합니다.**

<br />


### 03

**What is the terraform state command used for? Choose THREE correct answers.**

참조 링크: [https://nyyang.tistory.com/119](https://nyyang.tistory.com/119) 

• [ ] **Importing manually created resources to Terraform state.**
• [ ㅇ ] **Modifying/Updating Terraform state.**
• [ ] **Checking backend utilization state.**
• [ ] **Initializing new changes in remote state configuration.**
• [ ㅇ ] **Moving items in/out a Terraform state.**
• [ ㅇ ] **Advanced state management.**

<br />


### 04

**Which command can be used to inspect a plan to ensure that the planned operations are expected?**

• [ ] **terraform inspect**
• [ ] **terraform state**
•  [ ㅇ ] **terraform show**
• [ ] **terraform plan**

해설: The terraform show command is used to provide human-readable output from a state or plan file. This can be used to inspect a plan to ensure that the planned operations are expected, or to inspect the current state as Terraform sees it.

<br />


### 05

**You have started getting lots of conflicts and inconsistencies in the infrastructure managed by Terraform ever since You began using Amazon S3 as Terraform backend. After some troubleshooting, you observed that this only happens when more than one user tries to make infrastructure changes with Terraform. How will you resolve it?**
Amazon S3를 Terraform 백엔드로 사용하기 시작한 이후로 Terraform이 관리하는 인프라에서 많은 충돌과 불일치가 발생하기 시작했습니다. 몇 가지 문제 해결 후 두 명 이상의 사용자가 Terraform으로 인프라 변경을 시도할 때만 이 문제가 발생한다는 것을 확인했습니다. 어떻게 해결하시겠습니까?

정답: Enable Terraform state locking for the Amazon S3 backend by using a DynamoDB table to avoid race around the condition and concurrent execution problem of the same Terraform configuration.

해설: 상태 잠금이 필요한 이유는 무엇입니까? 상태를 원격으로 저장하면 원활하게 협업할 수 있습니다. 이를 통해 각 팀 구성원이 최신 상태 파일에 액세스할 수 있지만 우발적인 상태 파일 손상의 위험도 있습니다. 특히 두 명의 팀 구성원이 동시에 Terraform을 실행할 때 여러 terraform 적용 명령으로 경합 상태가 발생할 수 있습니다. 동일한 상태 파일에 대한 동시 업데이트로 인해 충돌 및 데이터 손실, 상태 파일 손상이 발생합니다. 이미 사용 중인 상태 파일을 열지 못하게 하는 기능인 상태 잠금으로 이 문제를 해결할 수 있습니다. 모든 백엔드가 잠금을 지원하는 것은 아닙니다. 백엔드가 지원하고 활성화된 경우 Terraform은 상태를 쓸 수 있는 모든 작업에 대해 상태를 잠급니다. 이렇게 하면 다른 사람이 잠금을 획득하고 잠재적으로 상태를 손상시키는 것을 방지할 수 있습니다.

<br />


### 06

**To publish a module to Terraform Public Registry. A module must meet the following requirements. Choose TWO correct answers.**

• [ ] GitHub repository must have HarshiCorp Licence (Version ~> 2).
• [x] **The module must be on a public GitHub repo with at least one release tag in x.y.z format.**
• [ ] Module GitHub repositories must use this three-part name format terraform-<PROVIDER>-<NAME> where <NAME> segment can not contain any special characters.
• [ ] The module must be on GitHub and must be a private repo with at least one release tag in x.y.z format.
• [x] **Module GitHub repositories must use this three-part name format terraform-<PROVIDER>-<NAME> where <NAME> segment can contain additional hyphens.**
• [ ] Module GitHub repositories must use this three-part name format module-<PROVIDER>-<NAME> where <NAME> segment can contain additional hyphens.

Requirements The list below contains all the requirements for publishing a module. Meeting the requirements for publishing a module is extremely easy. The list may appear long only to ensure we're detailed, but adhering to the requirements should happen naturally. * GitHub. The module must be on GitHub and must be a public repo. This is only a requirement for the public registry. If you're using a private registry, you may ignore this requirement.

요구 사항 아래 목록에는 모듈을 게시하기 위한 모든 요구 사항이 포함되어 있습니다. 모듈 게시 요구 사항을 충족하는 것은 매우 쉽습니다. 자세한 내용을 확인하기 위해 목록이 길게 표시될 수 있지만 요구 사항을 준수하는 것은 자연스럽게 이루어져야 합니다. * 깃허브. 모듈은 GitHub에 있어야 하며 공개 리포지토리여야 합니다. 이는 공개 레지스트리에 대한 요구사항일 뿐입니다. 개인 레지스트리를 사용하는 경우 이 요구 사항을 무시해도 됩니다.

<br />


### 07

**The terraform console command provides an interactive console for evaluating expressions.**

terraform console 명령은 식을 평가하기 위한 대화형 콘솔을 제공합니다.

<br />


### 08

**In Terraform 0.12 terraform init cannot automatically download third-party providers.**
Terraform 0.12에서 terraform init는 타사 공급자를 자동으로 다운로드할 수 없습니다

해설

Anyone can develop and distribute their own Terraform providers. These third-party providers must be manually installed, since terraform init cannot automatically download them. Install third-party providers by placing their plugin executables in the user plugins directory. The user plugins directory is in one of the following locations, depending on the host operating system:

누구나 자체 Terraform 공급자를 개발하고 배포할 수 있습니다. 이러한 타사 공급자는 terraform init에서 자동으로 다운로드할 수 없으므로 수동으로 설치해야 합니다. 플러그인 실행 파일을 사용자 플러그인 디렉토리에 배치하여 타사 공급자를 설치합니다. 사용자 플러그인 디렉토리는 호스트 운영 체제에 따라 다음 위치 중 하나에 있습니다.


<br />


### 09

**What are the features exclusive to Terraform Enterprise. Select all that apply. Choose TWO correct answers.**

Terraform Enterprise 전용 기능은 무엇입니까?
• [x] **SAML single sign-on**
• [ ] **Sentinel**
• [x] **Audit logging**
• [ ] **Cost Estimation**

해설: Terraform Enterprise는 Terraform Cloud의 자체 호스팅 배포판입니다. 기업에 리소스 제한이 없고 감사 로깅 및 SAML 싱글 사인온과 같은 추가 엔터프라이즈급 아키텍처 기능이 있는 Terraform Cloud 애플리케이션의 개인 인스턴스를 제공합니다.

<br />

### 10

**Every Terraform configuration has at least one module.**

해설: Every Terraform configuration has at least one module, known as its root module, which consists of the resources defined in the .tf files in the main working directory.

모든 Terraform 구성에는 기본 작업 디렉터리의 .tf 파일에 정의된 리소스로 구성된 루트 모듈이라고 하는 모듈이 하나 이상 있습니다.

<br />

### 11

**Why should you constrain the acceptable provider versions in Terraform configuration for production use?**

프로덕션 사용을 위해 Terraform 구성에서 허용 가능한 제공자 버전을 제한해야 하는 이유는 무엇입니까?

정답

Provider는 Terraform과 별도의 리듬으로 출시되는 플러그인이므로 공급자는 고유한 버전 번호를 가지며 버전 제약 조건은 향후 terraform init에 의해 주요 변경 사항이 있는 새 버전이 자동으로 설치되지 않도록 합니다. 

<br />

### 12

**The default setting of Terraform Open Source is to store your state file on your local laptop or your workstation.**

해설

By default, Terraform states are stored in a local file named terraform.tfstate under the current working directory, but it can also be stored remotely, which works better in a team environment.

<br />

### 13

**A calling module can directly access child module resource attributes.**

정답: false

해설

The resources defined in a module are encapsulated, so the calling module cannot access their attributes directly. However, the child module can declare output values to selectively export certain values to be accessed by the calling module. For example, if the ./app-cluster module referenced in the example above exported an output value named instance_ids then the calling module can reference that result using the expression
모듈에 정의된 자원은 캡슐화되므로 호출 모듈은 해당 속성에 직접 액세스할 수 없습니다. 그러나 하위 모듈은 호출 모듈에서 액세스할 특정 값을 선택적으로 내보내도록 출력 값을 선언할 수 있습니다. 예를 들어 위의 예에서 참조된 ./app-cluster 모듈이 instance_ids라는 출력 값을 내보낸 경우 호출 모듈은 다음 식을 사용하여 해당 결과를 참조할 수 있습니다.

<br />

### 14

**The expression of a local value can refer to other locals**

The expression of a local value can refer to other locals, but as usual reference cycles are not allowed. That is, a local cannot refer to itself or to a variable that refers (directly or indirectly) back to it.

지역 값의 표현은 다른 지역을 참조할 수 있지만 일반적인 참조 순환은 허용되지 않습니다. 즉, 로컬은 자신을 참조하거나 (직접 또는 간접적으로) 다시 참조하는 변수를 참조할 수 없습니다.

<br />

### 15

**Your staging and production environment points to the same module, which is making it harder to test a change in staging without any chance of affecting production. Your colleague suggested you to use versioned modules so that you can use one version in staging and a different version in production. How can you use this approach if you are sourcing modules from a private git repo?**
스테이징 및 프로덕션 환경이 동일한 모듈을 가리키므로 프로덕션에 영향을 주지 않고 스테이징 변경 사항을 테스트하기가 더 어렵습니다. 동료는 스테이징에서 한 버전을 사용하고 프로덕션에서 다른 버전을 사용할 수 있도록 버전이 지정된 모듈을 사용할 것을 제안했습니다. 비공개 git 저장소에서 모듈을 소싱하는 경우 이 접근 방식을 어떻게 사용할 수 있습니까?

정답

**Create separate tags and use a stable one for the production and a new one for the staging.**

기본적으로 Terraform은 선택한 리포지토리에서 기본 분기(HEAD에서 참조)를 복제하고 사용합니다. ref 인수를 사용하여 이를 재정의할 수 있습니다. ref 매개변수를 사용하면 특정 Git 커밋 sha1 해시, 분기 이름 또는 특정 Git 태그를 지정할 수 있습니다. 태그는 커밋에 대한 포인터일 뿐이지만 사람이 읽을 수 있는 친근한 이름을 사용할 수 있습니다. 예를 들어 Git 태그 v1.2.0을 사용해야 하는 경우 아래와 같이 모듈 블록 아래의 소스 URL을 수정할 수 있습니다.

<br />


### 16

**Which of the following is right about Terraform? Choose THREE correct answers.**

[x] Customer teams can use shared Terraform configurations and use them as a black box tool to manage their services.
[ ] Terraform can codified infrastructure configuration; however, configuration sharing within an organization still not possible.
[x] Terraform is not limited to physical providers like AWS. Resource schedulers can be treated as a provider, enabling Terraform to request resources from them.
[x] Terraform can help team creating disposable parallel environments.
[ ] Terraform is not cloud-agnostic

일회용 환경 프로덕션 및 스테이징 또는 QA 환경을 모두 갖는 것이 일반적입니다. 이러한 환경은 프로덕션 환경의 더 작은 복제본이지만 프로덕션에서 릴리스하기 전에 새 애플리케이션을 테스트하는 데 사용됩니다. Terraform을 사용하면 프로덕션 환경을 코드화한 다음 스테이징, QA 또는 개발과 공유할 수 있습니다. 이러한 구성을 사용하여 테스트할 새 환경을 신속하게 가동한 다음 쉽게 폐기할 수 있습니다. Terraform은 병렬 환경 유지의 어려움을 길들이는 데 도움이 될 수 있으며 이를 탄력적으로 만들고 파괴하는 것이 실용적입니다.

<br />

### 17

**Workspaces in Terraform Cloud and Terraform CLI aren’t the same.**

Terraform Cloud와 Terraform CLI에는 둘 다 "작업 공간"이라는 기능이 있지만 약간 다릅니다. CLI 작업공간은 동일한 작업 디렉토리에 있는 대체 상태 파일입니다. 하나의 구성을 사용하여 여러 유사한 리소스 그룹을 관리하기 위한 편리한 기능입니다.

<br />

### 18

**Which Terraform Enterprise feature allows users to create and confidentially share infrastructure modules within an organization?**

정답

Private Module Registry

해설

registry.terraform.io의 레지스트리는 공개 모듈만 호스팅하지만 대부분의 조직에는 공개할 수 없거나 공개할 필요가 없는 일부 모듈이 있습니다. 버전 제어 및 기타 소스에서 직접 비공개 모듈을 로드할 수 있지만 이러한 소스는 대규모 조직에서 생산자 및 소비자 콘텐츠 모델을 활성화하는 데 중요한 두 가지 모두 버전 제약 조건 또는 탐색 가능한 모듈 시장을 지원하지 않습니다. 팀이 다른 팀에서 만든 모듈을 자주 사용할 정도로 조직이 전문화된 경우 개인 모듈 레지스트리의 이점을 얻을 수 있습니다.

<br />

### 19

**How to add a provider to Terraform v0.12 configuration?**

정답

- Add any resource from the required provider and Terraform will automatically download the provider while running terraform init command.
- Explicitly add provider configuration using a provider block.

해설

새 공급자는 공급자 블록을 통해 명시적으로 또는 해당 공급자의 리소스를 추가하여 구성에 추가할 수 있습니다. Terraform은 이를 사용하기 전에 초기화해야 합니다. 초기화는 나중에 실행할 수 있도록 공급자의 플러그인을 다운로드하고 설치합니다. terraform init 명령은 아직 초기화되지 않은 공급자를 다운로드하고 초기화합니다.

<br />

### 20

**You have created a new workspace called DEV under directory /home/quiz_experts/iac/ and initialized terraform by running terraform init command with local backend. If you run terraform apply, where does Terraform will write its state data?**

정답
 `/home/quiz_experts/iac/terraform.tfstate.d/DEV/terraform.tfstate`

여러 작업 공간의 일반적인 용도는 기본 프로덕션 인프라를 수정하기 전에 일련의 변경 사항을 테스트하기 위해 인프라 세트의 병렬 고유 사본을 생성하는 것입니다. 

예를 들어 복잡한 일련의 인프라 변경 작업을 수행하는 개발자는 기본 작업 영역에 영향을 주지 않고 변경 사항을 자유롭게 실험하기 위해 새로운 임시 작업 영역을 만들 수 있습니다. 작업 공간은 상태 파일의 이름을 바꾸는 것과 기술적으로 동일합니다. 그들은 그것보다 더 복잡하지 않습니다. Terraform은 원격 상태에 대한 보호 및 지원 세트로 이 간단한 개념을 래핑합니다. 로컬 상태의 경우 Terraform은 terraform.tfstate.d라는 디렉토리에 작업공간 상태를 저장합니다. 이 디렉토리는 로컬 전용 terraform.tfstate와 유사하게 취급되어야 합니다. 따라서 기본 작업 공간이 아닌 경우 상태는 파일에 저장됩니다.

<br />

### 21

**Which command automatically updates configurations in the current directory for easy readability and consistency?**

정답
`terraform fmt`

해설

We recommend using consistent formatting in files and modules written by different teams. The terraform fmt command automatically updates configurations in the current directory for easy readability and consistency. Format your configuration. Terraform will return the names of the files it formatted. If your configuration file is already formatted correctly, Terraform won't return any file names.

서로 다른 팀에서 작성한 파일 및 모듈에서 일관된 형식을 사용하는 것이 좋습니다. terraform fmt 명령은 쉬운 가독성과 일관성을 위해 현재 디렉토리의 구성을 자동으로 업데이트합니다. 구성을 포맷합니다. Terraform은 포맷한 파일의 이름을 반환합니다. 구성 파일이 이미 올바르게 형식화된 경우 Terraform은 파일 이름을 반환하지 않습니다.
<br />


### 22

**Terraform recommends using provisioners instead of configuration management tools to invoke scripts or installing software on a newly built VMs.**
→ False

해설

프로비저너는 최후의 수단" 프로비저너는 최후의 수단으로만 사용해야 합니다. 대부분의 일반적인 상황에는 AWS EC2의 user_data와 같은 더 나은 대안이나 구성 관리를 위해 특별히 구축된 Chef/Ansible/Saltstack과 같은 도구가 있습니다.

<br />

### 23

**The parent module has to provide input variables and providers to a child module explicitly in the module block as this information can not be passed down to descendent modules implicitly through inheritance.**
→ False

공급자는 상속을 통해 암시적으로 또는 모듈 블록 내에서 공급자 인수를 통해 명시적으로 두 가지 방법으로 하위 모듈로 전달될 수 있습니다. 이 두 가지 옵션은 다음 섹션에서 자세히 설명합니다. - 공급자 구성은 루트 Terraform 모듈에서만 정의할 수 있습니다.

<br />

### 24

**Which of the following variable type allows multiple values of several distinct types to be grouped together as a single value? Choose TWO correct answers.**

[x] Object
[ ] List
[ ] Map
[x] Tuple
[ ] Set

구조 유형을 사용하면 여러 고유 유형의 여러 값을 단일 값으로 그룹화할 수 있습니다. 구조 유형에는 어떤 유형이 어떤 요소에 허용되는지 지정하기 위해 인수로 스키마가 필요합니다. Terraform 언어의 두 종류의 구조 유형은 다음과 같습니다. * object(...): 각각 고유한 유형을 갖는 명명된 속성 모음입니다.

<br />

### 25

**You created a Terraform module to launch new VMs in Cloud. During your tests in a dev environment with a few VMs terraform plan and apply commands were running fine. You started using this module on production and have launched hundreds of VMs with it, and with time, you started observing slowness during terraform plan and apply commands execution. What could be the possible reason of slowness? Choose THREE correct answers.**

Cloud에서 새 VM을 시작하기 위해 Terraform 모듈을 만들었습니다. 몇 개의 VM이 있는 개발 환경에서 테스트하는 동안 terraform 계획 및 적용 명령이 제대로 실행되었습니다. 프로덕션 환경에서 이 모듈을 사용하기 시작했고 수백 대의 VM을 시작했으며 시간이 지남에 따라 terraform 계획 및 적용 명령 실행 중에 속도 저하를 관찰하기 시작했습니다. 속도 저하의 가능한 원인은 무엇입니까?

• [ ] VM running Terraform CLI has no enough resources.
• [x] Many cloud providers do not provide APIs to query multiple resources at once, and the round trip time for each resource is hundreds of milliseconds; hence for a large infrastructure querying every resource causes slowness during the plan and apply command.

많은 클라우드 공급자는 한 번에 여러 리소스를 쿼리하는 API를 제공하지 않으며 각 리소스의 왕복 시간은 수백 밀리초입니다. 따라서 모든 리소스를 쿼리하는 대규모 인프라의 경우 계획 및 적용 명령 중에 속도가 느려집니다.

• [x] Cloud providers have API rate-limiting, and for a large infrastructure, Terraform has to make a lot of calls to querying every resource; hence API rate-limitation stops Terraform from making many concurrent API calls at once that make a plan and apply command slow.

클라우드 공급자는 API 속도 제한이 있으며 대규모 인프라의 경우 Terraform은 모든 리소스를 쿼리하기 위해 많은 호출을 수행해야 합니다. 따라서 API 속도 제한은 Terraform이 한 번에 많은 동시 API 호출을 수행하여 계획 및 적용 명령을 느리게 만드는 것을 방지합니다.

• [ ] Terraform creates one VM at a time; hence, plan and apply commands were fast during test since you only created a few VMs, but on production, you may have to create many VMs at a time, which makes it slow.
• [x] By default, for every plan and apply, Terraform will sync all resources in your state with the real world to know the current state of resources to effectively determine the changes that it needs to make to reach your desired configuration. For large infrastructures, this behavior of querying every resource makes plan and apply commands execution very slow.

기본적으로 모든 계획 및 적용에 대해 Terraform은 원하는 구성에 도달하는 데 필요한 변경 사항을 효과적으로 결정하기 위해 리소스의 현재 상태를 알기 위해 상태의 모든 리소스를 실제 세계와 동기화합니다. 대규모 인프라의 경우 모든 리소스를 쿼리하는 이러한 동작으로 인해 계획 및 적용 명령 실행이 매우 느려집니다.

해설

기본 리소스 매핑 외에도 Terraform은 상태의 모든 리소스에 대한 속성 값의 캐시를 저장합니다. 이는 Terraform 상태의 가장 선택적 기능이며 성능 향상으로만 수행됩니다. Terraform 계획을 실행할 때 Terraform은 원하는 구성에 도달하는 데 필요한 변경 사항을 효과적으로 결정하기 위해 리소스의 현재 상태를 알아야 합니다. 소규모 인프라의 경우 Terraform은 공급자를 쿼리하고 모든 리소스의 최신 특성을 동기화할 수 있습니다. 이것은 Terraform의 기본 동작입니다. 모든 계획 및 적용에 대해 Terraform은 상태의 모든 리소스를 동기화합니다.

<br />

### 26

**Which of the following is the primitive type constraint in Terraform? Choose TWO correct answers.**
→ String, Bool

<br />

### 27

**How does Terraform create provider dependencies and understand why a particular provider is needed?**

Terraform은 공급자 종속성을 어떻게 생성하고 특정 공급자가 필요한 이유를 이해합니까?

- **Resource belonging to a particular provider removed from configuration but exists in the current state.**
- **Use of any resource belonging to a particular provider in a resource or data block in the configuration.**
- **Explicit use of a provider block in configuration, optionally including a version constraint.**

공급자 종속성은 여러 가지 방법으로 생성됩니다. * 선택적으로 버전 제약 조건을 포함하여 구성에서 공급자 블록을 명시적으로 사용합니다. * 구성에서 리소스 또는 데이터 블록의 특정 제공자에 속하는 모든 리소스 사용. * 현재 상태에서 특정 공급자에 속하는 리소스 인스턴스의 존재. 예를 들어 특정 리소스가 구성에서 제거되면 해당 인스턴스가 소멸될 때까지 공급자에 대한 종속성이 계속 생성됩니다. terraform provider 명령은 특정 공급자가 필요한 이유를 이해하는 데 도움이 되도록 현재 모든 종속성에 대한 개요를 제공합니다.

<br />

### 28

**A module can access all the variables of the parent module.**
→ False

모듈은 모든 상위 모듈 변수에 액세스할 수 없습니다. 따라서 자식 모듈에 변수를 전달하려면 호출 모듈이 모듈 블록의 특정 값을 전달해야 합니다.

<br />

### 29

**Which of the below command do you need to run when you are using New module for the first time?**

`terraform init`

`terraform get`

<br />

### 30

**Which among the following are True w.r.to Terraform module installation?**

• If it is a local modules, Terraform will create a symlink to the module's directory.
• All the module configuration files will be downloaded to .terraform/modules directory within your configuration working directory.
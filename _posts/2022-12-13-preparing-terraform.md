---
layout: post
title: Terraform Associate 준비하기 01
subtitle: 오답노트 01
tags: []
comments: true
---

## 오답노트 01



### How do you create a workspace? 


🔥 정답 : terraform workspace new


💡해설

terraform workspace
Subcommands:

    show      Show the current workspace name.
    list      List workspaces.
    select    Select a workspace.
    new       Create a new workspace.
    delete    Delete an existing workspace.


 참고 자료
- [배포 환경 분리하기](https://medium.com/@blaswan/terraform-workspaces-for-deployment-environments-2deff99356f6)
<hr style="height:1px;border:1px solid black;"/>


### Which Terraform Workflow ( Write -> Plan -> Create ) does this describe? 


```
- The project resides in a repo, and the backend is 
- configured to use Terraform Cloud
- Pull requests are submitted to the repo with new changes When the Pull Request is approved Terraform Cloud runs terraform apply
```

🔥 정답 : Core Workflow Enhanced



💡해설
The Core Terraform Workflow: 
  
    Write: 
    에디터에서 코드를 작성하는 것처럼 테라폼 구성을 작성
    팀/개인 관계 없이 버전 관리 저장소에 작업을 저장하는 것이 일반적인 방식
    애플리케이션 코드 작업방식과 유사

    Plan: 
    Write 단계에서의 피드백이나, 변경사항이 적합하면 커밋하고 플랜을 진행
    terraform apply 명령이 인프라의 변경을 진행하기 전에 플랜을 보여줌

    Apply: 
    At this point, 
    it's common to push your version control repository 
    to a remote location for safekeeping.



<hr style="height:1px;border:1px solid black;"/>

### When we want the most verbose information from terraform logging what severity should we set? 

🔥 정답 : Trace


💡해설

테라폼에는 TF_LOG 환경변수를 통해 활성화 할 수 있는 로그가 있음
Trace로 올라갈 수록 자세한 로그, Error는 애플리케이션을 사용할 수 없을 정도의 fatal한 로그를 의미함. 

    Trace - Only when I would be "tracing" the code 
    and trying to find one part of a function specifically. 

    Debug - Information that is diagnostically helpful 
    to people more than just developers (IT, sysadmins, etc.)

    Info - Generally useful information to log (service start/stop, configuration assumptions, etc). 
    Info I want to always have available but usually don't care about under normal circumstances. 
    This is my out-of-the-box config level

    Warn - Anything that can potentially cause application oddities, but for which I am automatically recovering. 
    (Such as switching from a primary to backup server, retrying an operation, missing secondary data, etc.)

    Error - Any error which is fatal to the operation, but not the service or application (can't open a required file, missing data, etc.). 
    These errors will force user (administrator, or direct user) intervention. 
    These are usually reserved (in my apps) for incorrect connection strings, missing services, etc.



<hr style="height:1px;border:1px solid black;"/>

### Which is NOT a valid argument for remote-exec? 

🔥 정답 : interpreter



💡해설

interpreter is an argument available to local-exec
local-exec이랑 헷갈리지 말자. 

    inline - This is a list of command strings. 
    They are executed in the order they are provided. 
    This cannot be provided with script or scripts.

    script - This is a path (relative or absolute) to a local script that will be copied to the remote resource and then executed. 
    This cannot be provided with inline or scripts.

    scripts - This is a list of paths (relative or absolute) to local scripts that will be copied to the remote resource and then executed. 
    They are executed in the order they are provided. This cannot be provided with inline or script.





<hr style="height:1px;border:1px solid black;"/>

### Is this a valid configuration for remote-exec? 

```
resource "aws_instance" "web" {
  # ...

  provisioner "remote-exec" {
    inline = [
      "puppet apply",
      "consul join ${aws_instance.web.private_ip}",
    ]
   interpreter = ["bash", "-e"]
  }
}

```

🔥 정답 : FALSE



💡해설

interpreter is not a valid argument for remote-exec. 
local-exec does have a valid argument called interpreter

remote-exec에는 interpreter 인자가 없음. 



<hr style="height:1px;border:1px solid black;"/>

### The Terraform Registry contains both public and private providers and modules?
 


🔥 정답 : FALSE



💡해설

The Terraform Registry only contains public providers and modules.

테라폼 레지스트리는 오직 퍼블릭 프로바이더와 모듈만 있음. 





<hr style="height:1px;border:1px solid black;"/>

### A DevOps Engineer needs to reference an existing AMI (machine image) for an AWS Virtual machine called example. What would be the correct resource address to assign this AMI to another virtual machine?

 

```
resource "aws_instance" "example" {
  ami           = "ami-abc123"
  instance_type = "t2.micro"

  ebs_block_device {
    device_name = "sda2"
    volume_size = 16
  }
  ebs_block_device {
    device_name = "sda3"
    volume_size = 20
  }
}
```

🔥 정답 :
```
resource "aws_instance" "example2" {
  ami = aws_instance.example.ami

```

💡해설

리소스 블럭을 참조한다면, 정답과 같이 쓰는 것이 맞고, 데이터 블럭을 참조한다면 아래와 같기 작성하는 것이 맞음

```
ami = data.aws_ami.example.id
```



<hr style="height:1px;border:1px solid black;"/>

### How can you quickly start using Sentinel with Terraform Cloud? 
<br />

🔥 정답 : Inspect your previous list of runs, download a sentinel mock file and then import a mock


💡해설

센티넬(Sentinel)은 Infra의 보안 요구사항을 사전에 정의하고, 이를 코드로 관리하는 Policy-as-Code 프레임워크다. 


<hr style="height:1px;border:1px solid black;"/>

### Which of the following is NOT a built-in string function? 
<br />

🔥 정답 : slice


💡해설

slice는 collection function임



<hr style="height:1px;border:1px solid black;"/>

### What does the coalesce built-in function in Terraform do? 
<br />

🔥 정답 : coalesce takes any number of arguments and returns the first one that isn't null or an empty string.


💡해설

coalesce: 인자를 받아서, null이나 빈 문자열이 아닌 첫번째 인자를 리턴함. 하지만 인자들의 타입이 같아야 함. 컬렉션 타입과 원시타입을 함께 쓸 수 없음.  
인자의 타입이 리스트이면, coalescelist를 쓸 수 있음. 

compact: 문자열 리스트를 받아서, 빈 문자열 요소를 삭제한 새로운 리스트를 리턴함

transpose : 문자열 요소를 가지는 리스트의 맵을 받아서, 키와 값을 스왑하고, 문자열 리스트로 구성된 새로운 맵을 만듬


<hr style="height:1px;border:1px solid black;"/>

### When running terraform fmt what changes will occur to the configuration file?
 

```
resource "aws_instance" "my_example_server"
{
  ami           = "INVALID_AMI_VALUE"
  instance_type = "t2.nano"
}
```

<br />


🔥 정답 : Terraform fmt will produce a syntax error, asking for the user to correct the curly brackets


💡해설

  The format of the brackets is incorrect since they should be the same line as the block. However in this case it will result in a syntax error since terraform fmt does not appear capable of linting and correcting curly brackets.

  위 코드에서 {}의 형태가 맞지 않음. 그러나 테라폼 fmt가 중괄호를 수정해주지는 않아서, syntax error가 나게 됨. (그러고 보니, syntax에러를 만나본 적이 없구만 허허)



<hr style="height:1px;border:1px solid black;"/>

###  When passing the filename of a saved plan file to terraform apply FILENAME what will happen? 
<br />

🔥 정답 : Terraform apply will not prompt for approval


💡해설

[Saved Plan Mode](https://developer.hashicorp.com/terraform/cli/commands/apply#saved-plan-mode)
이 모드를 사용하면, 프롬프트에 approval을 묻지 않고(plan)없이 바로 진행함. 




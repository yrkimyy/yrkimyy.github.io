---
layout: post
title: Terraform Associate 준비하기 01
subtitle: 오답노트 01
tags: [Terraform]
comments: true
---

## 오답노트


<details>
<summary>1. How do you create a workspace? </summary>
<div markdown="1">
🔥 정답 : terraform workspace new

```
💡해설
terraform workspace
Usage: terraform workspace

  Create, change and delete Terraform workspaces.

Subcommands:

    show      Show the current workspace name.
    list      List workspaces.
    select    Select a workspace.
    new       Create a new workspace.
    delete    Delete an existing workspace.
```

 참고 자료
- [배포 환경 분리하기](https://medium.com/@blaswan/terraform-workspaces-for-deployment-environments-2deff99356f6)

</div>
</details>


<details>
<summary>2. Which Terraform Workflow ( Write -> Plan -> Create ) does this describe? </summary>
<div markdown="1">

```
- The project resides in a repo, and the backend is 
- configured to use Terraform Cloud
- Pull requests are submitted to the repo with new changes When the Pull Request is approved Terraform Cloud runs terraform apply
```

🔥 정답 : Core Workflow Enhanced


```
💡해설
The Core Terraform Workflow

Write 
에디터에서 코드를 작성하는 것처럼 테라폼 구성을 작성
팀/개인 관계 없이 버전 관리 저장소에 작업을 저장하는 것이 일반적인 방식
애플리케이션 코드 작업방식과 유사

Plan
Write 단계에서의 피드백이나, 변경사항이 적합하면 커밋하고 플랜을 진행
terraform apply 명령이 인프라의 변경을 진행하기 전에 플랜을 보여줌

Apply
At this point, 
it's common to push your version control repository 
to a remote location for safekeeping.
```

</div>
</details>

<details>
<summary>3. When we want the most verbose information from terraform logging what severity should we set? </summary>
<div markdown="1">

🔥 정답 : Trace


```
💡해설

테라폼에는 TF_LOG 환경변수를 통해 활성화 할 수 있는 로그가 있음. 
Trace로 올라갈 수록 자세한 로그, Error는 애플리케이션을 사용할 수 없을 정도의 fatal한 로그를 의미함. 

Trace - Only when I would be "tracing" the code and trying to find one part of a function specifically. 

Debug - Information that is diagnostically helpful to people more than just developers (IT, sysadmins, etc.)

Info - Generally useful information to log (service start/stop, configuration assumptions, etc). Info I want to always have available but usually don't care about under normal circumstances. This is my out-of-the-box config level

Warn - Anything that can potentially cause application oddities, but for which I am automatically recovering. (Such as switching from a primary to backup server, retrying an operation, missing secondary data, etc.)

Error - Any error which is fatal to the operation, but not the service or application (can't open a required file, missing data, etc.). These errors will force user (administrator, or direct user) intervention. These are usually reserved (in my apps) for incorrect connection strings, missing services, etc.
```

</div>
</details>


<details>
<summary>4. Which is NOT a valid argument for remote-exec? </summary>
<div markdown="1">

🔥 정답 : interpreter


```
💡해설

interpreter is an argument available to local-exec
local-exec이랑 헷갈리지 말자. 

inline - This is a list of command strings. They are executed in the order they are provided. This cannot be provided with script or scripts.

script - This is a path (relative or absolute) to a local script that will be copied to the remote resource and then executed. This cannot be provided with inline or scripts.

scripts - This is a list of paths (relative or absolute) to local scripts that will be copied to the remote resource and then executed. They are executed in the order they are provided. This cannot be provided with inline or script.


```

</div>
</details>



<details>
<summary>5. Is this a valid configuration for remote-exec? </summary>
<div markdown="1">

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

🔥 정답 : interpreter


```
💡해설

interpreter is an argument available to local-exec
local-exec이랑 헷갈리지 말자. 

inline - This is a list of command strings. They are executed in the order they are provided. This cannot be provided with script or scripts.

script - This is a path (relative or absolute) to a local script that will be copied to the remote resource and then executed. This cannot be provided with inline or scripts.

scripts - This is a list of paths (relative or absolute) to local scripts that will be copied to the remote resource and then executed. They are executed in the order they are provided. This cannot be provided with inline or script.


```

</div>
</details>
---
title: "Aws::CodeDeployCommand::Errors::InvalidSignatureException - Signature expired:"
excerpt: 
categories:
  - gugumo
tags: 
permalink: /project/gugumo/codedeployError
toc: true
toc_sticky: true
date: 2024-06-01
last_modified_at: 2024-06-01
---
아래 내용은 사이드 프로젝트(gugumo)를 진행하면서 겪은 문제를 정리한 것입니다.  

---

코드 자동 배포를 설정한 이후에 배포가 안되는 문제가 발생해서 찾아봤다.  

```
The overall deployment failed because too many individual instances failed deployment, too few healthy instances are available for deployment, or some instances in your deployment group are experiencing problems.
```

다음과 같은 문제로 codeDeploy에서 배포가 안되고 있었는데, 이 메시지만 봐서는 정확한 이유를 찾기가 힘들어서 ec2의 로그를 확인해봤다.  

로그는 다음 명령어를 통해 확인할 수 있다.   

`cat /var/log/aws/codedeploy-agent/codedeploy-agent.log`

로그를 확인한 결과 다음과 같은 에러로 배포가 안되고 있었다.  

```
 Aws::CodeDeployCommand::Errors::InvalidSignatureException - Signature expired:
```

aws 공식 문서에서 위와 같은 에러를 찾아보았다.  

```
CodeDeploy 작업을 수행하려면 정확한 시간 참조가 필요합니다. 인스턴스의 날짜 및 시간이 올바르게 설정되지 않은 경우 배포 요청의 서명 날짜와 일치하지 않을 수 있으며, 이 날짜와 시간은 CodeDeploy 거부됩니다.

잘못된 시간 설정과 관련된 배포 실패를 피하려면 다음 항목을 참조하세요.
```

이유는 알았지만 어떻게 해결해야 할지는 찾지 못해서 일단 인증 정보를 삭제하고 `codeDeploy`를 재시작 해보았다.  

```
인증 정보 삭제
sudo rm -rf /root/.aws/credentials
codeDeploy 재시작
sudo systemctl restart codedeploy-agent
```

이렇게 하니 배포도 정상적으로 동작했다.  

얼마 전에 프론트단에서 코드 리팩토링 및 테스트를 진행하다가 수많은 요청이 발생되면서 서버가 다운된 적이 있었는데 그 때 뭔가 꼬이면서 시간이 안맞는 이슈가 발생한게 아닐까 싶다.  

---

일단 해결은 했는데 올바른 해결 방법인지는 모르겠다.  


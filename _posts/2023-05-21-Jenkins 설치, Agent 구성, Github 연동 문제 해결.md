---
layout: post
title:  "Jenkins 설치, Agent 구성, Github 연동 문제 해결"
description: "Jenkins 설치, Agent 구성, Github 연동 문제 해결"
date:   2023-05-21 21:55:00 +0900
categories: CI⁄CD
excerpt_separator: <!--more-->
---

# Jenkins 설치, Agent 구성, Github 연동 문제 해결

Oracle Cloud(OS는 Oracle Linux)에서 Jenkins 설치, 구성 중 생긴 문제들과 문제 해결 과정을 기록한다. 설치한 Jenkins 버전은 2.387.3 이다.
<!--more-->

우선 sudo yum install jenkins 명령어로 Jenkins를 설치한다. 설치가 완료되고 sudo service jenkins start 명령어로 Jenkins 서비스를 시작하려고 하니 프로세스는 실행되는데 정상적인 상태가 아닌 거 같다. 실행 결과에 Failed 라는 문구와 함께 Job for jenkins.service failed because a timeout was exceeded 라는 메시지가 뜬다. journalctl -xe 명령어로 자세한 내용을 보려고 해도 timeout 부분 외에 의미 있는 내용을 찾지 못했다. 다른 네트워크 문제가 있는 게 아닌가 한참을 찾아 헤매다 알아낸 결론은 정말 단순 타임아웃 문제가 맞았다. /lib/systemd/system/jenkins.service 설정에서 TimeoutStartSec 설정의 숫자를 높이는 걸로 해결됐다.

그렇게 해서 설치는 잘 끝났고 내가 원하는 작업들을 수행해줄 Agent를 설치한다. Agent는 Controller에 연결해서 실행시키는 방법과 SSH로 실행시키는 방법이 있다. 이전에 Docker와 SSH를 이용해 실행시키는 방법을 이용했는데 과정이 복잡해서 이번엔 Controller에 연결해서 실행시키는 방법을 해보기로 했다. 이 방법은 TCP를 이용하는 방법과 웹소켓을 이용하는 방법 중에서 하나를 택해야 한다. Node 설정에서 Use Websocket을 체크하지 않으면 TCP로 이용하게 되는데, 기본 설정에서 TCP가 Disable 되어있으면 에러 문구가 표시된다. Jenkins 관리 > Configure Global Security 설정에서 Agents의 TCP port for inbound agents 부분을 Random이나 Fixed로 바꾸면 된다. 지금 같은 경우는 클라우드 환경이기 때문에 Fixed로 설정했다. Node의 나머지 설정은 기본으로 두고 설정을 완료하면 Agent 실행 방법이 다음과 같이 나온다.

```
curl -sO <https://젠킨스호스트/jnlpJars/agent.jar>
java -jar agent.jar -jnlpUrl <https://호스트/manage/computer/permanent/jenkins-agent.jnlp> -secret 보안키 -workDir "/작업경로"
```

나와있는대로 실행을 해보니 타임아웃 문제가 여기서도 발생했다.

```
INFO: Connecting to <젠킨스호스트:포트> (retrying:2)
java.io.IOException: Failed to connect to <호스트명:포트>
        at org.jenkinsci.remoting.engine.JnlpAgentEndpoint.open(JnlpAgentEndpoin
t.java:243)
```

firewall-cmd 로 내부 방화벽은 포트 개방을 했기 때문에 내부에서 차단하는 건 아니었다. Cloud에 등록된 보안그룹 규칙이 문제였는데, Agent 실행에 사용할 포트 관련 규칙을 등록할 때 소스 포트는 범위를 한정하지 않고 대상 포트만 범위를 한정해야 정상적으로 동작한다. 규칙 등록 후 다시 실행하니 정상적으로 연결되고 Node 상태창에서도 ‘Agent is connected.’ 라는 메시지가 나온다.

Agent까지 설치 했으니 이젠 새로운 프로젝트를 생성한다. [https://젠킨스호스트/view/all/newJob](https://xn--9t4ba423bv7jrtctuh/view/all/newJob) 으로 들어가거나 Dashboard에서 새로운 Item 버튼을 클릭 후 새로운 project를 생성한다. Github Project 정보를 적고 소스 코드 관리 부분에서 Git Repository 정보를 입력한다. Credential을 등록한 게 없으면 Credentials 부분에 Add 버튼을 눌러서 그 창에서 바로 등록하면 된다. Credential Kind는 Secret File이나 Secret Key가 아니라 Username with password를 선택한다. username에는 Github 사용자 ID, 비밀번호는 GIthub의 [Personal Access Token](https://github.com/settings/tokens)을 입력하면 된다. Token의 scope는 보통 repo와 admin:repo\_hook이 있으면 되는데 사용 환경에 따라 더 필요할 수도 있다. 생성한 Credential을 선택하면 Github 관련 설정은 끝난다. 이 때 Jenkins가 설치된 시스템에 Git이 설치가 안 돼있으면 에러가 난다. sudo yum install git으로 설치하면 된다.

이렇게 하면 Github 연동까지 완료된다. 추가 설정을 안 해서 수동 빌드만 가능하다. 프로젝트 페이지에서 지금 빌드를 누르면 Github에서 Repository의 최신 소스를 Jenkins 작업 폴더에 가져온다.

여기까지 하는데 오래도 걸렸다. 배포 자동화 좀 해본다고 도메인 사고 거의 1년은 걸린 거 같다. 그 사이에 도메인 만료 시점이 다가와서 도메인 갱신도 했다. 스스로 너무 게을렀음을 느낀다. 첫 빌드를 성공했지만 빌드가 애플리케이션 빌드도 아니고 그냥 Repository에서 소스 가져온 거에 불과하다. 다음은 webhook 설정과 애플리케이션 빌드 설정을 해봐야겠다.
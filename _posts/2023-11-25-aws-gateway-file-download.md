---
title: SCP로 EC2의 파일을 로컬로 복사할 때 subsystem request filed on channel 0 에러
categories:
  - tinytip
tags:
  - AWS
  - EC2
  - SCP
---

#### 배경

회사 EC2는 바로 접속 가능한게 아니라, gateway를 경유해서 접속하도록 되어있다. 이렇게 접속 가능한 EC2에 있는 파일을 로컬에 다운받고 싶을 때는 경유해서 다운받을 수 있도록 해야한다. 이에 대한 가이드로 Jumpbox 기능(openssh 7.3이상 버젼에서 사용 가능)을 이용한게 있어서 따라하다가 마지막 단계에서 `subsystem request filed on channel 0`이라는 에러와 마주쳤다.

#### 해결

-O 옵션을 추가해서 성공했다.

```bash
scp -o "ProxyJump <id>@gw-addr" -O <id>@<ip>:<source> <target>   ### source와 target은 각각 remote, local의 경로.
```

`-O` option은 파일을 받는데 내부적으로 SFTP프로토콜 대신 SCP old protocol을 사용하라는 뜻이라고 한다. SFTP를 구현하지 않은 서버와 통신하거나, 하위 호환성을 맞추거나, 특정 wildcard를 사용할 때 SFTP의 사용과 충돌하면 사용하면 된다고 한다.

* https://stackoverflow.com/questions/74311661/subsystem-request-failed-on-channel-0-scp-connection-closed-macbook
* https://github.com/PowerShell/Win32-OpenSSH/issues/1945
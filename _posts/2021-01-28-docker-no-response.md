---
title: docker command가 먹히지 않을 때
excerpt_separator: "<!--more-->"
categories:
  - tinytip
tags:
  - docker
---

리눅스 머신을 재시작 했을 때, docker 커맨드가 먹히지 않는 경우가 있다.

run만 안되는 경우는 단순히 deamon 재시작만 해주면 되나, 나의 경우는 `docker --version`같은 기본적인 커맨드도 완전 먹통이 되는 경우.
처음엔 당황했는데, [구글링으로 해결 방법을 찾았다](https://forums.docker.com/t/what-to-do-when-all-docker-commands-hang/28103/5).

* docker를 서비스를 멈추고 `sudo service docker stop`
* 문제가 되는 파일들을 지운다

```bash
sudo rm -rf /var/run/docker
sudo rm /var/run/docker.*
```

* docker 서비스를 다시 시작한다 `sudo service docker start`

이제 다시 docker 커맨드를 날리거나 컨테이너를 실행시키면 잘 동작한다 :)

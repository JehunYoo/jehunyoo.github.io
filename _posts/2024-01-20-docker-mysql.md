---
title: "Docker 첫 사용기 - MySQL 사용하기"
date: 2024-01-20 19:00:00 +0900
tags: ["docker", "mysql"]
categories: [Docker]
toc: true
math: false
img_path: /assets/img/
---

# Introduction

최근에 MySQL을 프로젝트에서 사용하기로 결정하기도 했고, MySQL에 대해 깊이 있게 공부하기 위해서 맥북에 설치가 필요했다. \
그냥 로컬에 설치하는건 어렵지 않지만 MariaDB도 설치해야 하는 경우에 애를 먹었던 기억이 있어서 다른 방법으로 설치를 하고 싶었다. \
(정확하지는 않지만) 두 데이터베이스가 설치 경로를 공유하는 문제였던 것으로 기억한다.

그래서 생각해낸 방법이 Docker이다. \
Docker를 사용하면 심지어 두 데이터베이스를 필요에 따라 동시에 사용할 수도 있을 것이다. 

# Docker에 대해서 간단하게

Docker image와 container에 대해서 간단하게 설명해보겠다. \
image는 어떤 애플리케이션을 위한 환경을 모두 담아둔 것이다. \
container는 이 image를 바탕으로 호스트에서 동작하는 프로세스다.

따라서 나는 MySQL Docker image를 통해 container를 구성하고 필요할 때마다 MySQL을 사용할 계획이다. \
만약 나중에 MariaDB를 사용해야 한다면, 마찬가지로 MariaDB Docker image를 통해 container를 구성하고 필요할 때마다 MariaDB를 사용할 수 있다. \
만약 동시에 사용할 일이 생긴다면, 포트 포워딩 정도만 수정해주면 간단하게 해결될 것이다.

# Docker로 MySQL 사용하기

Docker가 설치되어 있다고 가정한다. \

1. `$ docker pull mysql:<tag>`
	- MySQL Docker 이미지를 다운로드 한다. 이때 원하는 tag를 지정할 수 있다. 지정하지 않으면 latest이다.
2. `$ docker images`
	- 이미지 대한 정보를 확인할 수 있다.
3. `$ docker run --name <container-name> -e MYSQL_ROOT_PASSWORD=<password> -d -p 3306:3306 mysql:<tag>`
	- `run` 명령어로 Docker 컨테이너를 실행한다. 이때 어떤 이미지를 기반으로 할지 tag와 함께 명시한다.
	- `--name` 옵션으로 컨테이너의 이름을 부여할 수 있다.
	- `-e` 옵션으로 환경변수를 지정할 수 있다. 여기서는 root user가 사용할 비밀번호를 지정했다.
	- `-d` 옵션으로 컨테이너를 백그라운드에서 (detached) 실행할 수 있다. 실행 결과로 container id를 출력한다.
	- `-p` 옵션으로 포트포워딩을 설정할 수 있다. 호스트의 3306 포트를 컨테이너 내부의 3306 포트로 포워딩 해준다.
4. `$ docker ps -a`
	- `ps` 명령어로 컨테이너 목록을 조회할 수 있다.
	- `-a` 옵션으로 백그라운드 컨테이너도 함께 조회할 수 있다.
5. `$ docker exec -it <container-id> bash
	- `-i` 옵션으로 stdin을 활성화하고 컨테이너가 attach되어있지 않아도 stdin을 유지할 수 있다. bash에 명령을 입력하기 위해 사용한다.
	- `-t` 옵션으로 TTY 모드를 사용할 수 있다. 이 옵션을 사용하지 않으면 명령을 입력할 수는 있어도 shell이 표시되지 않는다.
	- 위의 두 옵션을 하나로 합쳐서 `-it`로 많이 사용한다.
	- `exec` 명령어는 실행되고 있는 컨테이너에 작업을 할 때 사용된다. 따라서 container id가 필요하다.
6. 접속된 상태에서 `$ mysql -u root -p`
	- MySQL 컨테이너에 접속한 상태에서 MySQL을 사용하는 것이다.
7. 다음은 컨테이너를 중지 / 시작 / 재시작하는 명령어이다.
	- `$ docker stop <container-id>`
	- `$ docker start <container-id>`
	- `$ docker restart <container-id>`


# `-e MYSQL_ROOT_PASSWORD=<password>`를 사용하지 않으면 어떻게 될까?

처음에 굳이 필요하지 않을 수도 있겠다고 생각해서 사용하지 않았었다. \
`docker run --name <container-name> -d -p 3306:3306 mysql:<tag>`를 실행했는데, 에러가 나지는 않았다. \
그러나 이후에 `docker exec -it <container-id> baash`를 실행하려고 하는데 다음과 같은 메시지가 나왔다.

```
Error response from daemon: Container is not running
```

난 분명히 방금 `docker run`으로 컨테이너를 실행했는데 무슨 소린가 싶어서 `docker ps -a`로 현재 컨테이너의 상태를 확인했다.

```
STATUS
Exited (1) 8 seconds ago
```

내가 생각한대로 컨테이너가 정상적으로 동작하지 않고 바로 종료된 것이다. \
구글링을 해보니 [도커 컨테이너는 컨테이너가 작업할 것이 없으면 즉시 종료된다](https://stackoverflow.com/questions/29599632/container-is-not-running)고 한다. \
이를 통해 MySQL 서버가 종료되었음을 추측할 수 있었고, 컨테이너를 실행하는 과정에서 문제가 있었다고 생각했다. \
그래서 처음에 사용하지 않았던 `-e` 옵션으로 환경 변수 `MYSQL_ROOT_PASSWORD`를 정의해주었더니 문제가 해결되었다.

# MySQL Docker Container를 종료하면 DB에 저장된 데이터가 사라질까?

테스트를 해보기 위해 위와 같이 컨테이너를 실행시키고, MySQL에 접속해서 새로운 데이터베이스, 테이블, 데이터를 추가했다. \
이후에 `docker stop`으로 해당 컨테이너를 중지시키고 `docker start`로 시작시키고 MySQL에 접속해서 확인해봤다.\
그 결과, 데이터가 그대로 남아있었다!

# [과제] 개발 워크스테이션 구축 및 Docker/Git 실습 보고서

## 1. 프로젝트 개요

미션 목표: 리눅스 터미널 조작, 파일 권한 관리, Docker 컨테이너 기반 웹 서버 구축, Git/GitHub 연동을 통한 개발 환경 구성 역량을 습득한다.

- 터미널 명령어를 활용하여 디렉토리 및 파일을 생성·수정·삭제하고 권한을 제어한다.
- Docker를 설치·점검하고, 컨테이너를 빌드·실행·운영하며 이미지와 컨테이너의 분리 원칙을 이해한다.
- Dockerfile로 커스텀 웹 서버 이미지를 제작하고, 포트 매핑·바인드 마운트·볼륨을 통해 격리 환경과 데이터 영속성을 검증한다.
- Git으로 로컬 버전 관리를 수행하고 GitHub 원격 저장소와 연동하여 협업 기반 소스코드 관리를 경험한다.

### 1.1 프로젝트 디렉토리 구조 설계

본 과제의 각종 실습 결과물(소스 코드, 문서화 리소스, 일시적인 명령/마운트 실습 등)이 혼재되지 않도록 **관심사 분리(Separation of Concerns)** 원칙을 기준으로 디렉토리를 구성하였습니다.

```text
1-1/
├── README.md               # 메인 과제 보고서
├── Dockerfile              # 웹 서버 컨테이너 빌드 명세서
├── src/                    # 서빙할 애플리케이션 소스 코드
│   └── index.html          # 커스텀 이미지 배포용 웹 문서
├── docs/                   # 문서화 관련 에셋 모음
│   └── screenshots/        # 보고서 첨부용 스크린샷 이미지
└── labs/                   # 격리된 환경에서 안전하게 수행하기 위한 실습용 디렉토리
    ├── permissions/        # 리눅스 파일/디렉토리 조작 및 권한 제어 실습용 (test.txt)
    └── bind-mount/         # Docker 바인드 마운트 데이터 매핑 실습용 (index.html)
```

**구성 기준 및 설명:**
- **`src/` (소스 코드 분리):** 실제 컨테이너 내부에 내장되어 서빙될 코드를 분리함으로써, 빌드 컨텍스트를 깔끔하게 유지합니다.
- **`docs/` (문서 자산 분리):** 메인 기술 문서(`README.md`)에 포함되는 부가적인 이미지 자산들을 하위로 분리하여 최상단 경로의 복잡도를 낮췄습니다.
- **`labs/` (실습 환경 격리):** 터미널 실습이나 볼륨 마운트 테스트처럼 일시적으로 파일을 생성 및 삭제하는 과정이 메인 애플리케이션 작업과 겹치지 않도록 그룹화했습니다.

---

## 2. 실행 환경

| 항목 | 내용 |
|------|------|
| OS | macOS 26.3 (Build: 25D125) |
| Shell/Terminal | Zsh 5.9 (arm64-apple-darwin25.0) / Terminal.app |
| Docker 버전 | Docker version 29.3.0, build 5927d80c76 |
| Git 버전 | git version 2.53.0 |

---

## 3. 수행 항목 체크리스트

| 항목 | 완료 | 결과 위치 |
|------|------|-----------|
| 터미널 기본 조작 로그 (pwd, ls, cd, mkdir, cp, mv, rm, cat, touch) | ✅ | [§4.1 터미널 조작](#41-터미널-조작-및-권한-실습) |
| 파일/디렉토리 권한 변경 실습 (파일 1개 + 디렉토리 1개) | ✅ | [§4.1 권한 변경](#41-터미널-조작-및-권한-실습) |
| Docker 설치 및 데몬 동작 확인 (--version, info) | ✅ | [§4.2 Docker 기본 운영](#42-docker-기본-운영) |
| Docker 기본 운영 명령 (images, ps, logs, stats) | ✅ | [§4.2 Docker 기본 운영](#42-docker-기본-운영) |
| hello-world 및 ubuntu 컨테이너 실행 실습 | ✅ | [§4.2 컨테이너 실행](#42-docker-기본-운영) |
| 커스텀 Dockerfile 기반 웹 서버 컨테이너 제작 | ✅ | [§4.3 커스텀 웹 서버](#43-커스텀-웹-서버-제작-dockerfile) |
| 포트 매핑 및 브라우저 접속 검증 (스크린샷 포함) | ✅ | [§4.3 포트 매핑](#43-커스텀-웹-서버-제작-dockerfile) |
| 바인드 마운트 반영 (호스트 변경 전/후 비교) | ✅ | [§4.4 바인드 마운트](#44-데이터-영속성-바인드-마운트--볼륨) |
| Docker 볼륨 영속성 검증 (컨테이너 삭제 전/후 비교) | ✅ | [§4.4 볼륨 영속성](#44-데이터-영속성-바인드-마운트--볼륨) |
| Git 설정 및 GitHub/VSCode 저장소 연동 | ✅ | [§4.5 Git & GitHub](#45-git--github-설정) |

---

## 4. 상세 수행 내용 및 검증 로그

### 4.1 터미널 조작 및 권한 실습

**수행 명령:** `pwd`, `ls -al`, `cd`, `mkdir`, `cp`, `mv`, `rm`, `cat`, `touch`

```bash
# 현재 위치 확인
$ pwd
/Users/iyeonjae/Desktop/shockwave/codyssey/1-1

# 목록 확인 (숨김 파일 포함)
$ ls -al
total 32
drwxr-xr-x  9 iyeonjae  staff    288 Mar 31 20:29 .
drwxr-xr-x  5 iyeonjae  staff    160 Mar 31 20:29 ..
-rw-r--r--  1 iyeonjae  staff    256 Mar 31 16:45 Dockerfile
drwxr-xr-x  3 iyeonjae  staff     96 Mar 31 20:29 labs/bind-mount
-rw-r--r--  1 iyeonjae  staff    854 Mar 31 20:29 index.html
drwxr-xr-x  3 iyeonjae  staff     96 Mar 31 16:45 lab
-rw-r--r--  1 iyeonjae  staff  17133 Mar 31 17:42 README.md
drwxr-xr-x  2 iyeonjae  staff     64 Mar 31 17:42 screenshots

# 디렉토리 생성 및 이동
$ mkdir -p labs/permissions
$ cd labs/permissions

# 빈 파일 생성
$ touch test.txt
$ ls -al
total 0
drwxr-xr-x  3 iyeonjae  staff   96 Mar 31 16:45 .
drwxr-xr-x  9 iyeonjae  staff  288 Mar 31 20:29 ..
-rw-r--r--  1 iyeonjae  staff    0 Mar 31 20:29 test.txt

# 파일 내용 입력 및 확인
$ echo "Hello, Codyssey!" > test.txt
$ cat test.txt
Hello, Codyssey!

# 파일 복사
$ cp test.txt test_backup.txt
$ ls
test_backup.txt
test.txt

# 파일 이름 변경
$ mv test_backup.txt test_old.txt
$ ls
test_old.txt
test.txt

# 파일 삭제
$ rm test_old.txt
$ ls
test.txt

# 상위 디렉토리로 복귀
$ cd ..
$ pwd
/Users/iyeonjae/Desktop/shockwave/codyssey/1-1
```

**권한 변경 실습 (chmod) — 파일 1개 + 디렉토리 1개:**

```bash
# --- [파일 권한 변경] ---
$ ls -l labs/permissions/test.txt

-rw-r--r--  1 iyeonjae  staff  17 Mar 31 20:29 labs/permissions/test.txt
# 기본 644: 소유자 rw- / 그룹 r-- / 기타 r--

$ chmod 755 labs/permissionss/permissions/test.txt
$ ls -l labs/permissions/test.txt
-rwxr-xr-x  1 iyeonjae  staff  17 Mar 31 20:29 labs/permissions/test.txt
# 755: 소유자 rwx / 그룹 r-x / 기타 r-x (실행 권한 부여)

$ chmod 644 labs/permissions/test.txt
$ ls -l labs/permissions/test.txt
-rw-r--r--  1 iyeonjae  staff  17 Mar 31 20:29 labs/permissions/test.txt
# 원복: 644

# --- [디렉토리 권한 변경] ---
$ ls -ld labs/permissions
drwxr-xr-x  3 iyeonjae  staff  96 Mar 31 20:30 lab
# 기본 755: 소유자 rwx / 그룹 r-x / 기타 r-x

$ chmod 700 labs/permissions
$ ls -ld labs/permissions
drwx------  3 iyeonjae  staff  96 Mar 31 20:30 lab
# 700: 소유자만 rwx, 그룹/기타 접근 불가

$ chmod 755 labs/permissions
$ ls -ld labs/permissions
drwxr-xr-x  3 iyeonjae  staff  96 Mar 31 20:30 lab
# 원복: 755
```

> **권한 표기 규칙:**
> - `rwx` = read(4) + write(2) + execute(1)
> - `755` = 소유자(7=rwx) / 그룹(5=r-x) / 기타(5=r-x)
> - `644` = 소유자(6=rw-) / 그룹(4=r--) / 기타(4=r--)
> - 절대 경로 예시: `/Users/iyeonjae/Desktop/shockwave/codyssey/1-1`
> - 상대 경로 예시: `./labs/permissions/test.txt`

---

### 4.2 Docker 기본 운영

**설치 확인:**

```bash
$ docker --version
Docker version 29.3.0, build 5927d80c76

$ docker info
Client: Docker Engine - Community
 Version:    29.3.0
 Context:    desktop-linux
 Debug Mode: false
 Plugins:
  buildx: Docker Buildx (Docker Inc.)
    Version:  v0.19.2-desktop.1
  compose: Docker Compose (Docker Inc.)
    Version:  v2.31.0-desktop.2

Server:
 Containers: 3
  Running: 3
  Paused:  0
  Stopped: 0
 Images: 3
 Server Version: 27.4.0
 Storage Driver: overlay2
 OSType: linux
 Architecture: aarch64
 CPUs: 8
 Total Memory: 7.654GiB
 Name: docker-desktop
```

**이미지 및 컨테이너 확인:**

```bash
$ docker images
REPOSITORY      TAG         IMAGE ID       CREATED          SIZE
my-web          latest      0c277365c6ad   4 minutes ago    181MB
nginx           latest      f0bb7029dace   6 days ago       181MB
hello-world     latest      eb84fdc6f2a3   7 days ago       5.2kB
postgres        16-alpine   0f7a666480d0   4 weeks ago      272MB
ubuntu          latest      e3847ac055b4   5 weeks ago      101MB
qdrant/qdrant   latest      8b8cd3bda28b   5 weeks ago      202MB
n8nio/n8n       2.7.3       fc1b8eb32d33   7 weeks ago      999MB

$ docker ps -a
CONTAINER ID   IMAGE                COMMAND                  CREATED          STATUS                PORTS                              NAMES
77b4fa190681   my-web               "/docker-entrypoint.…"   3 minutes ago    Up 3 minutes          0.0.0.0:8080->80/tcp               web-container
56a996a5bc36   qdrant/qdrant        "./entrypoint.sh"        4 days ago       Up 4 days             0.0.0.0:6333-6334->6333-6334/tcp   youthful_leakey
339554c7d5a3   n8nio/n8n:2.7.3      "tini -- /docker-ent…"   4 weeks ago      Up 4 days             0.0.0.0:5678->5678/tcp             n8n
b01a86bb1494   postgres:16-alpine   "docker-entrypoint.s…"   4 weeks ago      Up 4 days (healthy)   5432/tcp                           self-hosted-ai-starter-kit-postgres-1
```

**hello-world 컨테이너 실행:**

```bash
$ docker run --rm hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
58dee6a49ef1: Pull complete
Digest: sha256:452a468a4bf985040037cb6d5392410206e47db9bf5b7278d281f94d1c2d0931
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (arm64v8)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.
```

**ubuntu 컨테이너 내부 진입 실습:**

```bash
$ docker run -it --rm ubuntu bash
root@34e7f4672a73:/# ls /
bin  boot  dev  etc  home  lib  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@34e7f4672a73:/# echo "Hello from container"
Hello from container
root@34e7f4672a73:/# exit
exit
```

> **컨테이너 종료/유지 방식 차이:**
> - `docker run -it ... bash` → 셸 종료(exit) 시 컨테이너도 함께 종료됨
> - `docker run -d ...` → 백그라운드 실행, 컨테이너가 계속 구동
> - `docker exec -it <name> bash` → 실행 중인 컨테이너에 추가 셸 접속 (종료해도 컨테이너는 유지)

**컨테이너 로그 및 리소스 확인:**

```bash
$ docker logs web-container
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2026/03/31 11:31:21 [notice] 1#1: using the "epoll" event method
2026/03/31 11:31:21 [notice] 1#1: nginx/1.29.7
2026/03/31 11:31:21 [notice] 1#1: built by gcc 14.2.0 (Debian 14.2.0-19)
2026/03/31 11:31:21 [notice] 1#1: OS: Linux 6.10.14-linuxkit
2026/03/31 11:31:21 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2026/03/31 11:31:21 [notice] 1#1: start worker processes
2026/03/31 11:31:21 [notice] 1#1: start worker process 29
2026/03/31 11:31:21 [notice] 1#1: start worker process 30
2026/03/31 11:31:21 [notice] 1#1: start worker process 31
2026/03/31 11:31:21 [notice] 1#1: start worker process 32

$ docker stats web-container --no-stream
CONTAINER ID   NAME            CPU %     MEM USAGE / LIMIT     MEM %     NET I/O     BLOCK I/O     PIDS
77b4fa190681   web-container   0.00%     7.531MiB / 7.654GiB   0.10%     746B / 0B   0B / 8.19kB   9
```

> **[이미지와 컨테이너의 관점별 차이]**
> - **빌드(Build):** 이미지는 애플리케이션과 실행 환경이 포함된 **불변(Immutable)의 템플릿**이다.
> - **실행(Run):** 컨테이너는 이미지를 기반으로 메모리에 적재되어 실제 동작하는 **격리된 프로세스**다.
> - **변경(Modify):** 컨테이너 내부의 변경사항은 임시적이다. 영구 반영을 위해서는 Dockerfile을 새로 빌드하거나 볼륨을 마운트해야 한다.

---

### 4.3 커스텀 웹 서버 제작 (Dockerfile)

**선택 베이스:** `nginx:latest` (공식 NGINX 이미지)

**커스텀 포인트:**
- 직접 작성한 `index.html`을 NGINX 웹 루트에 배포
- `LABEL`로 메타데이터(maintainer, description) 추가
- 환경 변수(`AUTHOR`, `COURSE`) 설정으로 설정과 코드 분리 시연

**Dockerfile:**

```dockerfile
FROM nginx:latest

LABEL maintainer="iyeonjae"
LABEL description="E1-1 Custom NGINX Web Server"

ENV AUTHOR="iyeonjae" \
    COURSE="DevOps E1-1"

COPY src/index.html /usr/share/nginx/html/index.html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

**빌드 및 실행:**

```bash
$ docker build -t my-web .
#0 building with "desktop-linux" instance using docker driver

#1 [internal] load build definition from Dockerfile
#1 transferring dockerfile: 332B done
#1 DONE 0.0s

#2 [internal] load metadata for docker.io/library/nginx:latest
#2 DONE 2.3s

#3 [internal] load .dockerignore
#3 transferring context: 2B done
#3 DONE 0.0s

#4 [internal] load build context
#4 transferring context: 930B done
#4 DONE 0.0s

#5 [1/2] FROM docker.io/library/nginx:latest@sha256:7150b3a3...
#5 DONE 3.2s

#6 [2/2] COPY src/index.html /usr/share/nginx/html/index.html
#6 DONE 0.1s

#7 exporting to image
#7 writing image sha256:0c277365c6ad... done
#7 naming to docker.io/library/my-web done
#7 DONE 0.0s

$ docker run -d -p 8080:80 --name web-container my-web
77b4fa190681ab0d1d55a4e22cc46d5a985bc5c0fa7e8fbe72b04c18caa51a0e
```

**포트 매핑 접속 검증:**

```bash
$ curl -I localhost:8080
HTTP/1.1 200 OK
Server: nginx/1.29.7
Date: Tue, 31 Mar 2026 11:31:34 GMT
Content-Type: text/html
Content-Length: 854
Last-Modified: Tue, 31 Mar 2026 11:29:33 GMT
Connection: keep-alive
ETag: "69cbb01d-356"
Accept-Ranges: bytes
```

→ **HTTP 200 OK** 응답 확인.

브라우저에서 `http://localhost:8080` 접속 시 커스텀 페이지 정상 출력 (주소창 포함 스크린샷):

![브라우저 접속 화면](./docs/screenshots/browser-8080.png)

> **포트 매핑이 필요한 이유:** 컨테이너는 호스트와 격리된 네트워크 namespace에서 동작한다. 컨테이너 내부 80 포트에 외부에서 접근하려면 `-p 8080:80`처럼 호스트 포트를 컨테이너 포트에 명시적으로 연결해야 한다.

---

### 4.4 데이터 영속성 (바인드 마운트 / 볼륨)

#### 바인드 마운트

호스트의 `labs/bind-mount/` 디렉토리를 컨테이너 웹 루트에 마운트:

```bash
$ mkdir -p labs/bind-mount
$ echo "<h1>Bind Mount Test - Original</h1>" > labs/bind-mount/index.html

$ docker run -d -p 8081:80 --name bind-nginx \
    -v $(pwd)/labs/bind-mount:/usr/share/nginx/html nginx:latest

# [수정 전] 응답 확인
$ curl -s localhost:8081
<h1>Bind Mount Test - Original</h1>

# 호스트 파일 수정 (컨테이너 재시작 없이)
$ echo "<h1>Bind Mount - Changes from host are reflected immediately</h1>" > labs/bind-mount/index.html

# [수정 후] 응답 확인
$ curl -s localhost:8081
<h1>Bind Mount - Changes from host are reflected immediately</h1>
```

→ **호스트 파일 수정이 컨테이너에 즉시 반영**됨을 확인.

#### Docker 볼륨 영속성

```bash
# 볼륨 생성
$ docker volume create my-data
my-data

# 볼륨 목록 확인
$ docker volume ls
DRIVER    VOLUME NAME
local     my-data
local     self-hosted-ai-starter-kit_n8n_storage
local     self-hosted-ai-starter-kit_postgres_storage
local     self-hosted-ai-starter-kit_qdrant_storage

# 볼륨을 마운트하여 데이터 저장
$ docker run -d --name volume-ubuntu -v my-data:/data ubuntu \
    bash -c "echo 'persistent data test' > /data/hello.txt && sleep 30"

# [삭제 전] 데이터 확인
$ docker exec volume-ubuntu cat /data/hello.txt
persistent data test

# 컨테이너 삭제
$ docker rm -f volume-ubuntu
volume-ubuntu

# [삭제 후] 새 컨테이너로 동일 볼륨 마운트 → 데이터 유지 확인
$ docker run --rm -v my-data:/data ubuntu cat /data/hello.txt
persistent data test
```

→ **컨테이너를 삭제해도 볼륨 데이터는 유지**됨을 확인.

> **Docker 볼륨:** Docker가 관리하는 영속 스토리지로, 컨테이너의 생명주기와 독립적이다. 컨테이너를 삭제해도 데이터가 남아 있어 DB 데이터, 설정 파일 등 보존이 필요한 데이터에 적합하다.

---

### 4.5 Git & GitHub 설정

**Git 사용자 설정:**

```bash
$ git config --global user.name "이연재"
$ git config --global user.email "leeyj0304@gmail.com"
$ git config --global init.defaultBranch main

$ git config --list
credential.helper=osxkeychain
user.name=이연재
user.email=leeyj0304@gmail.com
filter.lfs.clean=git-lfs clean -- %f
filter.lfs.smudge=git-lfs smudge -- %f
filter.lfs.process=git-lfs filter-process
filter.lfs.required=true
core.autocrlf=input
init.defaultbranch=main
core.repositoryformatversion=0
core.filemode=true
core.bare=false
core.logallrefupdates=true
core.ignorecase=true
core.precomposeunicode=true
remote.origin.url=https://github.com/eajnoeyeel/codyssey.git
remote.origin.fetch=+refs/heads/*:refs/remotes/origin/*
branch.main.remote=origin
branch.main.merge=refs/heads/main
```

**로컬 저장소 커밋:**

```bash
$ cd ~/Desktop/shockwave/codyssey
$ git add 1-1/
$ git commit -m "feat: E1-1 개발 워크스테이션 구축 과제 제출"
[main (root-commit) 5b95ca4] feat: E1-1 개발 워크스테이션 구축 과제 제출
 7 files changed, 280 insertions(+)
 create mode 100644 1-1/Dockerfile
 create mode 100644 1-1/README.md
 create mode 100644 1-1/labs/bind-mount/index.html
 create mode 100644 1-1/src/index.html
 create mode 100644 1-1/labs/permissions/test.txt

$ git log --oneline
5b95ca4 (HEAD -> main) feat: E1-1 개발 워크스테이션 구축 과제 제출
```

**GitHub 원격 저장소 Push:**

```bash
$ git push -u origin main
To https://github.com/eajnoeyeel/codyssey.git
 * [new branch]      main -> main
branch 'main' set up to track 'origin/main'.
```

**VSCode GitHub 연동:**

VSCode 좌측 Source Control 패널(Ctrl+Shift+G)에서 GitHub 계정으로 로그인 후, `eajnoeyeel/codyssey` 원격 저장소와 연동 완료. 이후 VSCode UI를 통해 변경 사항을 커밋·푸시하여 정상 동작 확인.

![VSCode GitHub 연동 화면](./docs/screenshots/vscode-github.png)

> **Git과 GitHub의 역할 차이:**
> - **Git** — 로컬 컴퓨터에서 파일의 변경 이력을 추적·관리하는 분산 버전 관리 도구 (커밋, 브랜치, 병합 등)
> - **GitHub** — Git 저장소를 원격으로 호스팅하여 팀 간 공유·협업·백업을 가능하게 하는 플랫폼

---

## 5. 트러블슈팅 (3건)

### 트러블슈팅 1: `docker` 명령어 인식 안 됨

| 항목 | 내용 |
|------|------|
| **문제 상황** | 터미널에서 `docker --version` 실행 시 `zsh: command not found: docker` 출력 |
| **원인 가설** | Docker가 설치되지 않았거나 PATH에 등록되지 않음 |
| **확인** | `which docker` → 결과 없음. macOS에는 Docker가 기본 포함되어 있지 않음 |
| **해결** | Docker Desktop 설치 후 앱을 실행하면 내부 Docker 엔진이 자동으로 구동됨. 이후 `docker --version` 정상 출력 확인 |

### 트러블슈팅 2: 바인드 마운트 시 파일이 보이지 않는 문제

| 항목 | 내용 |
|------|------|
| **문제 상황** | `-v ./labs/bind-mount:/usr/share/nginx/html`로 마운트했으나 컨테이너에서 파일이 비어 있음 |
| **원인 가설** | 상대 경로 `./labs/bind-mount`가 Docker 엔진 기준으로 올바르게 해석되지 않을 수 있음 |
| **확인** | `docker inspect bind-nginx`로 Mounts 확인 → Source 경로가 예상과 다르게 설정됨 |
| **해결** | `$(pwd)/labs/bind-mount`로 절대 경로를 명시하여 마운트 → 정상 동작 확인. Docker 바인드 마운트는 절대 경로 사용이 안전함 |

### 트러블슈팅 3: 포트 매핑 실패 시 진단 프로세스

| 항목 | 내용 |
|------|------|
| **문제 상황** | `docker run -p 8080:80` 실행 시 `Bind for 0.0.0.0:8080 failed: port is already allocated` 와 같이 호스트 포트 충돌 에러가 발생함 |
| **진단 1 (호스트 점검)** | `lsof -i :8080` 명령을 통해 현재 해당 포트를 점유 중인 프로세스가 무엇인지 확인 |
| **진단 2 (도커 점검)** | `docker ps -a`를 통해 다른 컨테이너가 해당 포트를 이미 선점하고 있는지 확인 |
| **해결** | 충돌하는 프로세스나 기존 컨테이너를 종료(`kill`, `docker rm -f`)하거나, 새로 띄울 매핑 포트를 다른 빈 포트로 변경하여 실행함 |

---

## 6. 학습 결과 요약

| 항목 | 달성 | 설명 |
|------|------|------|
| 절대 경로 vs 상대 경로 | ✅ | `/Users/iyeonjae/...`(절대), `./labs/permissions/test.txt`(상대) |
| 파일 권한 rwx, 755, 644 해석 | ✅ | 755 = rwxr-xr-x, 644 = rw-r--r-- / 파일·디렉토리 각 1개 실습 |
| Dockerfile 커스텀 이미지 + 포트 매핑 | ✅ | nginx:latest 기반 빌드, 8080→80 매핑, curl 200 OK 확인 |
| Docker 볼륨 영속성 | ✅ | volume create → 컨테이너 삭제 후 데이터 유지 확인 |
| Git과 GitHub 역할 차이 | ✅ | Git = 로컬 버전 관리 / GitHub = 원격 호스팅·협업 플랫폼 |


# [Hands-on] 01. Environment setup - Docker

![](./img/hands_on.png)

<br>

# Contents

**[1. VM Instance 만들기](#1-vm-instance-만들기)**  
**[2. VM Instance 접속하기](#2-vm-instance-접속하기)**  
**[3. Docker 설치하기](#3-docker-설치하기)**  
**[4. Git clone하기 (실습파일 다운로드)](#4-git-clone하기-실습파일-다운로드)**  

---

<br>

## 1. VM Instance 만들기

실습을 위한 환경구성을 진행합니다.

먼저 SCP Console에 로그인합니다.  
- SCP Console : [https://cloud.samsungsds.com/console/](https://cloud.samsungsds.com/console/)

![](./img/scp_1.png)
> **email**, **비밀번호**, **OTP**를 이용해서 로그인 합니다.

<br><br><br>

![](./img/scp_2.png)
> 화면 상단에서 **프로젝트**를 선택합니다.

<br><br><br>

VPC와 Internet gateway 등 네트워킹 구성은 선행과정(개발)의 자원을 그대로 사용합니다.  
이번 과정의 실습을 위해서 한 개의 Virtual Server만 추가하도록 하겠습니다.


![](./img/scp_vs_1.png)
> 좌측 메뉴에서 자원관리를 선택하고, 상품별 자원관리에서 Compute > Virtual Server > Virtual Server를 선택하고, 우측상단의 상품신청 버튼을 클릭합니다.

<br><br><br>

![](./img/scp_vs_2.png)
> 다음과 같이 신청 정보를 선택하고 다음을 클릭합니다.
> - 종류 : 표준 > **Ubuntu**
> - 이미지 버젼 : **Ubuntu 20.04**

## ☢️ 😱 Ubuntu 버젼은 ***20.04*** 입니다. 꼭 확인해주세요. 꼭이요~ 🙏🏻
## 🙅🏻‍♀️ 22.04 아닙니다.

<br><br><br>

![](./img/scp_vs_3.png)
> 다음과 같이 상품 구성을 설정하고 다음을 클릭합니다.
> - 서버 수 : `1`
> - 서버 타입 : `Standard s1v2m4 (vCPU 2 | Memory 4G)`
> - 약정기간 : `None`
> - Block Storage : `dev-ubuntu` , `100GB`

<br><br><br>

![](./img/scp_vs_4.png)
화면과 같이 정보를 입력합니다.
> - Key pair : `dev-key-pair` (앞의 과정에서 생성한 Key pair)
> - 서버별 Prefix : `dev-ubuntu`
> - 네트워크 설정
>   - VPC : `devVPC` (앞의 과정에서 생성한 VPC)
>   - 일반 서브넷 : `devPublicSub (Public)` (앞의 과정에서 생성한 Subnet)
>   - NAT : `사용`
> - AZ 설정 : `AZ2`
> - Security Group : `선택` 버튼 클릭

## 🙅🏻‍♀️ AZ는 AZ1 아닙니다. ***AZ2***로 설정해주세요. 🙏🏻

<br><br><br>

![](./img/scp_vs_5.png)
> Security Group에서 devSG를 선택합니다. (앞의 과정에서 생성한 Security Group)

<br><br><br>

![](./img/scp_vs_6.png)
> 화면과 같이 devSG가 선택된 것을 확인하고 **다음**을 클릭합니다.

<br><br><br>

![](./img/scp_vs_7.png)
> 앞에서 입력한 내용을 확인하고, 문제가 없는 경우 완료를 클릭합니다.

<br><br><br>

이제 생성한 Virtual Server의 Inbound/Outbound 네트워크 허용을 위해 방화벽과 Security Group을 설정합니다.

![](./img/scp_vs_8.png)
> 좌측 메뉴에서 자원관리를 선택하고, 상품별 자원관리에서 Networking > Firewall을 선택하고, 우측 리스트의 Firewall(FW_IGW_devVPC)을 클릭합니다.

<br><br><br>

![](./img/scp_vs_9.png)
> **규칙**탭에서 `규칙추가`버튼을 클릭합니다.

<br><br><br>

![](./img/scp_vs_10.png)
> 화면과 같이 **Inbound 규칙**을 입력하고 **확인**을 클릭합니다.
> - 출발지 IP : 내 PC의 `IP` ([https://www.myip.com/](https://www.myip.com/)이나 [https://ifconfig.me/](https://ifconfig.me/) 에서 확인 가능. )
> - 목적지 IP : 10.0.1.0/24 (앞의 과정에서 설정한 Public subnet의 IP 대역)
> - 프로토콜 : TCP
> - 허용 포트 : 직접입력
>   - TCP : 22,80,8080,443,3000,30000-32767
> - 동작 : Allow
> - 방향 : Inbound

<br><br><br>

![](./img/scp_vs_11.png)
> 그림과 같이 새로운 규칙이 추가된 걸 확인합니다.

<br><br><br>

동일한 방법으로 Outbound 규칙도 하나 추가합니다.

![](./img/scp_vs_12.png)
> 화면과 같이 **Outbound 규칙**을 입력하고 **확인**을 클릭합니다.
> - 출발지 IP : 10.0.1.0/24 (앞의 과정에서 설정한 Public subnet의 IP 대역)
> - 목적지 IP : 0.0.0.0/0 ( 모든 IP Address를 의미합니다. ) 
> - 프로토콜 : TCP
> - 허용 포트 : 직접입력
>   - TCP : 80,443
> - 동작 : Allow
> - 방향 : Outbound

<br><br><br>

![](./img/scp_vs_13.png)
> 그림과 같이 새로운 규칙이 추가된 걸 확인합니다.

<br><br><br>

이제 Security Group에 규칙을 추가합니다.

![](./img/scp_vs_14.png)
> 좌측 메뉴에서 **자원관리**를 선택하고, 상품별 자원관리에서 Networking > Security Group을 선택하고, 우측 리스트의 `devSG`를 클릭합니다.

<br><br><br>

![](./img/scp_vs_15.png)
> **규칙**탭에서 `규칙추가`버튼을 클릭합니다.

<br><br><br>

![](./img/scp_vs_16.png)
> 화면과 같이 **Inbound 규칙**을 입력하고 **확인**을 클릭합니다.  
> - 방향 : Inbound 규칙
> - 대상 주소 : 내 PC의 `IP` ([https://www.myip.com/](https://www.myip.com/)이나 [https://ifconfig.me/](https://ifconfig.me/) 에서 확인 가능. )
> - 프로토콜 : TCP
> - 허용 포트 : 직접입력
>   - TCP : 22,80,8080,443,3000,30000-32767

<br><br><br>

![](./img/scp_vs_17.png)
> 그림과 같이 새로운 규칙이 추가된 걸 확인합니다.

<br><br><br>

![](./img/scp_vs_18.png)
> 화면과 같이 **Outbound 규칙**을 입력하고 **확인**을 클릭합니다.
> - 방향 : Outbound 규칙
> - 대상 주소 : 0.0.0.0/0 ( 모든 IP Address를 의미합니다. ) 
> - 프로토콜 : TCP
> - 허용 포트 : 직접입력
>   - TCP : 80,443

<br><br><br>

![](./img/scp_vs_19.png)
> 그림과 같이 새로운 규칙이 추가된 걸 확인합니다.

<br><br><br>

여기까지가 인프라 준비 입니다.  
이제 우리가 실습에 사용할 환경이 준비됐습니다. 👏🏻👏🏻👏🏻

<br><br><br><br><br>

## 2. VM Instance 접속하기

SSH 접속을 위해서는 다음을 먼저 확인해야 합니다.

- Virtual Server의 **NAT IP** ( 자원관리 > Compute > Virtual Server 에서 확인 )
- Key pair (**dev-key-pair.pem** 파일)
- SSH 접속을 위한 Inbound traffic (Port:22) 이 허용되어 있는지 확인 ( 앞에서 진행함 )

위의 내용을 확인 후 터미널 프로그램인 MobaXterm을 준비합니다.  
다음 순서대로 실행해주세요.

![](./img/mobaxterm_1.png)
> - [MobaXterm Download 페이지](https://mobaxterm.mobatek.net/download.html) 에서 Home Edition(Free) 하단 `Download now` 버튼을 클릭

<br><br><br>

![](./img/mobaxterm_2.png)
> - `MobaXterm Home Edition vxx.x (Portable edition)` 버튼을 클릭하여 zip파일을 다운로드 하고 적당한 위치에 압축해제 합니다.

<br><br><br>

![](./img/mobaxterm_3.png)
> 압축을 해재하면 그림과 같이 **MobaXterm_Personal_xx.x.exe** 파일이 있습니다. 이 파일을 실행합니다.

<br><br><br>

![](./img/mobaxterm_4.png)
> MobaXterm이 실행되면 화면 좌측 상단에서 `Session` 버튼을 클릭합니다.

<br><br><br>

![](./img/mobaxterm_5.png)
> 접속방식은 `SSH`를 선택하고 다음 정보를 입력합니다.  
> - **Remote host** : Virtual Server의 **NAT IP**
> - **Specify username** : vmuser

❗️ SSH 포트를 22에서 다른 포트로 변경한 경우 포트정보도 맞게 설정해주세요.

위와같이 설정한 다음 **Use private key** 영역의 파란 아이콘을 클릭합니다.

<br><br><br>

![](./img/mobaxterm_6.png)
> 미리 다운로드 받아놓은 **dev-key-pair.pem**파일을 찾아 선택합니다.

<br><br><br>

![](./img/mobaxterm_7.png)
> 그림과 같이 private key 파일이 선택되었으면, 아래 `OK` 버튼을 클릭하여 Virtual Server로 접속합니다.

<br><br><br>

![](./img/mobaxterm_8.png)
> 접속되면 위와같은 화면이 표시됩니다.  

<br><br><br>

![](./img/mobaxterm_9.png)
> 다음 번 접속부터는 좌측 **Quick connect**의 **User sessions**에 저장된 정보를 이용할 수 있습니다.

<br><br><br><br><br>

## 3. Docker 설치하기

실습을 위해서 Docker 설치를 진행합니다.
설치하는 방법은 아래 두 가지 방법이 있습니다.

- [Docker Desktop](https://docs.docker.com/desktop/) : One-click-install application for your Mac, Linux, or Windows
- [Docker Engine](https://docs.docker.com/engine/) : Open source containerization platform

우리 실습에는 앞에서 만든 **Ubuntu linux**에 **Docker Engine**을 설치해서 진행하도록 하겠습니다.

> 설치를 위해서는 [OS requirements](https://docs.docker.com/engine/install/ubuntu/#os-requirements)를 만족하는 조건이어야 합니다.  
> 우리가 만든 환경은 이 조건에 맞는 **Ubuntu 20.04(LTS)** 64bit 버젼 입니다.

### [Uninstall old versions](https://docs.docker.com/engine/install/ubuntu/#uninstall-old-versions)

기존에 설치된 버젼이 있거나 다시 설치를 진행하려고 하는 경우 먼저 기존버젼 삭제를 진행합니다.  
처음 설치를 하는 경우라면, 생략하고 다음 단계를 진행합니다.
```bash
ubuntu@ip-172-31-23-60:~$ sudo apt-get remove docker docker-engine docker.io containerd runc
Reading package lists... Done
Building dependency tree
Reading state information... Done
Package 'docker.io' is not installed, so not removed
E: Unable to locate package docker
E: Unable to locate package docker-engine
```

> 💻 명령어
>```bash
>sudo apt-get remove docker docker-engine docker.io containerd runc
>```

<br>

### [Install using the repository](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository)

Ubuntu의 [Advanced Packaging Tool (APT)](https://ubuntu.com/server/docs/package-management)를 이용해서 설치를 진행합니다.

먼저 package index를 업데이트 합니다.
```bash
ubuntu@ip-172-31-23-60:~$ sudo apt-get update
Get:1 http://security.ubuntu.com/ubuntu focal-security InRelease [114 kB]
Hit:2 http://us-east-1.ec2.archive.ubuntu.com/ubuntu focal InRelease
Get:3 http://us-east-1.ec2.archive.ubuntu.com/ubuntu focal-updates InRelease [114 kB]
Get:4 http://us-east-1.ec2.archive.ubuntu.com/ubuntu focal-backports InRelease [108 kB]
Get:5 http://security.ubuntu.com/ubuntu focal-security/main amd64 Packages [1993 kB]
Get:6 http://security.ubuntu.com/ubuntu focal-security/main Translation-en [326 kB]
Get:7 http://security.ubuntu.com/ubuntu focal-security/main amd64 c-n-f Metadata [12.2 kB]
Get:8 http://security.ubuntu.com/ubuntu focal-security/restricted amd64 Packages [1495 kB]
Get:9 http://security.ubuntu.com/ubuntu focal-security/restricted Translation-en [211 kB]
Get:10 http://us-east-1.ec2.archive.ubuntu.com/ubuntu focal/universe amd64 Packages [8628 kB]
...생략...
Fetched 25.7 MB in 4s (6120 kB/s)
Reading package lists... Done
```

> 💻 명령어
>```bash
>sudo apt-get update
>```

<br><br><br>

다음은, HTTPS를 이용하기 위해서 몇 가지 패키지를 설치합니다.  
```bash
ubuntu@ip-172-31-23-60:~$ sudo apt-get install -y ca-certificates curl gnupg lsb-release
Reading package lists... Done
Building dependency tree
Reading state information... Done
lsb-release is already the newest version (11.1.0ubuntu2).
lsb-release set to manually installed.
ca-certificates is already the newest version (20211016ubuntu0.20.04.1).
ca-certificates set to manually installed.
curl is already the newest version (7.68.0-1ubuntu2.15).
curl set to manually installed.
gnupg is already the newest version (2.2.19-3ubuntu2.2).
gnupg set to manually installed.
0 upgraded, 0 newly installed, 0 to remove and 11 not upgraded.
```

> 💻 명령어
>```bash
>sudo apt-get install -y ca-certificates curl gnupg lsb-release
>```

<br><br><br>

Docker GPG key를 추가합니다.
```bash
ubuntu@ip-172-31-23-60:~$ sudo mkdir -m 0755 -p /etc/apt/keyrings
ubuntu@ip-172-31-23-60:~$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

> 💻 명령어
>```bash
>sudo mkdir -m 0755 -p /etc/apt/keyrings
>curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
>
>```

<br><br><br>

Docker 설치를 위해서 APT Repository를 설정합니다.
```bash
ubuntu@ip-172-31-23-60:~$ echo \
>   "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
>   $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

> 💻 명령어
>```bash
>echo \
>  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
>  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
>```

<br><br><br>

다시 Package index를 업데이트 합니다.
```bash
ubuntu@ip-172-31-23-60:~$ sudo apt-get update
Hit:1 http://us-east-1.ec2.archive.ubuntu.com/ubuntu focal InRelease
Hit:2 http://us-east-1.ec2.archive.ubuntu.com/ubuntu focal-updates InRelease
Hit:3 http://us-east-1.ec2.archive.ubuntu.com/ubuntu focal-backports InRelease
Get:4 https://download.docker.com/linux/ubuntu focal InRelease [57.7 kB]
Get:5 http://security.ubuntu.com/ubuntu focal-security InRelease [114 kB]
Get:6 https://download.docker.com/linux/ubuntu focal/stable amd64 Packages [24.5 kB]
Fetched 196 kB in 1s (264 kB/s)
Reading package lists... Done
```

> 💻 명령어
>```bash
>sudo apt-get update
>```

<br><br><br>

그리고, 마지막으로 Docker를 설치합니다. (**Docker version : 20.10.23**)
```bash
ubuntu@ip-172-31-23-60:~$ sudo apt-get install -y docker-ce=5:20.10.23~3-0~ubuntu-focal docker-ce-cli=5:20.10.23~3-0~ubuntu-focal containerd.io docker-buildx-plugin docker-compose-plugin
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following additional packages will be installed:
  docker-ce-rootless-extras docker-scan-plugin pigz slirp4netns
Suggested packages:
  aufs-tools cgroupfs-mount | cgroup-lite
The following NEW packages will be installed:
  containerd.io docker-buildx-plugin docker-ce docker-ce-cli docker-ce-rootless-extras docker-compose-plugin docker-scan-plugin pigz slirp4netns
0 upgraded, 9 newly installed, 0 to remove and 11 not upgraded.
Need to get 139 MB of archives.
After this operation, 505 MB of additional disk space will be used.
Get:1 http://us-east-1.ec2.archive.ubuntu.com/ubuntu focal/universe amd64 pigz amd64 2.4-1 [57.4 kB]
Get:2 http://us-east-1.ec2.archive.ubuntu.com/ubuntu focal/universe amd64 slirp4netns amd64 0.4.3-1 [74.3 kB]
Get:3 https://download.docker.com/linux/ubuntu focal/stable amd64 containerd.io amd64 1.6.16-1 [27.7 MB]
Get:4 https://download.docker.com/linux/ubuntu focal/stable amd64 docker-buildx-plugin amd64 0.10.2-1~ubuntu.20.04~focal [25.9 MB]
Get:5 https://download.docker.com/linux/ubuntu focal/stable amd64 docker-ce-cli amd64 5:20.10.23~3-0~ubuntu-focal [42.6 MB]
Get:6 https://download.docker.com/linux/ubuntu focal/stable amd64 docker-ce amd64 5:20.10.23~3-0~ubuntu-focal [20.5 MB]
Get:7 https://download.docker.com/linux/ubuntu focal/stable amd64 docker-ce-rootless-extras amd64 5:23.0.1-1~ubuntu.20.04~focal [8765 kB]
...생략...
Processing triggers for man-db (2.9.1-1) ...
Processing triggers for systemd (245.4-4ubuntu3.19) ...
```

> 💻 명령어
>```bash
>sudo apt-get install -y docker-ce=5:20.10.23~3-0~ubuntu-focal docker-ce-cli=5:20.10.23~3-0~ubuntu-focal containerd.io docker-buildx-plugin docker-compose-plugin
>```

<br><br><br>


설치 후 다음 설정을 진행합니다. ([Docker Engine post-installation steps](https://docs.docker.com/engine/install/linux-postinstall/))

Docker daemon은 root 유저로 동작하고, Docker CLI(/usr/bin/docker)는 root 그룹/계정 권한을 가지고 있습니다.

```bash
ubuntu@ip-172-31-23-60:~$ ps -ef | grep -i dockerd
root        3039       1  0 08:56 ?        00:00:00 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
ubuntu      4078    1407  0 08:58 pts/0    00:00:00 grep --color=auto -i dockerd
ubuntu@ip-172-31-23-60:~$ ls -al /usr/bin/docker
-rwxr-xr-x 1 root root 50722320 Jan 19 17:34 /usr/bin/docker
```

root가 아닌 계정(우리 실습환경의 user는 `ubuntu` 입니다.)을 이용하여 Docker CLI를 사용하기 위해서 다음과 같이 진행합니다.

먼저 `docker`그룹을 추가합니다.

```bash
ubuntu@ip-172-31-23-60:~$ sudo groupadd docker
```

> 💻 명령어
>```bash
>sudo groupadd docker
>```
- 이미 docker 그룹이 있을수도 있습니다. (groupadd: group 'docker' already exists)

<br><br><br>

다음은 사용 중인 User(`ubuntu`)를 docker 그룹에 추가하고, 적용(docker 그룹으로 로그인)합니다.

```bash
ubuntu@ip-172-31-23-60:~$ sudo usermod -aG docker $USER
ubuntu@ip-172-31-23-60:~$ newgrp docker
```

> 💻 명령어
>```bash
>sudo usermod -aG docker $USER
>newgrp docker
>
>```

<br><br><br>

모두 정상적으로 설치되고 설정된 경우 다음과 같이 동작해야 합니다.
한 번 테스트 해보세요.
```bash
ubuntu@ip-172-31-23-60:~$ docker run --rm hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
2db29710123e: Pull complete
Digest: sha256:aa0cc8055b82dc2509bed2e19b275c8f463506616377219d9642221ab53cf9fe
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

> 💻 명령어
>```bash
>docker run --rm hello-world
>```

<br><br><br>

그리고, 실습에 필요한 몇 가지 패키지를 추가로 더 설치할게요.  
- net-tools : ifconfig, netstat 등의 명령어 사용
- tree : 디렉토리 구조를 보기위한 툴
- conntrack : 리눅스 커널의 network connection을 관리하고 추적
- jq : JSON 프로세서
```bash
ubuntu@ip-172-31-23-60:~$ sudo apt-get update
Hit:1 http://us-east-1.ec2.archive.ubuntu.com/ubuntu focal InRelease
Hit:2 http://us-east-1.ec2.archive.ubuntu.com/ubuntu focal-updates InRelease
Hit:3 http://us-east-1.ec2.archive.ubuntu.com/ubuntu focal-backports InRelease
Get:4 http://security.ubuntu.com/ubuntu focal-security InRelease [114 kB]
Hit:5 https://download.docker.com/linux/ubuntu focal InRelease
Fetched 114 kB in 1s (150 kB/s)
Reading package lists... Done
ubuntu@ip-172-31-23-60:~$ sudo apt-get install -y net-tools tree conntrack
Reading package lists... Done
Building dependency tree
Reading state information... Done
Suggested packages:
  nftables
The following NEW packages will be installed:
  conntrack net-tools tree
0 upgraded, 3 newly installed, 0 to remove and 13 not upgraded.
Need to get 270 kB of archives.
After this operation, 1083 kB of additional disk space will be used.
Get:1 http://us-east-1.ec2.archive.ubuntu.com/ubuntu focal/main amd64 conntrack amd64 1:1.4.5-2 [30.3 kB]
Get:2 http://us-east-1.ec2.archive.ubuntu.com/ubuntu focal/main amd64 net-tools amd64 1.60+git20180626.aebd88e-1ubuntu1 [196 kB]
Get:3 http://us-east-1.ec2.archive.ubuntu.com/ubuntu focal/universe amd64 tree amd64 1.8.0-1 [43.0 kB]
Fetched 270 kB in 0s (12.2 MB/s)
Selecting previously unselected package conntrack.
(Reading database ... 62118 files and directories currently installed.)
Preparing to unpack .../conntrack_1%3a1.4.5-2_amd64.deb ...
Unpacking conntrack (1:1.4.5-2) ...
Selecting previously unselected package net-tools.
Preparing to unpack .../net-tools_1.60+git20180626.aebd88e-1ubuntu1_amd64.deb ...
Unpacking net-tools (1.60+git20180626.aebd88e-1ubuntu1) ...
Selecting previously unselected package tree.
Preparing to unpack .../tree_1.8.0-1_amd64.deb ...
Unpacking tree (1.8.0-1) ...
Setting up net-tools (1.60+git20180626.aebd88e-1ubuntu1) ...
Setting up conntrack (1:1.4.5-2) ...
Setting up tree (1.8.0-1) ...
Processing triggers for man-db (2.9.1-1) ...
```

> 💻 명령어
>```bash
>sudo apt-get update
>sudo apt-get install -y net-tools tree conntrack jq
>
>```

<br><br><br><br><br>

## 4. Git clone하기 (실습파일 다운로드)
이후 진행되는 실습과정들에 사용되는 파일들을 다운로드 하겠습니다.

```bash
ubuntu@ip-172-31-23-60:~$ git clone https://github.com/mspt2/Docker-K8s.git
Cloning into 'Docker-K8s'...
remote: Enumerating objects: 419, done.
remote: Total 419 (delta 0), reused 0 (delta 0), pack-reused 419
Receiving objects: 100% (419/419), 105.58 MiB | 15.71 MiB/s, done.
Resolving deltas: 100% (42/42), done.
```

> 💻 명령어
>```bash
>git clone https://github.com/mspt2/Docker-K8s.git
>```
- **hands_on_files** 디렉토리 아래에 실습에 필요한 파일들이 있습니다.

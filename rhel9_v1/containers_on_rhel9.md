# RHEL9 상의 컨테이너

<br>
<br>

<hr>

<br>
<br>


## 1. 레드햇 리눅스 컨테이너

### 1.1 리눅스 컨테이너

#### 1.1.1 리눅스 컨테이너를 구현하는 핵심 기술

* 리소스 관리를 위한 컨트롤 그룹(cgroup)
* 프로세스 격리를 위한 네임스페이스
* 보안을 위한 SELinux
* 보안 멀티 테넌시

#### 1.1.2 컨테이너 엔진 없이 작동할 수 있는 CLI 도구 집합 (OCI와 호환)

|도구 이름|설명|
|:---|:---|
|podman|포드 및 컨테이너 이미지를 직접 관리하기 위해 (run, stop, start, ps, attach, exec 등)|
|buildah|컨테이너 이미지 빌드, 내보내기 및 서명|
|skopeo|이미지 복사, 검사, 삭제 및 서명|
|runc|podman 및 buildah에 컨테이너 실행 및 빌드 기능을 제공|
|crun|구성 가능한 선택적 런타임으로 루트리스 컨테이너에 대한 유연성, 제어 및 보안을 강화|
<br>

### 1.2 Podman, Buildah 및 Skopeo의 특성 

#### 1.2.1 장점

|장점|설명|
|:---|:---|
|rootless 모드에서 실행|rootless 컨테이너는 추가 권한 없이 실행되므로 훨씬 더 안전|
|필요한 데몬 없음|컨테이너를 실행하지 않는 경우 Podman이 실행되고 있지 않기 때문에 이러한 툴에는 유휴 상태의 리소스 요구 사항이 훨씬 적음<br><br>반면 Docker에는 데몬이 항상 실행 중임|
|기본 systemd 통합|odman을 사용하면 systemd 장치 파일을 생성하고 시스템 서비스로 컨테이너를 실행 가능|

#### 1.2.2 특성

* Podman, Buildah 및 CRI-O 컨테이너 엔진은 기본적으로 Docker 스토리지 위치 */var/lib/docker*를 사용하는 대신 동일한 백엔드 저장소 디렉터리인 */var/lib/containers*를 사용
* Podman, Buildah, CRI-O는 동일한 스토리지 디렉터리를 공유하지만 서로의 컨테이너와 상호 작용할 수 없음
  + 이러한 도구는 이미지를 공유할 수 있음
* Podman과 프로그래밍 방식으로 상호 작용하려면 Podman v2.0 RESTful API를 사용
  + rootful 환경과 rootless 환경에서 작동
<br>

### 1.3 RHEL에서 컨테이너 배포에 지원되는 아키텍처

|아키텍처|설명|
|:---|:---|
|AMD64 및 Intel 64|베이스 및 계층화된 이미지<br>32비트 아키텍처를 지원하지 않음|
|PowerPC 8 및 9 64비트|기본 이미지 및 대부분의 계층화된 이미지|
|64비트 IBM Z|기본 이미지 및 계층화된 이미지|
|ARM 64비트|기본 이미지만 해당|
<br>

### 1.4 컨테이너 도구 설치

#### 1.4.1 시스템 등록

실행 명령어
```bash
subscription-manager register --username 'shadowman@redhat.com' --password '<PASSWORD>'
```

실행 결과
```
[root@rhel94 ~]# subscription-manager register --username 'shadowman@redhat.com' --password '<PASSWORD>'
등록 대상: subscription.rhsm.redhat.com:443/subscription
시스템은 ID로 등록되어 있습니다: 521e211d-4752-45d9-8c78-aeb73e756cb6
등록된 시스템 이름: rhel94.thinkmore.net

[root@rhel94 ~]# 
```

#### 1.4.2 서브스크립션 구성

실행 명령어 - RHEL에 자동으로 서브스크립션 추가
```bash
subscription-manager attach --auto
```

또는,

실행 명령어 - 풀 ID로 RHEL에 자동으로 서브스크립션 추가
```bash
subscription-manager attach --pool <PoolID>
```

#### 1.4.3 메타 패키지 *container-tools* 설치

실행 명령어
```bash
dnf install -y container-tools
```

실행 결과
```
[root@rhel94 ~]# dnf install -y container-tools
서브스크립션 관리 저장소를 최신화하기.
Red Hat Enterprise Linux 9 for x86_64 - BaseOS (RPMs)                                          16 MB/s |  75 MB     00:04    
Red Hat Enterprise Linux 9 for x86_64 - AppStream (RPMs)                                       32 MB/s |  67 MB     00:02    
마지막 메타자료 만료확인(0:00:01 이전): 2025년 09월 04일 (목) 오후 01시 33분 58초.
종속성이 해결되었습니다.
==============================================================================================================================
 꾸러미                     구조              버전                          저장소                                       크기
==============================================================================================================================
설치 중:
 container-tools            noarch            1-14.el9                      rhel-9-for-x86_64-appstream-rpms            8.3 k
종속 꾸러미 설치 중:
 podman-docker              noarch            2:4.9.4-0.1.el9               rhel-9-for-x86_64-appstream-rpms            106 k
 podman-remote              x86_64            5:5.4.0-12.el9_6              rhel-9-for-x86_64-appstream-rpms             11 M
 python3-podman             noarch            3:5.3.0-1.el9                 rhel-9-for-x86_64-appstream-rpms            187 k
 python3-tomli              noarch            2.0.1-5.el9                   rhel-9-for-x86_64-appstream-rpms             37 k
 toolbox                    x86_64            0.2-1.el9_6                   rhel-9-for-x86_64-appstream-rpms            3.6 M
 udica                      noarch            0.2.8-2.el9                   rhel-9-for-x86_64-appstream-rpms             76 k

연결 요약
==============================================================================================================================
설치  7 꾸러미

전체 내려받기 크기: 15 M
설치된 크기 : 51 M
꾸러미 내려받기 중:
(1/7): container-tools-1-14.el9.noarch.rpm                                                     47 kB/s | 8.3 kB     00:00    
(2/7): python3-tomli-2.0.1-5.el9.noarch.rpm                                                   194 kB/s |  37 kB     00:00    
(3/7): udica-0.2.8-2.el9.noarch.rpm                                                           479 kB/s |  76 kB     00:00    
(4/7): python3-podman-5.3.0-1.el9.noarch.rpm                                                  1.1 MB/s | 187 kB     00:00    
(5/7): toolbox-0.2-1.el9_6.x86_64.rpm                                                          12 MB/s | 3.6 MB     00:00    
(6/7): podman-remote-5.4.0-12.el9_6.x86_64.rpm                                                 22 MB/s |  11 MB     00:00    
(7/7): podman-docker-4.9.4-0.1.el9.noarch.rpm                                                  23 kB/s | 106 kB     00:04    
------------------------------------------------------------------------------------------------------------------------------
합계                                                                                          3.2 MB/s |  15 MB     00:04     
연결 확인 실행 중
연결 확인에 성공했습니다.
연결 시험 실행 중
연결 시험에 성공했습니다.
연결 실행 중
  준비 중     :                                                                                                           1/1 
  설치 중     : toolbox-0.2-1.el9_6.x86_64                                                                                1/7 
  설치 중     : podman-remote-5:5.4.0-12.el9_6.x86_64                                                                     2/7 
  설치 중     : udica-0.2.8-2.el9.noarch                                                                                  3/7 
  설치 중     : podman-docker-2:4.9.4-0.1.el9.noarch                                                                      4/7 
  설치 중     : python3-tomli-2.0.1-5.el9.noarch                                                                          5/7 
  설치 중     : python3-podman-3:5.3.0-1.el9.noarch                                                                       6/7 
  설치 중     : container-tools-1-14.el9.noarch                                                                           7/7 
  구현 중     : container-tools-1-14.el9.noarch                                                                           7/7 
  확인 중     : python3-tomli-2.0.1-5.el9.noarch                                                                          1/7 
  확인 중     : container-tools-1-14.el9.noarch                                                                           2/7 
  확인 중     : podman-docker-2:4.9.4-0.1.el9.noarch                                                                      3/7 
  확인 중     : udica-0.2.8-2.el9.noarch                                                                                  4/7 
  확인 중     : python3-podman-3:5.3.0-1.el9.noarch                                                                       5/7 
  확인 중     : podman-remote-5:5.4.0-12.el9_6.x86_64                                                                     6/7 
  확인 중     : toolbox-0.2-1.el9_6.x86_64                                                                                7/7 
설치된 제품이 최신화되었습니다.

설치되었습니다:
  container-tools-1-14.el9.noarch         podman-docker-2:4.9.4-0.1.el9.noarch     podman-remote-5:5.4.0-12.el9_6.x86_64    
  python3-podman-3:5.3.0-1.el9.noarch     python3-tomli-2.0.1-5.el9.noarch         toolbox-0.2-1.el9_6.x86_64               
  udica-0.2.8-2.el9.noarch               

완료되었습니다!

[root@rhel94 ~]# 
```
* podman, buildah, skopeo 도구를 개별적으로 설치 가능

#### 1.4.4 (선택 사항) 패키지 *podman-docker*를 설치

실행 명령어
```bash
dnf install podman-docker
```

실행 결과
```
[root@rhel94 ~]# dnf install -y podman-docker
서브스크립션 관리 저장소를 최신화하기.
마지막 메타자료 만료확인(0:02:23 이전): 2025년 09월 04일 (목) 오후 01시 33분 58초.
꾸러미 podman-docker-2:4.9.4-0.1.el9.noarch가 이미 설치되어 있습니다.
종속성이 해결되었습니다.
==============================================================================================================================
 꾸러미                 구조            버전                                  저장소                                     크기
==============================================================================================================================
향상 중:
 podman                 x86_64          5:5.4.0-12.el9_6                      rhel-9-for-x86_64-appstream-rpms           17 M
 podman-docker          noarch          5:5.4.0-12.el9_6                      rhel-9-for-x86_64-appstream-rpms          104 k
종속 꾸러미 설치 중:
 passt                  x86_64          0^20250217.ga1e48a0-10.el9_6          rhel-9-for-x86_64-appstream-rpms          262 k
 passt-selinux          noarch          0^20250217.ga1e48a0-10.el9_6          rhel-9-for-x86_64-appstream-rpms           27 k

연결 요약
==============================================================================================================================
설치  2 꾸러미
향상  2 꾸러미

전체 내려받기 크기: 17 M
꾸러미 내려받기 중:
(1/4): passt-selinux-0^20250217.ga1e48a0-10.el9_6.noarch.rpm                                  140 kB/s |  27 kB     00:00    
(2/4): passt-0^20250217.ga1e48a0-10.el9_6.x86_64.rpm                                          1.2 MB/s | 262 kB     00:00    
(3/4): podman-docker-5.4.0-12.el9_6.noarch.rpm                                                634 kB/s | 104 kB     00:00    
(4/4): podman-5.4.0-12.el9_6.x86_64.rpm                                                        27 MB/s |  17 MB     00:00    
------------------------------------------------------------------------------------------------------------------------------
합계                                                                                           28 MB/s |  17 MB     00:00     
연결 확인 실행 중
연결 확인에 성공했습니다.
연결 시험 실행 중
연결 시험에 성공했습니다.
연결 실행 중
  준비 중     :                                                                                                           1/1 
  설치 중     : passt-0^20250217.ga1e48a0-10.el9_6.x86_64                                                                 1/6 
  구현 중     : passt-selinux-0^20250217.ga1e48a0-10.el9_6.noarch                                                         2/6 
  설치 중     : passt-selinux-0^20250217.ga1e48a0-10.el9_6.noarch                                                         2/6 
  구현 중     : passt-selinux-0^20250217.ga1e48a0-10.el9_6.noarch                                                         2/6 
  향상 중     : podman-5:5.4.0-12.el9_6.x86_64                                                                            3/6 
  향상 중     : podman-docker-5:5.4.0-12.el9_6.noarch                                                                     4/6 
  정리        : podman-docker-2:4.9.4-0.1.el9.noarch                                                                      5/6 
  구현 중     : podman-2:4.9.4-0.1.el9.x86_64                                                                             6/6 
  정리        : podman-2:4.9.4-0.1.el9.x86_64                                                                             6/6 
  구현 중     : passt-selinux-0^20250217.ga1e48a0-10.el9_6.noarch                                                         6/6 
  구현 중     : podman-2:4.9.4-0.1.el9.x86_64                                                                             6/6 
  확인 중     : passt-0^20250217.ga1e48a0-10.el9_6.x86_64                                                                 1/6 
  확인 중     : passt-selinux-0^20250217.ga1e48a0-10.el9_6.noarch                                                         2/6 
  확인 중     : podman-5:5.4.0-12.el9_6.x86_64                                                                            3/6 
  확인 중     : podman-2:4.9.4-0.1.el9.x86_64                                                                             4/6 
  확인 중     : podman-docker-5:5.4.0-12.el9_6.noarch                                                                     5/6 
  확인 중     : podman-docker-2:4.9.4-0.1.el9.noarch                                                                      6/6 
설치된 제품이 최신화되었습니다.

향상되었습니다:
  podman-5:5.4.0-12.el9_6.x86_64                             podman-docker-5:5.4.0-12.el9_6.noarch                            
설치되었습니다:
  passt-0^20250217.ga1e48a0-10.el9_6.x86_64                 passt-selinux-0^20250217.ga1e48a0-10.el9_6.noarch                

완료되었습니다!

[root@rhel94 ~]# 
```
* podman-docker 패키지는 Docker CLI 및 docker-api를 일치하는 Podman 명령으로 대신 교체
<br>
<br>

## 2. rootless 컨테이너

### 2.1 컨테이너 실행 권한

* 수퍼유저 권한(root 사용자)이 있는 사용자로 Podman, Skopeo 또는 Buildah와 같은 컨테이너 툴을 실행하는 것이 시스템에서 사용 가능한 모든 기능에 대한 전체 액세스 권한을 확보할 수 있도록 하는 가장 좋은 방법
* RHEL에서 일반적으로 제공되는 "Rootless Containers"라는 기능을 사용하면 일반 사용자로 컨테이너를 사용 가능
  + 시스템 관리자는 rootless 컨테이너 사용자를 설정
  + 일반 사용자의 컨테이너 활동이 손상될 수 있지만 사용자는 자신의 계정에서 대부분의 컨테이너 기능을 안전하게 실행

> [!WARNING]
> Docker와 같은 컨테이너 엔진을 사용하면 일반(non-root) 사용자로 Docker 명령을 실행할 수 있지만 해당 요청을 전송하는 Docker 데몬은 root로 실행됩니다. 따라서 일반 사용자는 컨테이너를 통해 시스템을 손상시킬 수 있는 요청을 할 수 있습니다. 
<br>

### 2.2 non-root 사용자의 podman 사용을 위한 설정

#### 2.2.1 podman 패키지를 설치

실행 명령어
```bash
dnf install -y podman
```

#### 2.2.2 새 사용자 추가 및 암호 설정

실행 명령어
```bash
useradd -c "Peter Lee" peter
passwd peter
```

실행 결과
```
[root@rhel94 ~]# useradd -c "Peter Lee" peter

[root@rhel94 ~]# passwd peter
peter 사용자의 비밀 번호 변경 중
새 암호: ******
새 암호 재입력: ******
passwd: 모든 인증 토큰이 성공적으로 업데이트 되었습니다.

[root@rhel94 ~]# 
```

#### 2.2.3 새 사용자에 대한 rootless podman 사용 권한 확인

실행 명령어
```bash
grep peter /etc/passwd
grep peter /etc/group
grep peter /etc/subuid
grep peter /etc/subgid
```

실행 결과
```
[root@rhel94 ~]# grep peter /etc/passwd
peter:x:1002:1002:Peter Lee:/home/peter:/bin/bash

[root@rhel94 ~]# grep peter /etc/group
peter:x:1002:

[root@rhel94 ~]# grep peter /etc/subuid
peter:231072:65536

[root@rhel94 ~]# grep peter /etc/subgid
peter:231072:65536

[root@rhel94 ~]#
```
* 사용자를 추가하면, rootless podman을 사용할 수 있도록 자동으로 구성됨
  + `useradd` 명령은 /etc/subuid 및 /etc/subgid 파일에서 액세스 가능한 사용자 및 그룹 ID의 범위를 자동으로 설정
  + /etc/subuid 또는 /etc/subgid를 수동으로 변경하는 경우, `podman system migrate` 명령을 실행하여 새 변경 사항 적용 필요

#### 2.2.4 새 사용자로 접속

실행 명령어
```bash
ssh peter@localhost
```

실행 결과
```
[root@rhel94 ~]# ssh peter@localhost
...<snip>...
peter@localhost's password: ******
Register this system with Red Hat Insights: insights-client --register
Create an account or view all your systems at https://red.ht/insights-dashboard

[peter@rhel94 ~]$
```

> [!WARNING]
> `su` 또는 `su -` 명령어를 사용하면 올바른 환경 변수를 설정하지 않으므로, 문제를 발생할 수 있습니다.

#### 2.2.5 컨테이너 이미지 가져오기

실행 명령어
```bash
podman pull registry.access.redhat.com/ubi9/ubi
podman images
```

실행 결과
```
[peter@rhel94 ~]$ podman pull registry.access.redhat.com/ubi9/ubi
Trying to pull registry.access.redhat.com/ubi9/ubi:latest...
Getting image source signatures
Checking if image destination supports signatures
Copying blob d3cc7dd07291 done   | 
Copying config b096626d71 done   | 
Writing manifest to image destination
Storing signatures
b096626d71c97a6d8f158dc5d054302156944908907fbc608b5cb35a165cdc84

[peter@rhel94 ~]$ podman images
REPOSITORY                           TAG         IMAGE ID      CREATED       SIZE
registry.access.redhat.com/ubi9/ubi  latest      b096626d71c9  14 hours ago  217 MB

[peter@rhel94 ~]$ 
```

#### 2.2.6 컨테이너를 실행하고 OS 버전 표시

실행 명령어
```bash
podman run --rm --name=myubi registry.access.redhat.com/ubi9/ubi \
  cat /etc/os-release
```

실행 결과
```
[peter@rhel94 ~]$ podman run --rm --name=myubi registry.access.redhat.com/ubi9/ubi \
  cat /etc/os-release
NAME="Red Hat Enterprise Linux"
VERSION="9.6 (Plow)"
ID="rhel"
ID_LIKE="fedora"
VERSION_ID="9.6"
PLATFORM_ID="platform:el9"
PRETTY_NAME="Red Hat Enterprise Linux 9.6 (Plow)"
ANSI_COLOR="0;31"
LOGO="fedora-logo-icon"
CPE_NAME="cpe:/o:redhat:enterprise_linux:9::baseos"
HOME_URL="https://www.redhat.com/"
DOCUMENTATION_URL="https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9"
BUG_REPORT_URL="https://issues.redhat.com/"

REDHAT_BUGZILLA_PRODUCT="Red Hat Enterprise Linux 9"
REDHAT_BUGZILLA_PRODUCT_VERSION=9.6
REDHAT_SUPPORT_PRODUCT="Red Hat Enterprise Linux"
REDHAT_SUPPORT_PRODUCT_VERSION="9.6"

[peter@rhel94 ~]$ 
```
* podman이 정상적으로 실행되는 것을 확인
<br>

### 2.3 rootless 컨테이너의 고려 사항

#### 2.3.1 호스트 컨테이너 스토리지 경로는 root와 non-root가 다름

실행 명령어
```bash
ls -ldZ /var/lib/containers/storage
ls -ldZ /home/peter/.local/share/containers/storage
```

실행 결과
```
[root@rhel94 ~]# ls -ldZ /var/lib/containers/storage
drwxr-xr-x. 8 root root system_u:object_r:container_var_lib_t:s0 4096  3월 22 19:29 /var/lib/containers/storage

[root@rhel94 ~]# ls -ldZ /home/peter/.local/share/containers/storage
drwx------. 9 peter peter unconfined_u:object_r:data_home_t:s0 4096  9월  4 14:43 /home/peter/.local/share/containers/storage

[root@rhel94 ~]# 
```
* 호스트 컨테이너 스토리지 경로
  + root 사용자: /var/lib/containers/storage
  + non-root 사용자: $HOME/.local/share/containers/storage

#### 2.3.2 rootless 컨테이너를 실행하는 사용자 권한

* 호스트 시스템에서 다양한 사용자 및 그룹 ID로 실행할 수 있는 특별 권한이 부여
* 호스트에서 운영 체제에 대한 루트 권한은 없음

#### 2.3.3 /etc/subuid 및 /etc/subgid 수동 변경하는 경우, `podman system migrate` 명령을 실행하여 새 변경 사항을 적용

```
[root@rhel94 ~]# podman system migrate --help
Migrate containers

Description:
        podman system migrate
        Migrate existing containers to a new version of Podman.


Usage:
  podman system migrate [options]

Options:
      --new-runtime string   Specify a new runtime for all containers

[root@rhel94 ~]# 
```

#### 2.3.4 rootless 컨테이너 환경을 구성해야 하는 경우 홈 디렉터리에 구성 파일을 생성

* $HOME/.local/containers
  + *storage.conf*: 스토리지 구성
  + *containers.conf*: 컨테이너 설정
  + *registries.conf*: podman을 사용하여 이미지를 가져오기, 검색, 및 실행할 때 사용할 수 있는 컨테이너 레지스트리
<br>

### 2.4 루트 권한 없는 일반 사용자가 rootless 컨테이너 실행 시 제한 사항

#### 2.4.1 루트 권한 없이는, 변경할 수 없는 시스템 기능이 있음

**예: 시스템 클럭 변경**
* 컨테이너 내에서 SYS_TIME 기능을 설정하고 네트워크 시간 서비스(ntpd)를 실행하여 시스템 클럭 변경 불가

**해결 방안**
* rootless 컨테이너 환경을 무시하고, root 사용자의 환경을 사용하여 컨테이너 실행 필요
  ```
  [root@rhel94 ~]# podman run -d --cap-add SYS_TIME ntpd
  ```
* 이 예제는 ntpd가 컨테이너 내부가 아닌 전체 시스템의 시간을 조정할 수 있음

#### 2.4.2 rootless 컨테이너는 1024 미만의 포트 번호에 액세스할 수 없음

**예: 컨테이너 내 80포트를 사용하는 httpd 서비스**
```
[peter@rhel94 ~]$ podman run -d httpd
```
* 같은 네임스페이스에서는 접속 가능
* 네임스페이스 외부에서는 접속 안됨

**해결 방안**
```
[root@rhel94 ~]# podman run -d -p 80:80 httpd
```
* 호스트 시스템에 해당 포트를 노출하려면 root 사용자의 컨테이너 환경을 사용하여 실행

> [!NOTE]
> 관리자는 일반 사용자가 1024보다 낮은 포트에 서비스를 노출할 수 있도록 구성할 수 있습니다.
> ```
> [root@rhel94 ~]# echo 80 > /proc/sys/net/ipv4/ip_unprivileged_port_start
> ```
> 하지만, 위와 같이 구성하는 경우에는 보안에 위협이 있을 수 있으므로 충분히 인지해야 합니다.
<br>
<br>

## 3. Podman 구성을 위한 모듈 사용

### 3.1 Podman 모듈을 사용하여 사전 정의된 구성 세트를 로드

#### 3.1.1 Podman 모듈은 TOML 형식의 containers.conf 파일

|사용자|모듈 위치|
|:---|:---|
|rootless 사용자의 경우|$HOME/.config/containers/containers.conf.modules|
|root 사용자 경우|/etc/containers/containers.conf.modules<br>/usr/share/containers/containers.conf.modules|

> [!NOTE]
> TOML은 [TOM's Obvious Mimimal Language](https://toml.io/en/)의 약자입니다.

#### 3.1.2 podman 명령어로 모듈 로드

온디맨드 모듈을 로드하여 시스템 및 사용자 구성 파일을 덮어쓰기
```bash
podman --module <your_module_name>
```
* module 옵션을 사용하여 모듈을 여러 번 지정 가능
* <your_module_name>이 절대 경로이면 구성 파일이 직접 로드됨
* 상대 경로는 이전에 언급한 세 개의 모듈 디렉터리에 따라 해결됩니다.
* $HOME의 모듈은 /etc/ 및 /usr/share/ 디렉터리의 모듈을 재정의

실행 명령어
```bash

```

실행 결과
```

```

<br>
<br>

------
[차례](../README.md)
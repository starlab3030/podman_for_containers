# RHEL9 상의 컨테이너

<br>
<br>

## 1. 컨테이너 이미지 유형

### 1.1 컨테이너 이미지

컨테이너 이미지는 단일 컨테이너를 실행하기 위한 모든 요구 사항과 요구 사항 및 기능을 설명하는 메타데이터를 포함하는 바이너리

**두 가지 유형**
* Red Hat Enterprise Linux 기본 이미지 (RHEL 기본 이미지)
* Red Hat Universal Base Images (UBI 이미지)

**이미지 비교**
* 공통점: 두 가지 유형의 컨테이너 이미지는 모두 RHEL의 일부에서 빌드
* 차이점: UBI 이미지에서 컨테이너 이미지를 다른 이미지와 공유 가능
<br>

### 1.2 RHEL 컨테이너 이미지 특성

RHEL 기본 이미지와 UBI 이미지에 모두 적용

|항목|설명|
|:---|:---|
|지원|레드햇에서 컨테이너화된 애플리케이션과 함께 사용할 수 있도록 지원<br>RHEL에 있는 것과 동일한 보안, 테스트 및 인증된 소프트웨어 패키지가 포함|
|카탈로그|Red Hat Container Catalog에 나열된 설명, 기술 세부 정보 및 각 이미지의 상태 색인|
|업데이트|최신 소프트웨어를 얻으려면 잘 정의된 업데이트 일정이 포함된 Red Hat Container Image Updates 문서를 참조|
|추적|레드햇 제품 Errata가 추적하여 각 업데이트에 추가된 변경 사항을 이해하는 데 도움이 됨|
|재사용|컨테이너 이미지를 한 번 다운로드하여 프로덕션 환경에 캐시<br>각 컨테이너 이미지는 기본으로 포함된 모든 컨테이너에서 재사용 가능|
<br>

### 1.3 UBI 이미지 특성

#### 1.3.1 RHEL 콘텐츠의 하위 집합에서 빌드

Red Hat Universal Base 이미지는 일반 Red Hat Enterprise Linux 콘텐츠의 하위 집합에서 빌드 됨

#### 1.3.2 재배포 가능

* UBI 이미지를 사용하면 Red Hat 고객, 파트너, ISV 등을 표준화 가능
* UBI 이미지를 사용하면 자유롭게 공유 및 배포할 수 있는 공식 Red Hat 소프트웨어 기반에 컨테이너 이미지 빌드 가능

#### 1.3.3 네 가지 기본 이미지 세트 제공

|이름|이미지|
|:---|:---|
|마이크로|ubi-micro|
|최소|ubi-minimal|
|표준|ubi|
|init|ubi-init|

#### 1.3.4 미리 빌드된 언어 런타임 컨테이너 이미지 세트를 제공

* Application Streams을 기반으로 하는 런타임 이미지
* python, perl, php, dotnet, nodejs 및 ruby와 같은 표준 지원되는 런타임을 활용할 수 있는 애플리케이션에 대한 기반을 제공

#### 1.3.5 관련 DNF 리포지토리 세트를 제공

DNF 리포지토리에는 애플리케이션 종속 항목을 추가하고 UBI 컨테이너 이미지를 다시 빌드할 수 있는 RPM 패키지 및 업데이트가 포함

|리포지토리 세트|설명|
|:---|:---|
|ubi-9-baseos|컨테이너에 포함할 수 있는 RHEL 패키지의 재배포 가능 하위 세트|
|ubi-9-appstream|UBI 이미지에 추가하여 특정 런타임이 필요한 애플리케이션에서 사용하는 환경을 표준화하는 데 도움이 되는 애플리케이션 스트림 패키지|
|UBI RPM 추가|사전 구성된 UBI 저장소의 UBI 이미지에 RPM 패키지를 추가|

#### 1.3.6 라이센스

레드햇 UBI EULA(End-User License Agreement)를 준수하면 UBI 이미지를 자유롭게 사용하고 재배포 가능
<br>

### 1.4 UBI 표준 (ubi) 이미지

RHEL에서 실행되는 모든 애플리케이션에 대상으로 설계됨

|주요 기능|설명|
|:---|:---|
|init 시스템|systemd 서비스를 관리하는 데 필요한 모든 systemd 초기화 시스템의 기능을 표준 기본 이미지에서 사용 가능<br>init 시스템을 사용하면 웹 서버(httpd) 또는 FTP 서버(ECDHE )와 같이 서비스가 자동으로 시작되도록 사전 구성된 RPM 패키지를 설치 가능|
|dnf|소프트웨어를 추가하고 업데이트하기 위해 사용 가능한 dnf 리포지토리에 액세스<br>표준 dnf 명령어 세트 사용 가능|
|유틸리트|tar,dmidecode,gzip,getfacl 및 추가 acl 명령, dmsetup 및 추가 장치 매퍼 명령을 포함|
<br>

### 1.5 UBI init (ubi-init) 이미지

#### 1.5.1 특징

* systemd 초기화 시스템이 포함되어 있어 웹 서버 또는 파일 서버와 같은 systemd 서비스를 실행할 이미지를 빌드하는 데 유용
* ubi-init 이미지에 포함한 컨텐츠는 ubi 이미지보다 작지만 ubi-minimal 이미지에 있는 것보다 더 많음

#### 1.5.2 ubi-init과 ubi의 차이점

ubi9-init 이미지는 ubi9 이미지 상단에 빌드되므로 해당 콘텐츠는 대부분 동일하며, 다음과 차이점이 있음

|이미지|설명|
|:---|:---|
|ubi9-init|<ul><li>CMD는 기본적으로 systemd Init 서비스를 시작하기 위해 /sbin/init 로 설정</li><li>ps 및 process 관련 명령 포함 (procps-ng package)</li><li>ubi9-init 의 systemd 는 종료(SIGTERM 및 SIGKILL)를 무시하므로 SIGRTMIN+3 을 StopSignal 으로 설정하지만 SIGRTMIN+3을 수신하면 종료</li></ul>|
|ubi9|<ul><li>CMD가 /bin/bash로 설정</li><li>ps 및 process 관련 명령(procps-ng package)을 포함하지 않음</li><li>료하기 위해 정상적인 신호를 무시하지 않음(SIGTERM 및 SIGKILL)</li></ul>|
<br>

### 1.6 UBI 최소 (ubi-minimal) 이미지

#### 1.6.1 특징

* 사전 설치된 최소화된 콘텐츠 세트와 패키지 관리자(microdnf)를 제공
* 이미지에 포함된 종속성을 최소화하면서 Containerfile을 사용

#### 1.6.2 주요 기능

|기능|설명|
|:---|:---|
|작은 크기|<ul><li>최소 이미지는 압축 시 디스크에서 약 92M과 32M</li><li>표준 이미지 크기의 절반 미만</li></ul>|
|소프트웨어 설치(microdnf)|<ul><li>dnf 기능을 포함하는 대신, 최소 이미지에는 microdnf 유틸리티가 포함</li><li>microdnf 는 패키지를 설치한 후 리포지토리를 활성화 및 비활성화하고 패키지를 제거 및 업데이트하고 캐시를 정리할 수 있도록 하는 dnf 의 축소 버전</li></ul>|
|RHEL 패키징 기반|<ul><li>일반 RHEL 소프트웨어 RPM 패키지가 포함되어 있으며 몇 가지 기능이 제거됨</li><li>systemd 또는 System V init, Python 런타임 환경 및 일부 쉘 유틸리티와 같은 초기화 및 서비스 관리 시스템이 포함되지 않음</li><li>이미지 빌드를 위해 RHEL 리포지토리를 사용할 수 있지만 최소한의 오버헤드를 수행</li></ul>|
|microdnf 모듈 지원|<ul><li>microdnf 명령에 사용되는 모듈은 사용 가능한 경우 동일한 소프트웨어의 여러 버전을 설치 가능</li><li>microdnf module enable, microdnf module disable, microdnf module reset을 사용하여 모듈 스트림을 각각 활성화, 비활성화 및 재설정</li></ul>|
<br>

### 1.7 UBI 마이크로 (ubi-micro) 이미지

* 패키지 관리자 및 일반적으로 컨테이너 이미지에 포함된 모든 종속 항목을 제외하고 얻은 최소 UBI 이미지
* ubi-micro 이미지를 기반으로 컨테이너 이미지의 공격 면적을 최소화하고 다른 애플리케이션에 UBI Standard, Minimal 또는 Init을 사용하는 경우에도 최소 애플리케이션에 적합
* 리눅스 배포 패키징이 없는 컨테이너 이미지를 Distroless 컨테이너 이미지라고 함
<br>
<br>

## 2. 

<br>
<br>

## 99. 

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
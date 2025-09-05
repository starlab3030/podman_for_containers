
# 컨테이너 네트워크


<br>
<br>

## 1. 네트워크 관리

### 1.1 컨테이너 네트워크 나열

#### 1.1.1 Podman 네트워킹

|네트워크|설명|
|:---|:---|
|rootless 네트워킹|네트워크가 자동으로 설정되며, 컨테이너에 IP 주소가 없음|
|rootful 네트워킹|컨테이너에 IP 주소가 있음|

#### 1.1.2 root 사용자로 모든 네트워크 나열

실행 명령어
```bash
podman network ls
```

실행 결과
```
[root@rhel94 ~]# podman network ls
NETWORK ID    NAME        DRIVER
2f259bab93aa  podman      bridge

[root@rhel94 ~]#
```
* 기본적으로 Podman은 브리지된 네트워크를 제공
* rootless 사용자의 네트워크 목록은 rootful 사용자의 것과 동일
<br>

### 1.2 네트워크 검사

#### 1.2.1 `podman network inspect` 명령어

나열된 네트워크의 IP 범위, 활성화된 플러그인, 네트워크 유형 등을 표시

#### 1.2.2 기본 *podman* 네트워크를 검사

실행 명령어
```bash
podman network inspect podman | jq '.'
```

실행 결과 - JSON 형식
```json
[
  {
    "name": "podman",
    "id": "2f259bab93aaaaa2542ba43ef33eb990d0999ee1b9924b557b7be53c0b7a1bb9",
    "driver": "bridge",
    "network_interface": "podman0",
    "created": "2025-09-05T07:59:03.592153308+09:00",
    "subnets": [
      {
        "subnet": "10.88.0.0/16",
        "gateway": "10.88.0.1"
      }
    ],
    "ipv6_enabled": false,
    "internal": false,
    "dns_enabled": false,
    "ipam_options": {
      "driver": "host-local"
    },
    "containers": {}
  }
]
```
<br>

### 1.3 네트워크 생성

#### 1.3.1 `podman network create` 명령어

* 기본적으로 외부 네트워크 생성
* podman network create –internal로 내부 네트워크 생성 가능
  + 내부 네트워크의 컨테이너는 호스트의 다른 컨테이너와 통신 가능
  + 호스트 외부의 네트워크에 연결할 수 없음

#### 1.3.2 새로운 외부 네트워크 생성

실행 명령어
```bash
podman network create my-net
podman network ls
```

실행 결과
```
[root@rhel94 ~]# podman network create my-net
my-net

[root@rhel94 ~]# podman network ls
NETWORK ID    NAME        DRIVER
c2007cb3a487  my-net      bridge
2f259bab93aa  podman      bridge

[root@rhel94 ~]#
```

#### 1.3.3 생성된 네트워크 확인

실행 명령어
```bash
podman network inspect my-net | jq '.'
```

실행 결과 - JSON 형식
```json
[
  {
    "name": "my-net",
    "id": "c2007cb3a487f956adf4c42b54a8706102e7e76269d9c38128d90d95bb5666f1",
    "driver": "bridge",
    "network_interface": "podman1",
    "created": "2025-09-05T08:23:14.98592301+09:00",
    "subnets": [
      {
        "subnet": "10.89.0.0/24",
        "gateway": "10.89.0.1"
      }
    ],
    "ipv6_enabled": false,
    "internal": false,
    "dns_enabled": true,
    "ipam_options": {
      "driver": "host-local"
    },
    "containers": {}
  }
]
```

> [!NOTE]
> Podman 4.0부터 podman network create 명령을 사용하여 새 외부 네트워크를 생성하는 경우 기본적으로 DNS 플러그인이 활성화됩니다.

#### 1.3.4 생성한 네트워크 구성 파일 확인

실행 명령어
```bash
ls -lh /etc/containers/networks/my-net.json
cat /etc/containers/networks/my-net.json
```

실행 결과
```
[root@rhel94 ~]# podman info | yq -r '.host|.networkBackend'
netavark

[root@rhel94 ~]# ls -lh /etc/containers/networks/my-net.json
-rw-r--r--. 1 root root 492  9월  5 08:23 /etc/containers/networks/my-net.json

[root@rhel94 ~]#
```
* Podman 네트워크 백엔드가 *netavark*
  + 기본 구성 파일 위치: /etc/containers/network/*.json

/etc/containers/networks/my-net.json 파일
```json
{
     "name": "my-net",
     "id": "c2007cb3a487f956adf4c42b54a8706102e7e76269d9c38128d90d95bb5666f1",
     "driver": "bridge",
     "network_interface": "podman1",
     "created": "2025-09-05T08:23:14.98592301+09:00",
     "subnets": [
          {
               "subnet": "10.89.0.0/24",
               "gateway": "10.89.0.1"
          }
     ],
     "ipv6_enabled": false,
     "internal": false,
     "dns_enabled": true,
     "ipam_options": {
          "driver": "host-local"
     }
}
```
<br>

### 1.4 네트워크에 컨테이너 연결

#### 1.4.1 `podman network connect` 명령어

네트워크에 지정한 컨테이너를 연결

#### 1.4.2 신규 컨테이너 생성 후 네트워크 연결

실행 명령어
```bash
podman run -dt --name my-rhel94 registry.access.redhat.com/ubi9/ubi:9.4 /bin/bash
podman ps
podman network connect my-net my-rhel94
```

실행 결과
```
[root@rhel94 ~]# podman run -dt --name my-rhel94 registry.access.redhat.com/ubi9/ubi:9.4 /bin/bash
c74bb9b7157fd2f8b4c117a19cd4bae4ae3c0868ada7a94719ed1549b304c0e9

[root@rhel94 ~]# podman ps
CONTAINER ID  IMAGE                                    COMMAND     CREATED        STATUS        PORTS       NAMES
c74bb9b7157f  registry.access.redhat.com/ubi9/ubi:9.4  /bin/bash   5 seconds ago  Up 6 seconds              my-rhel94

[root@rhel94 ~]# podman network connect my-net my-rhel94

[root@rhel94 ~]#
```

#### 1.4.3 컨테이너 *my-rhel94*에 *my-net* 네트워크 연결 확인

실행 명령어
```bash
podman inspect my-rhel94 | jq '.[].NetworkSettings.Networks'
```

실행 결과 - JSON 형식
```json
{
  "my-net": {
    "EndpointID": "",
    "Gateway": "10.89.0.1",
    "IPAddress": "10.89.0.2",
    "IPPrefixLen": 24,
    "IPv6Gateway": "",
    "GlobalIPv6Address": "",
    "GlobalIPv6PrefixLen": 0,
    "MacAddress": "2a:8a:10:7c:8b:74",
    "NetworkID": "c2007cb3a487f956adf4c42b54a8706102e7e76269d9c38128d90d95bb5666f1",
    "DriverOpts": null,
    "IPAMConfig": null,
    "Links": null,
    "Aliases": [
      "c74bb9b7157f"
    ]
  },
  "podman": {
    "EndpointID": "",
    "Gateway": "10.88.0.1",
    "IPAddress": "10.88.0.2",
    "IPPrefixLen": 16,
    "IPv6Gateway": "",
    "GlobalIPv6Address": "",
    "GlobalIPv6PrefixLen": 0,
    "MacAddress": "ce:ac:2d:5d:a3:c9",
    "NetworkID": "2f259bab93aaaaa2542ba43ef33eb990d0999ee1b9924b557b7be53c0b7a1bb9",
    "DriverOpts": null,
    "IPAMConfig": null,
    "Links": null,
    "Aliases": [
      "c74bb9b7157f"
    ]
  }
}
```
* 두 개의 네트워크가 연결되어 있음을 확인
  + *my-net*
  + *podman*
* *my-net*
  + IP: 10.89.0.2
* *podman*
  + IP: 10.88.0.2

#### 1.4.4 Podman 네트워크 *my-net*에서 할당된 IP 확인

실행 명령어
```bash
podman network inspect my-net | jq '.[]'
```

실행 결과 - JSON 형식
```json
{
  "name": "my-net",
  "id": "c2007cb3a487f956adf4c42b54a8706102e7e76269d9c38128d90d95bb5666f1",
  "driver": "bridge",
  "network_interface": "podman1",
  "created": "2025-09-05T08:23:14.98592301+09:00",
  "subnets": [
    {
      "subnet": "10.89.0.0/24",
      "gateway": "10.89.0.1"
    }
  ],
  "ipv6_enabled": false,
  "internal": false,
  "dns_enabled": true,
  "ipam_options": {
    "driver": "host-local"
  },
  "containers": {
    "c74bb9b7157fd2f8b4c117a19cd4bae4ae3c0868ada7a94719ed1549b304c0e9": {
      "name": "my-rhel94",
      "interfaces": {
        "eth1": {
          "subnets": [
            {
              "ipnet": "10.89.0.2/24",
              "gateway": "10.89.0.1"
            }
          ],
          "mac_address": "2a:8a:10:7c:8b:74"
        }
      }
    }
  }
}
```
* 컨테이너 *my-rhel94*에 인터페이스 eth1에 IP 10.89.0.2가 할당된 것을 확인
<br>

### 1.5 네트워크에서 컨테이너 연결 해제

#### 1.5.1 `podman network disconnect` 명령어

네트워크에서 지정한 컨테이너 연결 해제

#### 1.5.2 컨테이너 *my-rhel94*에서 *my-net* 네트워크의 연결을 끊음

실행 명령어
```bash
podman network disconnect my-net my-rhel94
```

실행 결과
```
[root@rhel94 ~]# podman network disconnect my-net my-rhel94

[root@rhel94 ~]#
```

#### 1.5.3 컨테이너 *my-rhel94*의 네트워크 연결 확인

실행 명령어
```bash
podman inspect my-rhel94 | jq '.[].NetworkSettings.Networks'
```

실행 결과 - JSON 형식
```json
{
  "podman": {
    "EndpointID": "",
    "Gateway": "10.88.0.1",
    "IPAddress": "10.88.0.2",
    "IPPrefixLen": 16,
    "IPv6Gateway": "",
    "GlobalIPv6Address": "",
    "GlobalIPv6PrefixLen": 0,
    "MacAddress": "ce:ac:2d:5d:a3:c9",
    "NetworkID": "2f259bab93aaaaa2542ba43ef33eb990d0999ee1b9924b557b7be53c0b7a1bb9",
    "DriverOpts": null,
    "IPAMConfig": null,
    "Links": null,
    "Aliases": [
      "c74bb9b7157f"
    ]
  }
}
```
* 컨테이너에는 기본 *podman* 네트워크가 연결되어 있음

#### 1.5.4 Podman 네트워크 *my-net*에서 할당된 IP 확인

실행 명령어
```bash
podman network inspect my-net | jq '.[]'
```

실행 결과 - JSON 형식
```json
{
  "name": "my-net",
  "id": "c2007cb3a487f956adf4c42b54a8706102e7e76269d9c38128d90d95bb5666f1",
  "driver": "bridge",
  "network_interface": "podman1",
  "created": "2025-09-05T08:23:14.98592301+09:00",
  "subnets": [
    {
      "subnet": "10.89.0.0/24",
      "gateway": "10.89.0.1"
    }
  ],
  "ipv6_enabled": false,
  "internal": false,
  "dns_enabled": true,
  "ipam_options": {
    "driver": "host-local"
  },
  "containers": {}
}
```
* 네트워크 *my-net*에는 컨테이너 할당된 IP가 없음
<br>

### 1.6 네트워크 제거

#### 1.6.1 `podman network rm` 명령어

지정된 네트워크를 제거

#### 1.6.2 네트워크 *my-net*을 제거

실행 명령어
```bash
podman network ls
podman network rm my-net
podman network ls
```

실행 결과
```
[root@rhel94 ~]# podman network ls
NETWORK ID    NAME        DRIVER
c2007cb3a487  my-net      bridge
2f259bab93aa  podman      bridge

[root@rhel94 ~]# podman network rm my-net
my-net

[root@rhel94 ~]# podman network ls
NETWORK ID    NAME        DRIVER
2f259bab93aa  podman      bridge

[root@rhel94 ~]#
```
<br>

### 1.7 사용되지 않는 모든 네트워크 제거

#### 1.7.1 `podman network prune` 명령어

* 사용되지 않는 모든 네트워크를 제거
  + 미사용 네트워크는 컨테이너가 연결되어 있지 않은 네트워크
* 해당 명령은 기본 네트워크인 *podman*은 제거하지 않음

#### 1.7.2 테스트를 위한 네트워크 생성

실행 명령어
```bash
podman network create my-net1
podman network create my-net2
podman network ls
podman network inspect  my-net1 my-net2|jq '.[]|{"name": .name, "containers": .containers}'
```

실행 결과
```
[root@rhel94 ~]# podman network create my-net1
my-net1
[root@rhel94 ~]# podman network create my-net2
my-net2

[root@rhel94 ~]# podman network ls
NETWORK ID    NAME        DRIVER
c949b2280001  my-net1     bridge
91d080f8565d  my-net2     bridge
2f259bab93aa  podman      bridge

[root@rhel94 ~]# podman network inspect  my-net1 my-net2|jq '.[]|{"name": .name, "containers": .containers}'
{
  "name": "my-net1",
  "containers": {}
}
{
  "name": "my-net2",
  "containers": {}
}

[root@rhel94 ~]#
```
* 생성된 두 개의 네트워크는 컨테이너에 할당한 IP가 없음

#### 1.7.3 네트워크 정리

실행 명령어
```bash
podman network prune
podman network ls
```

실행 결과
```
[root@rhel94 ~]# podman network prune
WARNING! This will remove all networks not used by at least one container.
Are you sure you want to continue? [y/N] y
my-net1
my-net2

[root@rhel94 ~]# podman network ls
NETWORK ID    NAME        DRIVER
2f259bab93aa  podman      bridge

[root@rhel94 ~]#
```
<br>
<br>

## 2. 컨테이너 간 통신

### 2.1 네트워크 모드 및 계층

|항목|설명|
|:---|:---|
|bridge|기본 브릿지 네트워크에 다른 네트워크 생성|
|container:<id>|<id>와 같은 ID가 있는 컨테이너와 동일한 네트워크를 사용|
|host|호스트 네트워크 스택 사용|
|network-id|`podman network create` 명령으로 생성된 사용자 정의 네트워크를 사용|
|private|컨테이너에 대한 새 네트워크 생성|
|slirp4nets|rootless 컨테이너의 기본 옵션인 slirp4netns를 사용하여 사용자 네트워크 스택을 만듦|
|pasta|slirp4netns를 대체하는 고성능 모드<br>Podman v4.4.1부터 사용 가능|
|none|컨테이너의 네트워크 네임스페이스를 생성<br>네트워크 인터페이스를 구성하지 않으며, 컨테이너에 네트워크 연결이 없음|
|NS:<path>|결합할 네트워크 네임스페이스의 경로|

> [!NOTE]
> **host** 모드는 컨테이너에 D-bus와 같은 로컬 시스템 서비스(예: 프로세스 간 통신)에 대한 전체 액세스 권한을 부여하므로 비보안으로 간주됩니다.
<br>

### 2.2 slirp4netns와 pasta의 차이점

**pasta의 장점**
* IPv6 포트 포워딩 지원
* slirp4netns보다 효율적
  + slirp4netns는 사전 정의된 IPv4 주소를 사용 
* 호스트의 인터페이스 이름을 사용
  + slirp4netns는 인터페이스 이름으로 tap0을 사용
* 호스트의 게이트웨이 주소를 사용
  + 자체 게이트웨이 주소를 정의하고 NAT를 사

> [!NOTE]
> rootless 컨테이너의 기본 네트워크 모드는 slirp4netns 입니다.
<br>

### 2.3 네트워크 모드 설정

#### 2.3.1 사용자 *peter*로 로그인

실행 명령어
```bash
ssh peter@localhost
```

실행 결과
```
[root@rhel94 ~]# ssh peter@localhost
peter@localhost's password: ******
Register this system with Red Hat Insights: insights-client --register
Create an account or view all your systems at https://red.ht/insights-dashboard
Last login: Thu Sep  4 14:35:25 2025 from ::1

[peter@rhel94 ~]$
```

#### 2.3.2 컨테이너 *peter-rhel94* 생성

실행 명령어
```bash
podman run -dt --name peter-rhel94 registry.access.redhat.com/ubi9/ubi:9.4 /bin/bash
```

실행 결과
```
[peter@rhel94 ~]$ podman run -dt --name peter-rhel94 --network podman registry.access.redhat.com/ubi9/ubi:9.4 /bin/bash
Trying to pull registry.access.redhat.com/ubi9/ubi:9.4...
Getting image source signatures
Checking if image destination supports signatures
Copying blob a20d5f0bec2b done   |
Copying config 769453be74 done   |
Writing manifest to image destination
Storing signatures
bbb07b7a461919e22657d2793c4fd233857872953dd257e4f316491466461570

[peter@rhel94 ~]$
```

#### 2.3.3 컨테이너 *peter-rhel94*의 네트워크 연결 확인

실행 명령어
```bash
podman inspect peter-rhel94 | jq -r '.[].NetworkSettings.Networks'
```

실행 결과
```
[peter@rhel94 ~]$ podman inspect peter-rhel94 | jq -r '.[].NetworkSettings.Networks'
null

[peter@rhel94 ~]$
```

#### 2.3.4 컨테이너 *peter-rhel94*의 네트워크 모드 확인

실행 명령어
```bash
podman inspect peter-rhel94 | jq -r '.[].HostConfig.NetworkMode'
```

실행 결과
```
[peter@rhel94 ~]$ podman inspect peter-rhel94 | jq -r '.[].HostConfig.NetworkMode'
pasta

[peter@rhel94 ~]$
```
* 네트워크 모드가 *pasta*

#### 2.3.5 컨테이너 *peter-rhel92-v2*를 네트워크 *podman*에 연결하여 생성

실행 명령어
```bash
podman run -dt --name peter-rhel94-v2 --network podman registry.access.redhat.com/ubi9/ubi:9.4 /bin/bash
```

실행 결과
```
[peter@rhel94 ~]$ podman run -dt --name peter-rhel94-v2 --network podman registry.access.redhat.com/ubi9/ubi:9.4 /bin/bash
89fff60690f892baa036ee0d29f294d83127dc0a6197b67c26732165f6272933

[peter@rhel94 ~]$
```

#### 2.3.6 컨테이너 *peter-rhel94-v2*의 네트워크 연결 확인

실행 명령어
```bash
podman inspect peter-rhel94 | jq -r '.[].NetworkSettings.Networks'
```

실행 결과
```
[peter@rhel94 ~]$ podman inspect peter-rhel94-v2 | jq -r '.[].NetworkSettings.Networks'
{
  "podman": {
    "EndpointID": "",
    "Gateway": "10.88.0.1",
    "IPAddress": "10.88.0.3",
    "IPPrefixLen": 16,
    "IPv6Gateway": "",
    "GlobalIPv6Address": "",
    "GlobalIPv6PrefixLen": 0,
    "MacAddress": "2a:e3:b0:61:0f:00",
    "NetworkID": "2f259bab93aaaaa2542ba43ef33eb990d0999ee1b9924b557b7be53c0b7a1bb9",
    "DriverOpts": null,
    "IPAMConfig": null,
    "Links": null,
    "Aliases": [
      "89fff60690f8"
    ]
  }
}

[peter@rhel94 ~]$
```

#### 2.3.7 컨테이너 *peter-rhel94-v2*의 네트워크 모드 확인

실행 명령어
```bash
podman inspect peter-rhel94 | jq -r '.[].HostConfig.NetworkMode'
```

실행 결과
```
[peter@rhel94 ~]$ podman inspect peter-rhel94-v2 | jq -r '.[].HostConfig.NetworkMode'
bridge

[peter@rhel94 ~]$
```
* 네트워크 모드가 *bridge*

#### 2.3.8 네트워크 *peter-net* 생성

실행 명령어
```bash
podman network create peter-net
podman network ls
```

실행 결과
```
[peter@rhel94 ~]$ podman network create peter-net
peter-net

[peter@rhel94 ~]$ podman network ls
NETWORK ID    NAME        DRIVER
fd526b54f8af  peter-net   bridge
2f259bab93aa  podman      bridge

[peter@rhel94 ~]$
```

#### 2.3.9 컨테이너 *peter-rhel92-v3*를 네트워크 *peter-net*에 연결하여 생성

실행 명령어
```bash
podman run -dt --name peter-rhel94-v3 --network peter-net registry.access.redhat.com/ubi9/ubi:9.4 /bin/bash
```

실행 결과
```
[peter@rhel94 ~]$ podman run -dt --name peter-rhel94-v3 --network peter-net registry.access.redhat.com/ubi9/ubi:9.4 /bin/bash
5e551699a1094fef945ac4bafba7e408616f05864744d39f950a7f58cdf10127

[peter@rhel94 ~]$
```

#### 2.3.10 컨테이너 *peter-rhel94-v3*의 네트워크 설정 및 모드 확인

실행 명령어
```bash
podman inspect peter-rhel94-v3 | jq -r '.[].NetworkSettings.Networks'
podman inspect peter-rhel94-v3 | jq -r '.[].HostConfig.NetworkMode'
```

실행 결과
```
[peter@rhel94 ~]$ podman inspect peter-rhel94-v3 | jq -r '.[].NetworkSettings.Networks'
{
  "peter-net": {
    "EndpointID": "",
    "Gateway": "10.89.0.1",
    "IPAddress": "10.89.0.2",
    "IPPrefixLen": 24,
    "IPv6Gateway": "",
    "GlobalIPv6Address": "",
    "GlobalIPv6PrefixLen": 0,
    "MacAddress": "de:fc:07:00:ee:22",
    "NetworkID": "fd526b54f8af1c1077bc70ce48eb8730365372a08a6e33ca2e95084adb8e4b53",
    "DriverOpts": null,
    "IPAMConfig": null,
    "Links": null,
    "Aliases": [
      "5e551699a109"
    ]
  }
}

[peter@rhel94 ~]$ podman inspect peter-rhel94-v3 | jq -r '.[].HostConfig.NetworkMode'
bridge

[peter@rhel94 ~]$
```

#### 2.3.11 컨테이너 *peter-rhel92-v4*를 네트워크 *slirp4netns* 모드로 생성

실행 명령어
```bash
podman run -dt --name peter-rhel94-v4 --network slirp4netns registry.access.redhat.com/ubi9/ubi:9.4 /bin/bash
```

실행 결과
```
[peter@rhel94 ~]$ podman run -dt --name peter-rhel94-v4 --network slirp4netns registry.access.redhat.com/ubi9/ubi:9.4 /bin/bash
9f7fe3b1a4773344f4180ddb44dff6b45f36310967ec0d59b486de79bacc1693

[peter@rhel94 ~]$
```

#### 2.3.12 컨테이너 *peter-rhel92-v4*를 네트워크 설정 및 모드 확인

실행 명령어
```bash
podman inspect peter-rhel94-v4 | jq -r '.[].NetworkSettings.Networks'
podman inspect peter-rhel94-v4 | jq -r '.[].HostConfig.NetworkMode'
```

실행 결과
```
[peter@rhel94 ~]$ podman inspect peter-rhel94-v4 | jq -r '.[].NetworkSettings.Networks'
null

[peter@rhel94 ~]$ podman inspect peter-rhel94-v4 | jq -r '.[].HostConfig.NetworkMode'
slirp4netns

[peter@rhel94 ~]$
```
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
# [Infra] Linux System CLI, VM NAT, Ubuntu Packaging, Swap Memory, 리눅스 파일 시스템 구조

### Virtual Box VM 네트워크

![image](https://github.com/user-attachments/assets/40bc6ac3-df8c-4c64-940a-5cd61f688b42)

- 공용 nat 생성
    - VM에 같은 네트워크 대역을 공유할 수 있게 만듬

![image](https://github.com/user-attachments/assets/dab8ce9f-6ad7-4432-9c0a-6a11844a6a14)

- VM 설정에 들어가서 네트워크에 들어간 후 NAT 네트워크 선택
    - 내가 생성한 NAT 선택

### Linux OS 확인 명령어

```
# 1. 시스템의 전체적인 정보, 커널 버전, 운영 체제 종류, 호스트 이름, 하드웨어 아키텍처 등을 표시

$uname -a

# 2. LSB(리눅스 표준 기반) 정보를 표시
# 운영 체제의 이름, 버전, 설명 및 기타 정보를 제공

$lsb_release -a

# 참고 : oracle docker 컨테이너 설치
# 수업시간에 설치했던 linux의 버전 확인 후 docker container 입성
# 명령어로 oracle db가 설치된 linux의 os정보 확인

# 3. 운영 체제 정보를 확인할 수 있음
#  많은 리눅스 배포판에서 사용되며, 운영 체제의 이름, 버전, ID 등의 정보를 포함

$cat /etc/os-release

# 4.현재 운영 체제의 버전 및 배포판 정보를 제공하는 텍스트 파일, 운영 체제 정보를 확인할 수 있음

$cat /etc/issue

# Linux 운영체제의 CPU 정보와 코어 개수 확인

$cat /proc/cpuinfo
```

### Oracle Virtual Machine NAT 설정

**NAT란?**

1. **네트워크 주소 변환(network address translation)**
2. 컴퓨터 네트워킹에서 쓰이는 용어로서, IP 패킷의 TCP/UDP 포트 숫자와 소스 및 목적지의 IP 주소 등을 재기록하면서 라우터를 통해 네트워크 트래픽을 주고 받는 기술
3. 1개의 실제 공인 IP 주소에, 다량의 가상 사설 IP 주소를 할당 및 매핑하는, 1:1 또는 1:多 주소 변환(Address Translation) 방식

<br>

![image](https://github.com/user-attachments/assets/044d2082-f7a8-44ef-a279-09f285096cca)

 
![image](https://github.com/user-attachments/assets/b8bc15d0-01c8-427a-bf6f-77e87006517b)

![image](https://github.com/user-attachments/assets/cd9c3f95-9777-45ee-841b-31bd123f6eba)

<br>

1. 네트워크 설정
    - **포트 포워딩**이란?
        - 127.0.0.1:22 주소로 요청시 10.0.2.4의 22번 포트로 연결

2. **VirtualBox 네트워크 종류**
    - 가상 머신이 사용 가능한 다양한 네트워크 환경 지원

| **네트워크 방식** | **설명** | **특징** |
| --- | --- | --- |
| NAT | 네트워크 주소를 변환하여 내외부를 연결 | 외부 통신 가능, 머신 간 통신 불가 |
| **NAT 네트워크** | **내외부 연결 및 내부끼리 통신 할 수 있음** | **외부 통신 가능, 머신 간 통신 가능** |
| 브리치 어댑터 | 호스트 pc와 동등한 수준의 네트워크 구성 | 외부 통신 가능, 머신 간 통신 가능 |
| 내부 네트워크 | 호스트 PC와 독립되어 있음 | 외부 통신 불가, 머신 간 통신 가능 |
| 호스트 전용 어댑터 | 호스트 PC 포함하고외부 접속 불가능 | 외부 통신 불가, 머신 간 통신 가능 |

### 우분투 패키지

1. **패키지(모듈)** 이란?
    1. **linux 시스템을 관리하기 위해 필요한 파일들을 다운로드 및 설치하는 프로그램**
    2. 소프트웨어의 모듈화된 형태로, 특정 프로그램 또는 기능을 설치, 업데이트, 관리
    3.  각 패키지에는 **실행 파일, 라이브러리, 설정 파일, 설명 문서 등이 포함되어 있으며, 이를 통해 소프트웨어를 쉽게 배포하고 관리할 수 있음**
    
2. **Ubuntu 패키지의 주요 개념**
    1. **패키지 관리자**
        1. 패키지를 설치, 제거, 업데이트하는 도구로 소프트웨어를 효율적으로 설치하고 관리할 수 있게 해 주는 기본 단위
        2.  **APT**(Advanced Package Tool)를 사용
            - apt-get or apt 명령어로 패키지를 관리
            
    2. **패키지 형식**
        1. Ubuntu는 **DEB** 형식의 패키지를 사용
            - Debian 기반 시스템에서 사용하는 패키지 형식으로, Ubuntu도 Debian을 기반으로 하기 때문에 이를 사용함
3. **의존성 관리**
    1. 패키지 설치 시, 해당 소프트웨어가 실행되기 위해 필요한 라이브러리나 다른 패키지를 자동으로 설치해 줌
    2. APT는 의존성을 해결하는 데 매우 효율적
        - 예시 : 특정 프로그램이 특정 라이브러리를 필요로 한다면, APT가 자동으로 해당 라이브러리도 함께 설치
        
4. **패키지 저장소**
    1. Ubuntu는 중앙화된 소프트웨어 저장소에서 패키지를 다운로드하고 설치함
        - **기본 저장소**: Canonical이 공식적으로 제공하는 패키지.
        - **PPA(Personal Package Archive)**: 사용자가 직접 제공하는 비공식 저장소.
    2. 우분투 패키지 파일 저장소(repository)
        1. [http://kr.archive.ubuntu.com/ubuntu/pool/](http://kr.archive.ubuntu.com/ubuntu/pool/)

### 패키지 설치 및 관리 명령어

**명령어 : apt-get(Advanced Packaging Tool)**

1. **의존성도 해결해주는 패키지 설치 도구**
2. 우분투를 포함한 데비안 계열 리눅스에서 사용되는 패키지 관리 tool
3. 우분투가 제공하는 파일 저장소에서 자동으로 파일을 다운로드 하여 설치
    1. **/etc/apt/sources.list** 파일에 저장되어 있음
    
    ```bash
    **$ vi /etc/apt/sources.list**
    ```
    
![image](https://github.com/user-attachments/assets/0a35a8f1-3e1a-4160-94c8-ceeab22f633e)


- 설치 명령어
    
    > **sudo apt install <패키지명>
    sudo apt-get install <패키지명>**
    > 

- 업데이트 명령어
    
    > **패키지 목록 갱신
    sudo apt update  
    
    설치된 패키지 업그레이드
    sudo apt upgrade**
    > 

- 제거 명령어
    
    > **sudo apt remove <패키지명>**
    > 
    
- 정보 확인
    
    > **apt show <패키지명>**
    > 

1. **apt vs apt-get** 명령어  차이
    - 특징 - 큰 차이는 없음
        
        
        > **apt**
        
        - 더 나은 대화식 사용을 위한 고급 명령 줄 인터페이스
        > 
        
        > **apt-get**
        
        - 인증된 소스에서 package에 대한 정보를 검색, 종속성과 함께 package를 설치, 업그레이드 및 제거
        > 
        
    - apt-get 옵션 명령어가 apt 명령어 보다 복잡
        - apt 명령어 사용 권장
        - 옵션들이 많아져서 자주 사용되는 기능을 apt로 제공하는 상황

| **apt 명령어** | **apt-get 명령어** | **기능** |
| --- | --- | --- |
| **apt update** | apt-get update | Refreshes repository index |
| **apt install [package]** | apt-get install [package] | Install a package |
| **apt upgrade** | apt-get upgrade | Upgrade available package updates |
| **apt remove [package]** | apt-get remove[package] | Remove a package |
| apt purge [package] | apt-get purge [package] | Remove a package with configuration |
| apt autoremove | apt-get autoremove | Remove unnecessary dependencies |
| apt full-upgrade | apt-get dist-upgrade | Update all packages and remove unnecessary dependencies |
| apt search [package] | apt-cache search [package] | Search for a package |
| **apt show [package]** | apt-cache show [package] | Show package details |
| apt policy | apt-cache policy | Show active repo information |
| apt policy [package | apt-cache policy [package] | Show installed and available package version |




 **update vs upgrade**

> **apt-get update**

OS에서 사용 가능한 package들과 버전에 대한 정보를 업데이트 하는 명령어
> 

> **apt-get upgrade**

OS에 apt-get install로 설치한 package들을 최신 상태로 upgrade 하는 명령어
> 

<aside>
📌 명령어 적용 순서

apt-get update로 업데이트한 package들의 최신 버전에 맞게 upgrade해야 함
$apt-get update && apt-get upgrade -y

</aside>

1. 기본 사용법
    - 일반 사용자인 경우 **sudo** 로 시작

```sql
0. 설치된 package list 확인하기
$apt list --installed

1. /etc/apt/sources.list 파일의 내용 업데이트
$apt-get update

2. 설치
**$apt-get install 패키지명

3. 설치되어 있는 패키지 삭제
$apt-get remove 패키지명

4. 설치되어 있는 패키지와 설정 파일 모두 삭제
$apt-get purge 패키지명

5. 사용하지 않는 패키지 모두 삭제
$apt-get autoremove

6. 설치할 때 다운로드한 파일과 과거의 파일 삭제
$apt-get clean 또는 apt-get autoclean

7. 설치전 패키지 정보와 의존성 문제 확인[패키지 검색]
$apt-cache show 패키지명**

```

### 각각의 게스트 OS 접속하기

1. 게스트 OS 고정 아이피 설정하기

고정 IP설정하기 

- netplan 설정 파일 확인 & 편집
- `00-installer-config.yaml` 또는 `01-netcfg.yaml` 같은 파일 수정
- **설정 파일 편집**
    - 설정 파일의 형식은 YAML
    - DHCP를 사용하지 않고 고정 IP(Static IP)를 설정하려면, 다음과 같이 수정해야 함
    - DHCP 기능? 유동 IP 할당 기능

```sql
**sudo nano /etc/netplan/00-installer-config.yaml

또는

sudo cat > /etc/netplan/00-installer-config.yaml**
```

- 편집 내용
    
    ```yaml
    network:
      version: 2
      renderer: networkd
      ethernets:
        enp0s3:
          addresses:
            - 10.0.2.8/24  # 변경된 고정 IP 주소
          routes:
            - to: default
              via: 10.0.2.1  # 게이트웨이
          nameservers:
            addresses:
              - 8.8.8.8
          dhcp4: false
    ```
    

- 설정 후 적용:

```sql
sudo netplan apply
```

- IP 주소가 제대로 적용되었는지 확인하려면 다음 명령어로 네트워크 상태를 확인합니다.

```sql
ip a  또는 ip addr
```

- NAT 설정 적용


<br>


![image](https://github.com/user-attachments/assets/31858841-673a-4cb6-b9ad-2ea3562f37e0)

### 각 VM Swap 메모리 비활성화

01. Swap 메모리란?

물리적인 RAM이 부족할 때 사용되는 디스크 공간

 Swap은 물리 메모리(RAM)가 다 소모되었을 때, 덜 자주 사용되는 데이터를 디스크의 Swap 영역으로 옮김으로써 시스템이 계속 동작할 수 있도록 함

### |01. Swap의 주요 기능

1. **메모리 확장**
    - RAM이 부족할 때 임시로 디스크 공간을 메모리처럼 사용하여 시스템이 메모리 부족으로 멈추는 것을 방지함
2. **비활성 페이지 저장**
    - 자주 사용되지 않는 메모리 페이지를 Swap으로 이동시켜, 자주 사용되는 데이터를 위한 RAM 공간을 확보하기도 함
    
    03. Swap의 종류
    
    - **Swap 파티션**: 디스크에 할당된 고정된 크기의 파티션.
    - **Swap 파일**: 디스크에 존재하는 파일로, 필요에 따라 동적으로 크기를 조정 가능.
    
    ### |02 Swap 사용의 한계
    
    - 디스크는 RAM보다 많이 느리므로, Swap이 많이 사용되면 시스템 성능이 크게 저하될 수 있음
    - **Swappiness** 설정을 통해 Swap 사용 빈도를 조정할 수 있음
        - 기본값은 60으로, 이 값을 줄이면 Swap 사용을 줄이고 RAM을 더 많이 활용하도록 시스템을 설정할 수 있음
        
    
    ### |03. Swap의 최적화
    
    - **서버**나 **데스크탑** 시스템에서는 물리 메모리를 충분히 확보하는 것이 가장 중요
    - SSD를 사용하는 경우, Swap을 너무 많이 사용하면 디스크 수명이 줄어들 수 있으므로 주의가 필요

### 02. Swap 메모리 비활성화(매우 중요)

- 컨테이너 기반의 서버를 원활하게 관리하기 위해서는 swap 메모리 영역을 비활성화 해야 함

```sql
# 메모리 공간 확인
free -h

# swap 활성화 여부 확인 - 0이 아닌 경우 비활성화해야 함
cat /proc/swaps

# swap 메모리 비활성화 및 확인
sudo -i
**swapoff --all**
cat /proc/swaps
```

- 재부팅시 swap 메모리 자동 활성화
    - 재부팅시에도 재활성화 무효화 설정 방법
    - fstab의 swap.img 라인 주석 → 재부팅

```sql
# fstab 파일 오픈
vim /etc/fstab

# 해당 라인 주석처리
**#/swap.img      none    swap    sw      0       0**

```

### 시간변경

- timedatectl 명령어로 확인

> $ **timedatectl list-timezones**

$ **timedatectl list-timezones | grep Seoul**
Asia/Seoul
> 

> **$ sudo timedatectl set-timezone Asia/Seoul
$ timedatectl**
> 

### 리눅스 구조

- **리눅스 파일 및 디렉토리 구조**
    - 역트리 구조
    - root 디렉토리 : /. 파일 시스템 계층의 시작점

| **디렉토리명** | **의미** | **용도** |
| --- | --- | --- |
| / | 최상의 디렉토리 | 시스템 운영 |
| /bin | 바이너리 및 기타 실행 프로그램을 찾을 수 있는 곳<br>**기본적인 명령어**가 저장되어 있음, mv, cp, rm 등이 존재<br>명령어 디렉토리 | 시스템 운영 |
| /sbin | **시스템 운영 및 관리에 관한 명령어들 보유 디렉토리**<br>계정관리, 디스크 관리 등 관리하는 명령어들 보유<br>대부분 root 사용자 전용 명령어<br>**권한을 부여 받은 일반 user도 sudo 명령어로 사용 가능** | 시스템 운영 |
| /usr | 일반 사용자들이 사용 가능한 명령어 파일들이 존재<br>파일 시스템의 주요 구성, 사용자가 실행할 프로그램들이 저장, read-only 데이터만 존재<br>/usr/bin과 동일 | 시스템 운영 |
| /etc | 시스템 설정 파일 보유<br>사용자 user들의 정보(passwd), hostname, web server config(httpd.conf) 등을 Linux의 text 파일인 ASCII text 파일 형식으로 구성(내용을 알아볼 수 있는 text 구조)<br><br>**ubuntu@ubuntu:/etc$ cat hostname** | 시스템 운영 |
| /var | 가변 데이터 보유 영역, 로그 파일 등<br>**많은 애플리케이션 데이터들 저장 폴더**<br>용량 부족 시 부팅이 안 될 수도 있음 | 시스템 운영 |
| /tmp | 일반적으로 재부팅 시 지워지는 임시 파일들이 저장된 공간<br>시스템과 애플리케이션은 임시 데이터를 여기에 저장<br>모든 사용자에게 읽기/쓰기 권한 | 시스템 운영 |
| /proc | 메모리에서 동작 중인 프로세스 정보를 확인<br><br>**ubuntu@ubuntu:/var$ cd /proc**<br>**ubuntu@ubuntu:/proc$ ls** | 리눅스의 메모리 정보, HW 정보 |
| /sys | 시스템 HW 정보나 가상 파일 시스템들 | 리눅스의 메모리 정보, HW 정보 |
| /root | 최상위 사용자, root의 home 디렉토리 |  |
| /home | **일반 사용자 홈 디렉토리**<br>useradd 명령어로 새로운 사용자를 생성하면 대부분 사용자의 ID와 동일한 이름의 디렉토리가 자동으로 생성됨 |  |
| /dev | 하드웨어 장치 파일(디바이스 디렉토리)<br>리눅스는 모든 것을 파일이라는 포인터 파일로 관리<br>현 리눅스 시스템이 인식한 파일들 목록(디바이스 파일)<br>예: ubuntu 이미지 보유 |  |
| /opt | 독립적인 소프트웨어 설치, 시스템의 다른 부분과 격리<br>서드파티 애플리케이션 설치<br>예: /opt/tomcat, /opt/eclipse |  |

04. 환경 변수  설정과 관련된 파일들

- **BASH 쉘 환경설정 파일을 읽어 들이는 순서**
    
    > **/etc/profile**  ->  ~/.bash_profile  ->  **~/.bashrc**  ->  /etc/bashrc
    > 

05. 기본 명령어 

| 파일 | 설명 |
| --- | --- |
| /etc/profile | 시스템 전역 초기화 파일, 로그인 셸에서 실행 |
| /etc/bashrc | /etc/bashrc와 함께 시스템의 일반적인 설정을 하지만, bashrc 파일보다는 이 파일이 보다 일반적으로 사용 |
| /etc/bash_profile.d/*.sh | 시스템의 내부의 각자의 설정을 sh로 확장자를 가진 파일로 설정 |
| ~/.bash_profile | 각 사용자의 홈 디렉토리에 존재하는 파일, 사용자의 로그인 과정에서 /etc/profile 다음 인식되며, 개별적으로 적용되는 셸 환경 설정 파일 |
| ~/.bashrc | 사용자가 정의한 별칭, alias, function 등을 사용하는 파일이며, 셸의 실행될 때마다 실행 |
| ~/.bash_logout | 사용자가 로그아웃 할 때 실행할 것들을 파일에 정의하는 파일 |
| BASH 쉘 환경설정 파일 | ~/.bash_profile, ~/.bashrc, ~/.bash_logout |
| 본 셸 환경 설정 파일 | ~/.profile |

<br>


1. 명령어 종류
- 참고 : 디렉터리 생성 및 삭제 후 tree 명령어로 구조 확인
    - 설치
        - $sudo apt install tree
        - 즐겨찾기 기능의 심볼릭 링크 확인도 가능

<br>

| **기능** | **형식** | **옵션** |
| --- | --- | --- |
| **명령어 도움말 보기** | `man <옵션> 키워드` | `space` : 다음 페이지<br>`enter` : 다음 줄<br>`b` : 뒤로 가기<br>`q` : 종료 |
| **파일 목록 보기** | `ls` | `-l` : 파일/디렉터리 상세 보기<br>`-a` : 숨겨진 파일까지 모두 보기<br>`-d` : 내용이 아닌 디렉토리 자체 보기<br>`-F` : 파일의 종류 의미 |
| **디렉토리 생성** | `mkdir <옵션> <디렉토리 이름>` | `-p` : 존재하지 않는 상위 디렉토리 생성<br>`-m` : 퍼미션 설정 |
| **디렉터리 삭제** | `rmdir <옵션> <디렉토리 이름>` | 기본은 빈 디렉토리만 삭제<br>`-p` : 상위 디렉토리까지 삭제 |
| **파일이나 디렉토리 삭제** | `rm <옵션> 파일이름 또는 디렉토리` | `-r` : 하위 내용을 포함한 디렉토리 삭제<br>`-f` : 파일을 삭제할 때 묻지도 따지지도 말고 삭제<br>`-i` : 파일을 삭제할 때 삭제 여부 확인 후 삭제 |
| **파일 복사** | `cp <옵션> 원본파일명 목적지파일명` | `-i` : 복사할 때 overwrite 할 것인지<br>`-f` : 복사할 때 overwrite 질문 없이 무조건 덮어쓰기<br>`-r` : 디렉토리 복사 |
| **파일 이동** | `mv <옵션> 원본파일명 새이름` | `-i` : 이름 변경 시 overwrite할 것인지<br>`-f` : 변경 시 overwrite 질문 없이 무조건 덮어쓰기 |


<br>

1. 명령어들 일부 사용 예시
    - root 경로로 이동 후 -F 옵션으로 파일 목록 보기

2. ln 링크 구성 명령어
    1. **링크란? 즐겨찾기**
    2. 명령어
        -  **$ln -s file link명**
        -  **확인명령어**
            -  **$ls -F**


### 파일의 종류를 나타내는 기호 의미
    
    - `/` : 디렉터리 (폴더)
    - `*` : 실행 가능한 파일 (executable file)
    - `@` : 심볼릭 링크 (symbolic link)
    
3.**파일 생성 및 내용 작성 명령어**

**Linux 파일 시스템은 모든 것을 파일로 간주(명령어들도 파일로 관리 및 제공)**
- cat : 파일을 만들고, 파일의 내용을 표시, 줄 번호를 표시
    - touch : 새 파일(다수)을 만들고 기존 파일 및 디렉터리의 타임스탬프를 업데이트하는 데 사용
        - 리디렉션(>) 기호 사용 :  파일을 만들 수도 있음
        - ‘>’ operator 를 사용 하면 기존 파일을 덮어쓰고 >> operator는 출력
    
- echo : 파일을 만드는 데 사용
- vi : 텍스트 편집기를 사용하여 파일을 만들 수 있음

<br>

---
<br>

### 리눅스 시스템 명령어

Ubuntu 설치 후 명령어등을 통해 시스템 정보를 확인하고 네트워크, 디스크, 메모리 등 주요 기능이 정상적으로 동작하는지 점검할 수 있음

### **1. 시스템 정보 확인**


1. **Ubuntu 버전 확인**

현재 설치된 Ubuntu의 버전 및 배포 정보를 확인

```sql
$lsb_release -a

No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 22.04.3 LTS
Release:        22.04
Codename:       jammy
```

2. **커널 버전 확인**
- 현재 사용 중인 Linux 커널 버전을 확인

```sql
uname -r

5.15.0-121-generic
```

1. **시스템 아키텍처 확인**
- 시스템이 32비트 또는 64비트인지 확인할 수 있음

```java
uname -m

x86_64
```

### **2. 네트워크 설정 확인**

**IP 주소 확인**

네트워크 인터페이스 및 IP 주소를 확인

```java
ip a
```



**인터넷 연결 확인**

외부 네트워크에 정상적으로 연결되어 있는지 확인

```java
ping google.com
```


**디스크 사용량 확인**

- 디스크의 전체 용량, 사용량, 남은 공간을 확인

```java
**df -h**

Filesystem                         Size  Used Avail Use% Mounted on
tmpfs                              794M  1.1M  793M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv   26G  7.3G   18G  30% /
tmpfs                              3.9G     0  3.9G   0% /dev/shm
tmpfs                              5.0M     0  5.0M   0% /run/lock
/dev/sda2                          2.0G  131M  1.7G   8% /boot
tmpfs                              794M  4.0K  794M   1% /run/user/1000
```


**메모리 사용량 확인**

- 시스템의 총 메모리, 사용 중인 메모리, Swap 사용량을 확인합니다.

```java
**free -h**

               total        used        free      shared  buff/cache   available
Mem:           7.8Gi       210Mi       7.2Gi       1.0Mi       347Mi       7.3Gi
Swap:          4.0Gi          0B       4.0Gi
```

### CPU 확인
- **CPU 아키텍처, 모델, 코어 수, 스레드 수, 클럭 속도 등의 CPU 관련 정보를 출력**

```java
$ lscpu

**# window host의 경우** 
Architecture:             x86_64
  CPU op-mode(s):         32-bit, 64-bit
  Address sizes:          42 bits physical, 48 bits virtual
  Byte Order:             Little Endian
CPU(s):                   4
  On-line CPU(s) list:    0-3
Vendor ID:                GenuineIntel
  Model name:             Intel(R) Core(TM) Ultra 7 155H
    CPU family:           6
    Model:                170
    Thread(s) per core:   1
    Core(s) per socket:   4
    Socket(s):            1
```

### **5. 서비스 및 프로세스 확인**

**활성화된 서비스 확인**

현재 활성화된 서비스 목록을 확인할 수 있음

```java
systemctl list-units --type=service --state=running
```

1.  **CPU 및 메모리 사용 중인 프로세스 확인**
- 실시간으로 CPU 및 메모리 사용량이 높은 프로세스를 확인

```java
top  또는 htop
```

### **6. 방화벽 상태 확인**

Ubuntu의 기본 방화벽인 UFW(Uncomplicated Firewall)가 활성화되어 있는지 확인

```java
sudo ufw status
```

### **7. SSH 상태 확인 (원격 접속 설정이 필요한 경우)**


SSH 서비스가 활성화되어 있는지 확인

원격 접속이 필요하다면 SSH 설정을 확인하는 것이 매우 중요

```java
sudo systemctl status ssh
```

### **8. 설치된 패키지 목록 확인**

시스템에 설치된 모든 패키지 목록을 확인할 수 있음

실행창 종료 단축키 : ctrl + z

```java
dpkg --list
```


### **9. 시간 및 날짜 확인 (타임존 확인)**

현재 시스템 시간이 올바르게 설정되어 있는지, 타임존이 맞는지 확인

```java
timedatectl
```
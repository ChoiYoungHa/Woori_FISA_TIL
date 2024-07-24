# FISA 클라우드엔지니어링 12일차

### 우분투에  Docker 사용해서 mysql, oracle 설치

```bash
sudo apt-get update
sudo apt install -y docker.io
sudo tail /etc/group
**$sudo usermod -a -G docker $USER
$newgrp docker # 설정한 그룹 즉각 인식하는 명령어, 생략시 재부팅 후에만 group 적용**
```

- docker 설치 후 docker 관련 명령어는 sudo를 사용해야 사용 가능했음.
- 하지만 docker 그룹에 user를 추가하면 sudo 없이 사용 가능

**$newgrp docker**

- rebooting 없이 docker 그룹 포함

**사용자가 docker 그룹에 포함된 사항 확인**

- **$tail /etc/group**

### Docker Nginx 설치하기

```bash
docker pull nginx
docker images
docker run -d -p 80:80 nginx
docker ps -a
docker inspect <container_id>
$ curl http://localhost:80
docker exec -it <container_id> bash
```

### Docker Tomcat 설치하기

```bash
docker run --name mytomcat **-d** -p 8080:8080 tomcat:9.0
docker inspect <container_id>
$ curl http://localhost:8080
docker exec -it <container_id> bash
```

### 컨테이너의 운영체제 정보 확인하기

```bash
**# 컨테이너의 운영 체제 정보 확인**
$docker exec 66552458d1bc sh -c 'cat /etc/os-release'
```

### 컨테이너 운영체제 접속할 때 인코딩 설정하기

```bash
# 한글 인코딩 해결책
**$docker exec -it -e LC_ALL=C.UTF-8 31d69847bf98 bash**
root@31d69847bf98:/# vi /usr/share/nginx/html/index.html
```

### Ubuntu 환경에서 한글 깨지지 않도록 설정하기

```bash
# locales 설치
apt-get -y update
apt-get install -y locales

# 한글 패키지 다운로드
apt-get install -y language-pack-ko

# locale-gen 명령어를 사용하여 locale 구성하기
# 템플릿을 사용하여 locale 구성하기
locale-gen ko_KR.utf8

#locale
```

### Docker Oracle 설치 후 호스트에서 Connection 해보기

```bash
docker search oracle-xe-11g
docker pull oracleinanutshell/oracle-xe-11g
docker run -p -d 1521:1521 oracleinanutshell/oracle-xe-11g
docker exec -it {containerid} bash
$sqlplus
$id : system
$pw : oracle
```

### Docker 기타명령어

```bash
docker container prune : 중지된 모든 컨테이너 삭제

docker image prune : 사용하지 않는 이미지 삭제(dangling images)

docker volume prune : 컨테이너와 연결되지 않은 모든 볼륨 삭제

docker network prune : 컨테이너와 연결되지 않은 모든 네트워크 삭제

docker system prune -a : 사용하지 않는 모든 오브젝트를 삭제`
```

### 미션

- sql 분석 후 ELK + RDBMS 관련된
- 타이타닉 데이터 → mysql → logstash → elasticsearch → 시각화 → html iframe 발표
- 데이터 표현 선별은 팀원들끼리
- 시작은 오늘 저녁부터 소통 및 회의는 내일부터 실제 구축은 금요일이고, 발표는 금요일 4시
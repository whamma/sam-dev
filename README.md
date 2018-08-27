# SAM 개발환경
SAM 개발환경을 구축하기 위해 웹서버, DB등을 반복 설치하는 번거로움을 개선하기 위해 Docker를 이용한 개발환경 설정 방안이다.

Docker환경을 목적에 맞게 만들수도 있겠으나, Laravel 개발만을 위해 최대한 간편하게 사용할 수 있는 laradock(http://laradock.io)을 사용하도록 한다.


## Docker 설치

우선 Docker를 설치해야 한다. 리눅스, 윈도우, 맥 모든 OS를 지원하기 때문에 본인의 OS에 맞게 설치한다.

> https://www.docker.com/community-edition#/download

Docker가 잘 설치되었는지 확인한다.

> $ docker -v

## 프로젝트에 개발환경 적용하기

### 1. sam-api 프로젝트 git 클론 하기

> $ git clone ssh://git@geminisoft.iptime.org/sbs/sbs-sam/sam-api.git sam-api

### 2. git 서브모듈 초기화

sam-api 프로젝트 디렉터리로 이동한다.

> $ cd sam-api

프로젝트 루트에 laradock이라는 폴더가 있지만 비여있을 것이다.

아래 명령으로 서브 모듈을 초기화 시킨다.

> $ git submodule init

초기화가 성공하면 아래와 같음 메세지가 출력된다.

```
Submodule 'laradock' (ssh://git@geminisoft.iptime.org/sbs/sbs-sam/sam-api.git) registered for path 'laradock'
```

그런 다음 아래 명령어로 submodule을 업데이트 한다.

> $ git submodule update

명령이 성공하면 아래와 같은 메세지가 출력된다.

```
Cloning into 'D:/dev/sam-api/laradock'...
Submodule path 'laradock': checked out '79ad68b771ae7b0e5c03d48be31fa359c293a928'
```

## laradock 환경설정

laradock 디렉터리로 이동한다.

> $ cd laradock

env-example 파일을 .env로 복사한다.

> $ cp env-example .env

.env파일을 열어 아래 항목을 수정한다.

```
# DB데이터 저장되는 위치
14 DATA_PATH_HOST=~/.sam-api/data
# 프로젝트 이름: 컨테이너 명 앞에 prefix로 붙는다 예를들어 nginx컨테이너면 sam-api_nginx_1으로 컨테이너가 실행된다.
31 COMPOSE_PROJECT_NAME=sam-api
# PHP 버전
38 PHP_VERSION=7.2
# 타임존
120 WORKSPACE_TIMEZONE=Asia/Seoul
``` 

## Docker Compose를 이용한 서비스 컨테이너 관리

1개 이상의 서비스 컨테이너를 다룰 때 Docker Compose를 활용한다. 근데 보통 nginx기준으로 nginx, php-fpm, workspace, mysql처럼 4개 이상을
사용하기 때문에 docker명령어로는 많이 번거롭다. 이를 위해 docker-compose 명령어를 사용하는 것이다.

모든 docker 명령어는 laradock 디렉터리 내에서 실행해야 한다.

### Docker 컨테이너 빌드/시작

아래 명령으로 nginx와 mysql 컨테이너를 빌드/시작 시킨다.

> $ docker-compose up nignx mysql

백그라운드로 실행하려면 -d 플래그를 추가한다.

> $ docker-compose up -d nignx mysql

처음으로 docker-compose up {컨테이너}명령을 실행하면 docker이미지를 자동으로 빌드하는데 컨테이너명 없이 그냥 docker-compose up 해버리면
laradock에서 지원하는 모든 컨테이너를 빌드하려고 하기 때문에 시간도 많이 걸리고, 쓸데없는 docker 이미지들이 빌드되어 하드디스크를 낭비하게 된다.
꼭! docker-compose up 뒤에 사용할 컨테이너명을 입력하도록 하자.

### Docker 컨테이너 빌드

혹시 필요에 따라 .env를 수정하거나 docker-compose.yml을 수정했다면 아래 명령으로 docker 이미지를 다시 빌드하도록 한다.

> $ docker-compose build nginx mysql

빌드 명령어 역시 반드시 컨테이너명을 입력하도록 한다.

### Docker 컨테이너 중지/제거

아래 명령을 실행하면 실행중인 컨테이너가 중지되고 삭제된다.

> $ docker-compose down

컨테이너가 삭제되면 컨테이너 내에서 했던 모든 작업이 초기화 된다.(라이브러리 설치, cron설정, php.ini설정 등)
nginx, mysql, php 설정을 아예 컨테이너 빌드 시 적용하려면 laradock 하위에 컨테이너 별로 폴더가 있는데 이곳에서 example이라는 문자열이 붙어있는
각, 컨테이너별 환경설정 파일을 복사해서 수정하고, docker이미지를 다시 빌드 시키면 해당 사항이 적용된다.(아직까지 개발 시는 기본으로 해도 크게 관계 없는듯...)

### Docker 컨테이너 시작

컨테이너를 시작시키려면 아래 명령어를 입력한다.

> $ docker-compose start

### Docker 컨테이너 종료

컨테이너를 종료시키려면 아래 명령어를 입력한다.

> $ docker-compose stop

### Docker 컨테이너 재시작

컨테이너를 재시작하려면 아래 명령어를 입력한다.

> $ docker-compose restart

### workspace 터미널 실행

모든 개발환경은 workspace 컨테이너에 설정되어 있으므로 의존성 패키지 설치 및 php artisan, phpunit과 같은 명령은 workspace에 진입해서 수행해야 한다.

아래 명령으로 workspace에 진입할 수 있다.

> $ docker-compose exec workspace bash

혹시 윈도우즈에서 다음과 같은 오류가 발생하면 명령어 앞에 winpty를 붙여서 실행한다.

```
the input device is not a TTY.  If you are using mintty, try prefixing the command with 'winpty'
```

> $ winpty docker-compose exec workspace bash

이후 composer install과 같은 명령을 수행할 수 있다.

### workspace 터미널 종료

> $ exit

## sam-api 개발 시작...

개발환경이 준비되고 "docker-compose up nginx mysql"또는 "docker-compose start"명령으로 개발에 필요한 컨테이너들를 구동시켰다면 다음과 같은 절차를 진행한다.

### 1. 라라벨 개발환경 설정

workspace에 접속한다.

> $ docker-compose exec workspace bash

.env-example을 복사하여 .env(닷이엔브이)를 생성한다.

> $ cp .env.example .env

.env파일을 열어 자신의 환경과 다른부분은 수정한다.

### 2. 의존성 패키지 설치

workspace에서 의존성 패키지를 설치한다.

> $ composer install

### 3. 데이터베이스 마이그레이션

아래 명령으로 데이터베이스를 마이그레이션/시딩 한다.

> $ php artisan migrate --seed

### 4. 서비스 확인

http://localhost로 접속하여 서비스가 정상적인지 확인한다.


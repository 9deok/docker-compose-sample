# 목차

1. [개요](#개요)
2. [Compose를 효과적으로 만들기 위한 네가지 기능](#Compose를-효과적으로-만들기-위한-기능)
3. [샘플 시작하기](#Getting-Started)

# 개요

## Compose 개요

Compose는 다중 컨테이너 Docker 애플리케이션을 정의하고 실행하기 위한 도구입니다.
Compose에서는 YAML 파일을 사용하여 애플리케이션 서비스를 구성합니다.
그런 다음 단일 명령으로 구성에서 모든 서비스를 만들고 시작합니다.

→ 하나의 서비스에서 여러개의 컨테이너를 사용해야 할경우 매번 docker build run stop rm restart 등의 명령어를 사용하지 않고 compose up 명령어로 한번에 편하게 여러개의 컨테이너를 실행하기 위함.

## Compose 기본 프로세스

1. `DockerFile`로 어디서나 재현 할 수 있도록 앱의 환경을 정의한다.
2. 앱을 구성하는 서비스를 정의하여 `docker-compose.yml` 이라는 격리된 환경에서 함께 실행할 수 있다.
3. `docker compose up` 으로 실행하면 Docker compose 명령이 전체 앱을 시작하고 실행한다.

docker-compose.yml 예시

```yaml
version: "3.9"  # optional since v1.27.0
services:
  web:
    build: .
    ports:
      - "5000:5000"
    volumes:
      - .:/code
      - logvolume01:/var/log
    links:
      - redis
  redis:
    image: redis
volumes:
  logvolume01: {}
```

Compose에는 애플리케이션의 전체 수명주기를 관리하기 위한 아래와 같은 명령이 있다.

- 서비스 시작, 중지 및 재구축
- 실행 중인 서비스 상태 보기
- 실행 중인 서비스의 로그 출력 스트리밍
- 서비스에서 일회성 명령 실행

# Compose를 효과적으로 만들기 위한 기능


## 단일 호스트에서 여러 개의 격리된 환경 구성

Compose는 프로젝트 이름을 사용하여 환경을 서로 격리한다.

- dev 호스트에서 프로젝트의 각 기능 분기에 대해 안정적인 복사본을 실행하려는 경우와 같이 단일 환경의 여러 복사본을 만들 수 있다.
- CI 서버에서 빌드가 서로 간섭하지 않도록 하기 위해 프로젝트 이름을 고유한 빌드 번호로 설정할 수 있다.
- 공유 호스트 또는 개발 호스트에서 동일한 서비스 이름을 사용할 수 있는 다른 프로젝트가 서로 간섭하는 것을 방지할 수 있다.

기본 프로젝트 디렉토리는 Compose 파일의 기본 디렉토리입니다. 이에 대한 사용자 지정 값은 `--project-directory`명령줄 옵션 으로 정의할 수 있습니다.

## 컨테이너 생성 시 볼륨 데이터 보존

Compose는 서비스에서 사용하는 모든 볼륨을 보존합니다. `docker-compose up` 을 실행 할 때 이전 실행에서 컨테이너를 찾으면 이전 컨테이너에서 새 컨테이너로 볼륨을 복사합니다. 이 프로세스는 볼륨에서 생성한 데이터가 손실되지 않도록 합니다.

## 변수 및 환경간 구성 이동

Compose는 Compose 파일의 변수를 지원합니다. 이러한 변수를 사용하여 다양한 환경 또는 다른 사용자에 맞게 구성을 사용자 지정할 수 있습니다.

`extends`필드를 사용하거나 여러 작성 파일을 작성 하여 작성 파일을 확장할 수 있습니다 .

# Getting Started

docker 공식문서에서는 python Flask + Redis로 샘플을 정의해 놓았다.
해당 문서에서는 Springboot + mariaDB로 샘플을 정의한다.

## 1단계 : 설정

1. [https://start.spring.io/](https://start.spring.io/) 로 접속하여 gradle기반 프로젝트를 생성한다. ( dependency `Spring Web` 추가)
2. 프로젝트 루트 디렉토리에 `Dockerfile` 이라는 이름으로 파일 생성
3. 프로젝트 루트 디렉토리에 `docker-compose.yml` 이라는 이름으로 파일 생성
4. src→main→resource에 `db_config`라는 이름의 빈 디렉토리 생성 후 `conf.d`,`data`,`initdb.d` 디렉토리 생성
5. `conf.d` 안에 `my.cnf` 파일을 만들고 아래의 내용을 작성

    ```yaml
    [client]
    default-character-set = utf8mb4

    [mysql]
    default-character-set = utf8mb4

    [mysqld]
    character-set-client-handshake = FALSE
    character-set-server           = utf8mb4
    collation-server               = utf8mb4_unicode_ci
    ```

6. `initdb.d` 안에 빈 `create_table.sql` 과 `load_data.sql` 파일을 생성 ( docker 컨테이너 최초 실행시 불러올 스크립트이다 필요시 사용 )
7. Gradle Task → build → bootjar 실행
8. TestController 작성

    ```java
    @RestController
    public class TestController {
        @GetMapping("/test")
        public String test() {
            return "Hello World";
        }
    }
    ```

## 2단계 : Dockerfile 작성

1. Dockerfile 작성

    ```yaml
    FROM openjdk:11-jre-slim
    ARG JAR_FILE=build/libs/*.jar
    COPY ${JAR_FILE} app.jar
    ENTRYPOINT ["java","-jar","/app.jar"]
    ```

## 3단계 : Compose 파일에서 서비스 정의

1. docker-compose.yml 작성 - [공식 사이트 문법 설명 참조](https://docs.docker.com/compose/compose-file/compose-file-v3/)

    ```yaml
    version: "3.9"
    services:
      app:
        build: .
        ports:
          - 8080:8080
        environment:
          SPRING_DATASOURCE_URL: jdbc:mariadb://database:3306/lunit?useUnicode=true
          SPRING_DATASOURCE_USERNAME: root
          SPRING_DATASOURCE_PASSWORD: 1234
        restart: always
        depends_on:
          - database
      database:
        image: mariadb
        environment:
          - MYSQL_DATABASE=lunit
          - MYSQL_ROOT_PASSWORD=1234
          - MYSQL_ROOT_HOST=%
        volumes:
          - ./src/main/resources/db_config/conf.d:/etc/myslq/conf.d
          - ./src/main/resources/db_config/data:/var/lib/mysql
          - ./src/main/resources/db_config/initdb.d:/docker-entrypoint-initdb.d
        ports:
          - 3306:3306
    ```

    - version : compose file의 버전을 나타낸다. [최신버전 정보](https://docs.docker.com/compose/compose-file/)에 3.8로 나와있지만 [공식문서 예제](https://docs.docker.com/compose/networking/)에 3.9로 안내하고 있어서 3.9로 작성하였다.
    - services : 이 항목 밑에 실행하고자 하는 컨테이너들을 정의한다. 도커 컨테이너 목록으로 이해하면 된다.
        - app : 서비스의 이름을 `app` 으로 정의
            - build : docker build 명령을 사용할 디렉토리 ( dockerfile을 미리 생성해 놓은 이유 )
            - ports : docker run 명령어의 -p 옵션에 해당. 외부포트와 내부포트 매핑한다.
            - environment : springboot 앱의 환경변수를 전달하는 역할 application.yml과 매핑된다고 생각하면 된다.
            - restart : 컨테이너가 종료 되었을 경우 다시시작하게 하는 옵션 `no`, `always`, `on-failure`, `unless-stopped` 옵션이 있다.
            - depneds_on : 컨테이너의 의존성을 명시한다. `databaes` 서비스에 의존하고 있으므로 database 이미지가 먼저 실행 후 `app`을 실행한다.
        - database : 서비스의 이름을 `database` 로 정의
            - image : 사용할 도커 이미지를 적는다. dockerhub 의 [공식 mariadb 이미지](https://hub.docker.com/_/mariadb)를 사용하였다.
            - environment : docker run 시에 명령어 -e에 해당된다. 이미지를 생성하며 스키마와 계정정보를 셋팅한다.
            - ~~command : 이미지를 띄울 때 해당 charset에 대한 파라미터를 넘겨 charset을 utf8mb4(이모지 + 한글 지원)로 셋팅한다.~~
            - volumes : 컨테이너를 지우거나 재시작해도 데이터를 남기기 위해 컨테이너 내부 데이터와 마운트 시키는 작업.( 내 로컬에 저장 )
            - ports : docker run 명령어의 -p 옵션에 해당. 외부포트와 내부포트 매핑한다.

   networking 의 경우 따로 지정하지 않으면 `project name_default`의 형태로 단일 네트워크가 생성된다.

   위 compose.yml 네트워크 생성 프로세스

    1. default 네트워크 생성
    2. app의 구성을 사용하여 컨테이너 생성 → default 네트워크에 합류
    3. database의 구성을 사용하여 컨테이너 생성 → default 네트워크에 합류

   네트워크가 생성된 이후에는 각 컨테이너의 호스트이름의 조회하거나 IP주소를 가져올 수 있다.

   ex) jdbc:mariadb://`database`:3306/lunit?useUnicode=true

   위 예시에서 ip주소를 직접 적지 않고 컨테이너 이름인 database를 적어 ip를 대신하였다.

## 4단계 : Compose로 앱 빌드 및 실행

1. `docker-compose up` 명령어 실행
2. [localhost:8080/test](http://localhost:8080/test) 로 접속하여 "Hello World" 가 잘 표시되는지 확인한다.
3. `docker-compose down` 명령어 실행
4. TestController를 수정 "Hello World"→ "Hello World Complete!!"
5. gradle task의 bootjar를 다시 실행.
6. docker 이미지를 삭제
7. `docker-compose up` 명령어 실행
8. [localhost:8080/test](http://localhost:8080/test) 로 접속하여 변경된 "Hello World Complete!!" 가 잘 표시되는지 확인한다.

현재 소스코드 변경후 적용 과정이 굉장히 귀찮다.

도커 내부에 소스코드와 gradle 파일들을 카피하고 jar파일을 다시 만든 후에 컨테이너를 재생성 하는 방식으로 진행을 시도해 보았으나 삽질만 하다 끝났다.

java8, gradle 6.6.1 에서는 성공하였으나 java11 , gradle 7.1.1에서는 실패하였다 추후 방법을 찾아 문서를 수정해야 한다.
추가정보 : jdk11은 openjdk11-alpine 공식 이미지가 없다.
# Dockerfile

### Dockerfile to Container

![](https://github.com/dragonappear/docs/assets/89398909/6efc5008-e069-4ab9-a13d-9367717ff499)

### [FROM](https://docs.docker.com/engine/reference/builder/#from)

```dockerfile
FROM alpine
```

- 베이스 이미지 선정
- 반드시 있어야 하는 명령어
- 이미지 레이어 중 가장 기본이 되는 이미지

### [WORKDIR](https://docs.docker.com/engine/reference/builder/#workdir)

```dockerfile
FROM alpine

WORKDIR /usr/src/app
```

- work directory 선정
- 뒤에 오는 모든 지시자(`RUN`,`CMD`,`COPY`,`ADD` 등)에 대한 작업 디렉토리 설정
- 리눅스 명령어의 `cd`와 비슷한 역할을 한다고 생각하자


### [COPY](https://docs.docker.com/engine/reference/builder/#copy)

```dockerfile
FROM alpine

WORKDIR /usr/src/app

COPY ./ ./
```

- 호스트 파일을 도커 컨테이너 안으로 복사함

### [RUN](https://docs.docker.com/engine/reference/builder/#run)

```dockerfile
FROM alpine

WORKDIR /usr/src/app

COPY ./ ./

RUN pip install -r requirements
```

- **이미지 빌드 시** 실행할 명령어

### [CMD](https://docs.docker.com/engine/reference/builder/#cmd)

```dockerfile
FROM ubuntu:16.04

CMD ["echo", "CMD test"]
```

```Bash
docker build . -f dockerfile_cmd

docker run cmd_test
```

- **컨테이너 생성 시** 실행할 명령어
- `RUN` 명령어는 이미지를 빌드할 때 실행되는 것과 달리, `CMD` 명령어는 이미지로부터 **컨테이너를 생성하여 최초로 실행할 때 수행된다.**


### [ENTRYPOINT](https://docs.docker.com/engine/reference/builder/#entrypoint)

```dockerfile

```

- **컨테이너가 생성되고 최초로** 실행될 명령어(ex: 서버 실행)

### Docker Image Layer

이미지 레이어 의존성 체크
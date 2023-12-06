# Docker Commands

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

### RUN

### [CMD](https://docs.docker.com/engine/reference/builder/#cmd)

### ENTRYPOINT


### Docker Build

이미지 레이어 의존성 체크
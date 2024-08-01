### Dockerfile 및 Docker 관련 주요 내용 요약

#### Dockerfile의 특징
- **환경 일관성**: 동일한 환경에서 애플리케이션을 실행 가능
- **이식성**: 다양한 운영체제에서 동일하게 동작
- **자동화**: 빌드 및 배포 자동화
- **반복 가능성**: 동일한 결과를 반복적으로 얻을 수 있음
- **확장성**: 쉽게 확장 가능

#### Dockerfile 지시자
- **`FROM`**: 베이스 이미지를 지정
- **`RUN`**: 명령어 실행
- **`CMD`**: 컨테이너가 시작될 때 실행할 기본 명령어
- **`LABEL`**: 이미지에 메타데이터 추가
- **`EXPOSE`**: 컨테이너에서 사용할 포트를 지정
- **`ENV`**: 환경 변수 설정
- **`ADD`**: 파일을 복사하고 압축을 해제
- **`COPY`**: 파일을 복사
- **`ENTRYPOINT`**: 컨테이너 시작 시 실행할 명령어 지정
- **`VOLUME`**: 호스트와 컨테이너 간의 디렉토리를 공유
- **`USER`**: 사용자 설정
- **`WORKDIR`**: 작업 디렉토리 설정
- ARG
- HEALTHCHECK
- MAINTAINER
- ONBUILD
- STOPSIGNAL

##### COPY 와 ADD 의 차이점 
COPY
- 빌드 컨텍스트 또는 멅 스테이지 빌드의 단계에서 파일을 컨테이너에 복사
ADD
- 원격 HTTPS or Git URL 에서 파일을 가져오기
- 빌드 컨텍스트에서 파일을 추가 할 때 tar 파일을 자동으로 추출

멀티 스테이지 빌드일때
- 한 단계에서 다른 단계로 파일을 복사하려면 COPY 사용
- 빌드 컨텍스트에서 컨테이너 파일로 임시로 추가하여 RUN 명령어를 실행해야 하는 경우, COPY 대신 바인드 마운트를 사용
- 바인드 마운트?
	```yaml
	RUN --mount=type=bind,source=requirements.txt,target=/tmp/requirements.txt \
	    pip install --requirement /tmp/requirements.txt
	```
	- 빌드 컨텐스트에서 컨테이너로 파일을 포함할때 COPY보다 효율적
	- 바인드 마운트

#### Docker 최적화 방법
1. **이미지 최적화**: `.dockerignore` 파일 사용하여 불필요한 파일 제외
2. **레이어 캐싱**: 자주 변경되지 않는 명령어를 상단에 배치하여 빌드 속도 증가
3. **보안 고려**: 비밀 정보는 환경 변수로 관리하지 않고 별도 파일로 관리
4. **성능 최적화**: 경량화된 베이스 이미지 사용
5. **유지 보수성**: 명확하고 일관된 Dockerfile 작성
6. **환경 변수 사용**: `ENV`를 통해 환경 변수 설정

#### Docker 저장소 및 이미지 관리
- **Docker Hub**: 이미지 호스팅 서비스
- **Docker Local Registry**: 로컬에서 이미지 관리

#### Docker 실습 예제
1. **개발자가 공유한 코드를 Docker 이미지로 생성**
    ```Dockerfile
    FROM ubuntu
    COPY . .
    RUN apt update && apt install -y python3 python3-pip && pip install flask
    CMD ["python3", "app.py"]
    ```

2. **Flask 애플리케이션 예제 (`app.py`)**
    ```python
    from flask import Flask, jsonify
    app = Flask(__name__)
    @app.route('/')
    def home():
        return jsonify(message="Hello, Docker World!")
    if __name__ == '__main__':
        app.run(host='0.0.0.0', port=5002)
    ```

3. **단일 스테이지 Dockerfile**
    ```Dockerfile
    FROM python:3.11-slim
    WORKDIR /app
    COPY requirements.txt ./
    RUN pip install --no-cache-dir -r requirements.txt
    COPY . .
    CMD ["python", "app.py"]
    ```

4. **Docker 명령어**
    ```sh
    docker build -t bootcamp .
    docker tag bootcamp lyle/kakao-bootcamp:latest
    docker push lyle/kakao-bootcamp:latest
    docker pull lyle/kakao-bootcamp
    docker run -it lyle/kakao-bootcamp /bin/bash
    ```

#### Multi-Stage Build 예제
1. **싱글 스테이지**
    ```Dockerfile
    FROM golang:1.20
    WORKDIR /app
    COPY go.mod go.sum ./
    RUN go mod download
    COPY . .
    RUN go build -o myapp .
    WORKDIR /root/
    CMD ["./myapp"]
    EXPOSE 8080
    ```

2. **멀티 스테이지**
    ```Dockerfile
    FROM golang:1.20 AS builder
    WORKDIR /app
    COPY go.mod go.sum ./
    RUN go mod download
    COPY . .
    RUN go build -o myapp .

    FROM alpine:latest
    WORKDIR /root/
    COPY --from=builder /app/myapp .
    CMD ["./myapp"]
    EXPOSE 8080
    ```

#### Q&A 요약
1. **의존성 문제**: 의존성을 알려주지 않는 경우도 있으므로 직접 테스트 필요
2. **윈도우에서 리눅스 이미지**: WSL2를 사용하여 가능
3. **도커 사용 환경**: 보통 EC2 서버를 사용
4. **로컬 빌드 후 AWS 배포**: 로컬에서 이미지 빌드 및 테스트 후 AWS에 배포
5. **이미지 생성 위치**: 모든 작업은 호스트에서 진행
6. **플랫폼 지정 빌드**: `docker buildx build --platform linux/amd64 --load -tag lyleseungju/kakaobootcamp2 .`
7. **requirements.txt**: 개발자로부터 제공받음
8. **불필요한 패키지**: 불필요한 패키지는 다이어트 필요
9. **CMD와 ENTRYPOINT**: CMD는 대체 가능, ENTRYPOINT는 무조건 실행

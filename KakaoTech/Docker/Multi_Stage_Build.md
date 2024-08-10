
#### 멀티 스테이지 빌드의 작동
```
# syntax=docker/dockerfile:1
FROM golang:1.21
WORKDIR /src
COPY <<EOF ./main.go
package main

import "fmt"

func main() {
  fmt.Println("hello, world")
}
EOF
RUN go build -o /bin/hello ./main.go

# 최종 빌드
FROM scratch
COPY --from=0 /bin/hello /bin/hello
CMD ["/bin/hello"]
```
1. 첫번째 빌드에서 생성된 이미지를 기반으로
2. FROM scratch 를 기반으로 하는 이미지를 불러오며 새로운 스테이지 생성
3. 아까 첫번째 빌드된 이미지를 복사하고 나머지 데이터는 가져오지 않음
4. 이를 통해 정말 필요한 데이터만 최종 이미지에 가져가게됨

이것저것
#### AS를 통해 빌드되는 이미지에 이름 붙이기
```
# syntax=docker/dockerfile:1
FROM golang:1.21 AS build
WORKDIR /src
COPY <<EOF /src/main.go
package main

import "fmt"

func main() {
  fmt.Println("hello, world")
}
EOF
RUN go build -o /bin/hello ./main.go

FROM scratch
COPY --from=build /bin/hello /bin/hello
CMD ["/bin/hello"]
```

#### 특정 빌드 스테이지에서 멈추기
```yaml
# 개발 환경
FROM node:16 AS development
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["npm", "run", "dev"]

# 빌드 환경
FROM node:16 AS build
WORKDIR /app
COPY --from=development /app .
RUN npm run build

# 디버그 환경
FROM node:16-slim AS debug
WORKDIR /app
COPY --from=build /app/build .
RUN apt-get update && apt-get install -y gdb
CMD ["node", "index.js"]

# 프로덕션 환경
FROM node:16-slim AS production
WORKDIR /app
COPY --from=build /app/build .
RUN npm install --production 
CMD ["node", "index.js"]
```
이렇게 만들어 두고 사용할때
 - 디버그 `docker build --target debug -t myapp:debug .`
 - `docker build --target prod -t myapp:prod .`

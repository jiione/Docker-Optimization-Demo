# 🐳 Docker 이미지 경량화 가이드

**Docker 이미지를 경량화**하기 위한 다양한 기술과 모범 사례를 소개합니다. 이를 통해 Docker 이미지 크기를 줄여 **배포 속도와 성능을 최적화하고, 보안성**을 높일 수 있습니다.

## 1. 🏗️ 최소한의 베이스 이미지 사용

도커 이미지의 크기를 줄이기 위해 최소한의 베이스 이미지를 사용하는 것이 좋습니다. 일반적으로 Ubuntu는 약 128MB의 크기를 가지지만, **Alpine** 이미지로 변경하면 약 5MB로 줄일 수 있습니다.  
그러나 더 작은 이미지를 원한다면 **Distroless** 이미지를 사용할 수 있습니다. Distroless 이미지는 애플리케이션과 필수적인 런타임만 포함하여 더 작은 크기와 더 안전한 환경을 제공합니다.

```bash
TAG                                  SIZE
debian                               124.0MB
alpine                               5.0MB
gcr.io/distroless/static-debian11    2.51MB
```

## 2. 🛠️ 멀티 스테이지 빌드 사용

멀티 스테이지 빌드를 통해 빌드 환경과 배포 환경을 분리함으로써 불필요한 파일을 최종 이미지에 포함시키지 않고 크기를 줄일 수 있습니다. 아래는 Node.js 애플리케이션에서 멀티 스테이지 빌드를 사용한 예시입니다.

```Dockerfile
# Build 단계 (빌드 환경)
FROM node:16-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .

# Production 단계 (경량화된 배포 환경)
FROM node:16-alpine
WORKDIR /app
COPY --from=build /app ./
EXPOSE 3000
CMD ["node", "app.js"]
```

## 3. 🚫 불필요한 파일 제외하기 (.dockerignore)

`.dockerignore` 파일을 사용하면 Docker 빌드 시 불필요한 파일을 제외할 수 있습니다. 이를 통해 빌드 컨텍스트에 불필요한 파일이 포함되지 않도록 하여 이미지 크기를 줄일 수 있습니다.

```plaintext
node_modules
npm-debug.log
Dockerfile*
.git
.github
dist/**
README.md
```

## 4. 🎯 이미지 레이어 최소화

이미지 레이어를 줄이기 위해 `RUN`, `COPY` 등의 명령어를 결합하여 하나의 레이어로 처리할 수 있습니다. 이는 이미지 크기를 줄이고 빌드 속도를 높입니다.

```Dockerfile
RUN apt-get update && \
    apt-get install -y git && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

## 5. 🛠️ 도구 활용

**Docker Slim**과 같은 도구를 사용하여 도커 이미지를 분석하고, 불필요한 부분을 제거해 이미지 크기를 더 줄일 수 있습니다.

```bash
docker-slim build --sensor-ipc-mode proxy --sensor-ipc-endpoint 172.17.0.1 --http-probe=false nginx;
```

## 6. 🔍 이미지 크기 비교 및 최적화

이미지 빌드 후 `docker images` 명령어로 이미지 크기를 비교하여 얼마나 최적화되었는지 확인할 수 있습니다.

```bash
docker images | grep my-node-app
```

---

# 🧪 실습: Node.js 애플리케이션에서 Docker 이미지 경량화 비교

### 🎯 목적
이 실습은 경량화된 Docker 이미지를 만드는 방법을 학습하는 데 그 목적이 있습니다. Node.js 애플리케이션을 대상으로 멀티 스테이지 빌드를 사용해 이미지를 경량화하고, 경량화 전후의 이미지를 비교하여 성능과 크기의 차이를 확인합니다.

### 🛠️ 실습 과정

1. **경량화 전 Dockerfile (`my-node-app-full`)**
   아래 Dockerfile은 경량화 없이 기본적인 Node.js 환경을 사용하여 빌드된 이미지입니다.

   ```Dockerfile
   FROM node:16
   WORKDIR /app
   COPY package*.json ./
   RUN npm install
   COPY . .
   EXPOSE 3000
   CMD ["node", "app.js"]
   ```

   빌드 명령어:
   ```bash
   docker build -t my-node-app-full .
   ```

2. **경량화 후 Dockerfile (`my-node-app-optimized`)**
   멀티 스테이지 빌드와 경량화된 Alpine 이미지를 사용하여 최적화된 Dockerfile입니다.

   ```Dockerfile
   # Build 단계
   FROM node:16-alpine AS build
   WORKDIR /app
   COPY package*.json ./
   RUN npm install
   COPY . .

   # Production 단계
   FROM node:16-alpine
   WORKDIR /app
   COPY --from=build /app ./
   EXPOSE 3000
   CMD ["node", "app.js"]
   ```

   빌드 명령어:
   ```bash
   docker build -t my-node-app-optimized .
   ```

3. **이미지 크기 비교**
   `docker images` 명령어를 사용하여 두 이미지의 크기를 확인하고 비교합니다.

   ```bash
   docker images | grep my-node-app
   ```

   출력 결과:
   ```plaintext
   REPOSITORY         TAG        IMAGE ID        CREATED        SIZE
   my-node-app-full   latest     abcdef123456    5 minutes ago  909MB
   my-node-app-optimized latest  ghijk456789     10 minutes ago  118MB
   ```

4. **이미지 레이어 분석**
   각각의 Docker 이미지가 어떻게 생성되었는지 확인하고, 각 레이어의 크기를 분석하여 최적화가 어디서 이루어졌는지 확인할 수 있습니다.

   ```bash
   docker history my-node-app-full
   docker history my-node-app-optimized
   ```

### 🔍 결과
- 경량화된 `my-node-app-optimized`는 기본 `my-node-app-full`에 비해 이미지 크기가 크게 줄어듭니다.
- 멀티 스테이지 빌드를 통해 빌드 시 불필요한 의존성을 제거하고, 경량화된 베이스 이미지로 전환하여 Docker 이미지의 크기를 최소화할 수 있었습니다.

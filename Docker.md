# Docker 환경 구성 및 ROBOTIS OMX에서 사용하는 이유

## 개요

Docker는 프로그램이 실행되는 운영체제, 라이브러리, 설정값을 하나의 독립된 환경으로 묶어 실행하는 도구입니다.

ROBOTIS OMX를 ROS 2와 함께 사용할 때는 ROS 2 버전, Ubuntu 버전, Python 패키지, CUDA 관련 설정 등이 서로 맞아야 합니다.
이 환경이 조금만 달라도 프로그램이 실행되지 않거나, 로봇 제어 과정에서 오류가 발생할 수 있습니다.

Docker를 사용하면 미리 구성된 실행 환경을 컨테이너로 실행할 수 있기 때문에, 개발 환경 차이로 인한 문제를 줄일 수 있습니다.

---

## ROBOTIS OMX를 Docker에서 진행하는 이유

ROBOTIS OMX를 Docker 환경에서 진행하는 이유는 다음과 같습니다.

### 1. ROS 2 환경을 일정하게 유지할 수 있음

ROS 2는 버전과 운영체제 조합이 중요합니다.
예를 들어 이 프로젝트에서는 컨테이너 내부에서 Ubuntu 24.04와 ROS 2 Jazzy 환경을 사용합니다.

호스트 PC의 환경을 직접 바꾸지 않고도, Docker 컨테이너 안에서 필요한 ROS 2 환경을 실행할 수 있습니다.

---

### 2. “내 컴퓨터에서는 되는데 다른 환경에서는 안 됨” 문제를 줄일 수 있음

로봇 소프트웨어는 여러 라이브러리와 패키지에 의존합니다.

Docker를 사용하면 같은 이미지에서 같은 컨테이너를 실행할 수 있기 때문에, 다른 PC나 로봇 하드웨어에서도 비슷한 환경을 재현하기 쉽습니다.

---

### 3. 호스트 PC를 보호할 수 있음

ROS 2, Python 패키지, CUDA 관련 라이브러리를 호스트 PC에 직접 설치하면 기존 환경과 충돌할 수 있습니다.

Docker 컨테이너 안에서 실행하면 실제 PC 환경과 분리되기 때문에, 실험과 개발을 더 안전하게 진행할 수 있습니다.

---

### 4. NVIDIA GPU를 활용한 AI 기능과 연결하기 좋음

Physical AI Tools나 카메라 기반 인식 기능을 사용할 경우, NVIDIA GPU와 CUDA가 필요할 수 있습니다.

이때 NVIDIA Container Toolkit을 사용하면 Docker 컨테이너 내부에서도 호스트 PC의 NVIDIA GPU를 사용할 수 있습니다.

즉, Docker는 실행 환경을 고정하고, NVIDIA Container Toolkit은 컨테이너가 GPU를 사용할 수 있도록 연결해 주는 역할을 합니다.

---

## 시스템 요구사항

| 항목          | 내용                         |
| ----------- | -------------------------- |
| 운영체제        | 모든 Linux 배포판               |
| 컨테이너 내부 환경  | Ubuntu 24.04 + ROS 2 Jazzy |
| 호스트 운영체제 조건 | 컨테이너와 버전이 반드시 일치할 필요 없음    |
| 하드웨어 요구 사항  | NVIDIA GPU(CUDA 지원)        |

---

## 필수 설치 항목

이 프로젝트를 실행하기 위해 필요한 대표적인 항목은 다음과 같습니다.

* Docker Engine
* Git
* NVIDIA Container Toolkit
* NVIDIA GPU Driver
* ROS 2 Jazzy가 포함된 Docker 이미지 또는 컨테이너 환경

---

## 1. Docker Engine 설치

Docker Engine은 Docker 컨테이너를 실행하기 위한 기본 엔진입니다.

최신 설치 방법은 Docker 공식 가이드를 참고합니다.

[Docker 공식 설치 가이드](https://docs.docker.com/engine/install/)

Ubuntu 기준 대표적인 설치 흐름은 다음과 같습니다.

```bash
sudo apt update
sudo apt install ca-certificates curl
```

Docker 공식 GPG key와 저장소를 추가합니다.

```bash
sudo install -m 0755 -d /etc/apt/keyrings

sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc

sudo chmod a+r /etc/apt/keyrings/docker.asc
```

Docker 저장소를 추가합니다.

```bash
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF
```

패키지 목록을 업데이트한 뒤 Docker를 설치합니다.

```bash
sudo apt update

sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Docker가 정상적으로 설치되었는지 확인합니다.

```bash
sudo docker run hello-world
```

---

## 2. Docker 설치 후 설정

Docker 명령어를 매번 `sudo` 없이 실행하고 싶다면 현재 사용자를 `docker` 그룹에 추가합니다.

```bash
sudo usermod -aG docker $USER
```

변경 사항을 적용하려면 로그아웃 후 다시 로그인하거나 다음 명령어를 사용할 수 있습니다.

```bash
newgrp docker
```

Docker가 부팅 시 자동으로 실행되도록 설정합니다.

```bash
sudo systemctl enable docker
```

정상 작동을 다시 확인합니다.

```bash
docker run hello-world
```

---

## 3. Git 설치

Git은 저장소를 복제하고 프로젝트 코드를 관리하기 위해 사용합니다.

```bash
sudo apt update
sudo apt install git
```

설치 확인:

```bash
git --version
```

---

## 4. NVIDIA Container Toolkit 개념

NVIDIA Container Toolkit은 Docker 컨테이너 안에서 NVIDIA GPU를 사용할 수 있게 해 주는 도구입니다.

일반 Docker 컨테이너는 기본적으로 호스트 PC의 GPU를 바로 사용할 수 없습니다.
NVIDIA Container Toolkit은 Docker와 NVIDIA GPU 사이를 연결하여, 컨테이너 내부에서도 CUDA 기반 프로그램을 실행할 수 있게 합니다.

중요한 점은 다음과 같습니다.

* NVIDIA GPU Driver는 호스트 PC에 설치됨
* Docker 컨테이너는 GPU를 직접 소유하지 않음
* NVIDIA Container Toolkit이 컨테이너에서 GPU에 접근할 수 있도록 런타임을 구성함
* `--gpus all` 옵션을 사용하면 컨테이너에서 GPU를 사용할 수 있음

즉, NVIDIA Container Toolkit은 Docker 컨테이너가 로봇 AI, 영상 처리, 딥러닝 추론 등 GPU가 필요한 작업을 수행할 수 있도록 도와주는 연결 도구입니다.

---

## 5. NVIDIA Container Toolkit 설치

NVIDIA Container Toolkit 공식 설치 가이드를 참고합니다.

[NVIDIA Container Toolkit 공식 설치 가이드](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html#with-apt-ubuntu-debian)

Ubuntu/Debian 계열에서는 다음 흐름으로 설치합니다.

필요한 패키지를 설치합니다.

```bash
sudo apt-get update

sudo apt-get install -y --no-install-recommends \
    ca-certificates \
    curl \
    gnupg2
```

NVIDIA Container Toolkit 저장소를 추가합니다.

```bash
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | \
sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg
```

```bash
curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
```

패키지 목록을 업데이트합니다.

```bash
sudo apt-get update
```

NVIDIA Container Toolkit을 설치합니다.

```bash
sudo apt-get install -y nvidia-container-toolkit
```

---

## 6. Docker 구성 가이드

NVIDIA Container Toolkit을 설치한 뒤에는 Docker가 NVIDIA 런타임을 사용할 수 있도록 구성해야 합니다.

공식 구성 가이드는 다음 문서를 참고합니다.

[Docker 구성 가이드](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html#configuring-docker)

Docker 런타임을 NVIDIA Container Runtime과 연결합니다.

```bash
sudo nvidia-ctk runtime configure --runtime=docker
```

이 명령어는 호스트 PC의 Docker 설정 파일을 수정하여 Docker가 NVIDIA Container Runtime을 사용할 수 있도록 합니다.

Docker 데몬을 재시작합니다.

```bash
sudo systemctl restart docker
```

---

## 7. GPU 사용 확인

NVIDIA GPU가 정상적으로 인식되는지 먼저 호스트에서 확인합니다.

```bash
nvidia-smi
```

Docker 컨테이너에서 GPU를 사용할 수 있는지도 확인합니다.

```bash
docker run --rm --gpus all nvidia/cuda:12.4.1-base-ubuntu22.04 nvidia-smi
```

컨테이너 내부에서 GPU 정보가 출력된다면 Docker에서 NVIDIA GPU를 사용할 준비가 된 것입니다.

---

## 8. Docker의 핵심 개념

### Image

이미지는 프로그램과 실행 환경을 담은 설계도입니다.

예를 들어 Ubuntu 24.04, ROS 2 Jazzy, 필요한 라이브러리가 포함된 이미지를 만들 수 있습니다.

---

### Container

컨테이너는 이미지를 실제로 실행한 독립된 공간입니다.

컨테이너 안에서 ROS 2 프로그램을 실행하면 호스트 PC 환경과 분리된 상태에서 로봇 소프트웨어를 실행할 수 있습니다.

---

### Volume

볼륨은 컨테이너와 실제 PC 폴더를 연결하는 기능입니다.

컨테이너가 종료되어도 코드와 데이터가 사라지지 않도록 하기 위해 사용합니다.

예시:

```bash
docker run -it --rm \
    -v ~/ROBOTIS_OMX_SW:/workspace/ROBOTIS_OMX_SW \
    ubuntu:24.04
```

이 명령어는 호스트 PC의 `~/ROBOTIS_OMX_SW` 폴더를 컨테이너 내부의 `/workspace/ROBOTIS_OMX_SW` 경로와 연결합니다.

---

## 정리

Docker는 ROBOTIS OMX를 실행하기 위한 ROS 2 개발 환경을 일정하게 유지하기 위해 사용합니다.

Docker를 사용하면 Ubuntu 24.04와 ROS 2 Jazzy 환경을 컨테이너로 실행할 수 있고, 호스트 PC의 환경 차이로 인한 오류를 줄일 수 있습니다.

NVIDIA Container Toolkit을 함께 사용하면 Docker 컨테이너 내부에서도 NVIDIA GPU를 사용할 수 있으므로, 카메라 인식이나 AI 기반 로봇 제어 기능을 실행하는 데 도움이 됩니다.

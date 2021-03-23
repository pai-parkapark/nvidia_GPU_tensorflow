## Guide for using nvidia GPU-tensorflow on RHEL8 VM(GCP)



### 1. make project

<img width="1792" alt="1_make_project" src="https://user-images.githubusercontent.com/71860179/111941952-9d9fc600-8b15-11eb-9fb8-cb792c61c4d0.png" style="zoom:40%;" >

GCP에 접속해서 새로운 프로젝트를 생성해 줍니다.



### 2. make instance

<img width="1792" alt="2_make_instance" src="https://user-images.githubusercontent.com/71860179/111942022-ca53dd80-8b15-11eb-95c8-d6563fb24bf2.png" style="zoom:40%;" >

새로 생성한 프로젝트에서 새로운 VM 인스턴스를 생성해줍니다.



### 3. instance 세부사항

<img width="1792" alt="instance" src="https://user-images.githubusercontent.com/71860179/111942427-97f6b000-8b16-11eb-9663-e6d5bbc6fb9a.png" style="zoom:40%;" >

인스턴스의 세부사항을 다음과 같이 설정해줍니다.

1. 1. Name - 원하는 이름으로 구성

   2. Region - us-central1 : us-central1-a

   3. 머신 구성

   4. 1. 머신 계열 : GPU
      2. 시리즈 : N1
      3. 머신 유형 : n1-standard-1

   5. CPU 플랫폼 : 자동

   6. GPU 유형 : NVIDIA Tesla V100

   7. GPU 수 : 2

   8. 부팅 디스크 

   9. 1. 운영체제 : Red Hat Enterprise Linux
      2. 버전 : Red Hat Enterprise Linux 8 

   10. 방화벽

   11. 1. HTTP 트래픽 허용 체크
       2. HTTPS 트래픽 허용 체크



### 4. Instance SSH 접속

<img width="1792" alt="4_ssh_instance" src="https://user-images.githubusercontent.com/71860179/111942587-e441f000-8b16-11eb-9d79-992f15adfe67.png" style="zoom:40%;" >

SSH 연결을 클릭해서 터미널을 열어줍니다.



### 5. VM에 nvidia-driver 설치

1. 드라이버를 설치하려는 VM에 연결합니다.
2. 최신 커널 패키지를 설치합니다. 필요한 경우 이 명령어는 시스템도 재부팅합니다.

```bash
sudo yum clean all
sudo yum install -y kernel | grep -q 'already installed' || sudo reboot
```

3. 이전 단계에서 시스템을 재부팅한 경우 VM에 다시 연결합니다.
4. 커널 헤더 및 개발 패키지를 설치합니다.

```bash
sudo yum install -y kernel-devel-$(uname -r) kernel-headers-$(uname -r)
```

5. `epel-release` 저장소를 설치합니다. 이 저장소에는 CentOS에 NVIDIA 드라이버를 설치하는 데 필요한 DKMS 패키지가 있습니다.

```bash
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
```

6. `yum-utils`를 설치합니다.

```bash
sudo yum install yum-utils
```

7. CUDA 툴킷의 드라이버 저장소를 선택하고 VM에 추가합니다.

```bash
sudo yum-config-manager --add-repo https://developer.download.nvidia.com/compute/cuda/repos/rhel8/x86_64/cuda-rhel8.repo
```

8. Yum 캐시를 삭제합니다.

```bash
sudo yum clean all
```

9. CUDA를 설치합니다.

```bash
sudo dnf -y module install nvidia-driver:latest-dkms
sudo dnf -y install cuda
```

10. NVIDIA 드라이버를 설치합니다. 이 명령어는 CUDA 11을 설치합니다.

```bash
sudo yum -y install cuda-drivers
```

11. 드라이버 설치 단계를 완료한 후 드라이버가 올바르게 설치 및 초기화되었는지 확인합니다.

```bash
sudo nvidia-smi
```

위의 명령어를 실행하면 다음과 비슷한 출력이 출력될 것입니다.

<img width="625" src="https://user-images.githubusercontent.com/71860179/111944716-a1364b80-8b1b-11eb-93f0-5435f79603b0.png">



### 6. install nvidia-docker

1. 필요한 tool들 설치해 줍니다.

```bash
sudo dnf install -y tar bzip2 make automake gcc gcc-c++ vim pciutils elfutils-libelf-devel libglvnd-devel iptables
```

2. Docker CE repository 셋업합니다.

```bash
sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
```

3. install `containerd.io` package

```bash
sudo dnf install -y https://download.docker.com/linux/centos/7/x86_64/stable/Packages/containerd.io-1.4.3-3.1.el7.x86_64.rpm
```

4. install the latest `docker-ce` package

```bash
sudo dnf install docker-ce -y
```

5. Ensure the Docker sevice is running with the following command

```bash
sudo systemctl --now enable docker
```

6. test your Docker installation by running the `hello-world` container

```bash
sudo docker run --rm hello-world
```

output will be simliar to picture below.

<img width="652" alt="hello_docker" src="https://user-images.githubusercontent.com/71860179/111946816-9c739680-8b1f-11eb-941d-a92b672d8d6b.png" style="zoom:100%;" >



### 7. Setting up NVIDIA Container Toolkit

1. Setup the `stable` repository and the GPG key

```bash
distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
   && curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.repo | sudo tee /etc/yum.repos.d/nvidia-docker.repo
```

2. install the nvidia-docker2 package after updating the package listing

```bash
sudo dnf clean expire-cache --refresh
```

```bash
sudo dnf install -y nvidia-docker2
```

3. restart the Docker daemon to complete the installation after setting the default runtime

```bash
sudo systemctl restart docker
```

4. Test working setup by running a base CUDA container

```bash
sudo docker run --rm --gpus all nvidia/cuda:11.0-base nvidia-smi
```

output will be simliar to picture below.

<img width="739" alt="docker_nvidia_smi" src="https://user-images.githubusercontent.com/71860179/111947415-caa5a600-8b20-11eb-9339-13e9aed24d73.png">



### 8. Check GPU on python 

Tensorflow Docker 이미지를 설치후, shell에 접속해서 GPU 세팅이 되어 있는지 확인해 보도록 하겠습니다. 

```bash
docker run --gpus all -it --rm -p 8888:8888 tensorflow/tensorflow:latest-gpu-py3-jupyter bash
```

위 명령어를 실행하면 텐서플로우 Docker 이미지 설치후, Docker 컨테이너 shell에 접속할 수 있습니다.

이후 Docker shell 내부에서 `nvidia-smi` 명령어를 실행해 보면 다음과 같이 GPU 두개가 잡혀 있는 것을 확인 할 수 있습니다.

<img width="893" src="https://user-images.githubusercontent.com/71860179/111947773-6df6bb00-8b21-11eb-8f34-cf465d064551.png">



이후 Docker shell 내부에서 Python shell을 실행 후, GPU 인식 상태를 확인해 보도록 하겠습니다.

```python
from tensorflow.python.client import device_lib
device_lib.list_local_devices()
```

이후 다음과 같은 결과를 확인 할 수 있습니다. 

<img width="891" src="https://user-images.githubusercontent.com/71860179/111950499-b57f4600-8b25-11eb-84e0-bb1deee2fedf.png">


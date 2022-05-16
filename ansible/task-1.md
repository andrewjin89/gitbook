---
description: ansible awx 과 git 연동
---

{% hint style="info" %}
- Ansible: <https://www.ansible.com/>
- AWX: <https://github.com/ansible/awx>
{% endhint %}

## ansible&AWX 사용 목적
- 인프라운영 업무의 단순/반복적인 업무를 자동화하여 효율적인 업무 수행을 위해
- ansible은 UI환경이 제공되지 않아 OpenSource 프로젝트인 AWX를 통해 UI환경 도임

## Ansible
- python기반의 오픈소스 IT자동화 도구
- ssh를 이용해서 다수의 호스트 자동화 구성 관리 도구
- Agent 방식은 지원하지 않음
- yaml구문으로 playbook을 작성하여 환경 구성.

## AWX
- ansible에 없는 UI지원 오픈소스
- http API를 AWS CLI로 지원

## Ansible 특징
- 멱등성
- agentless

## Ansible 용어/개녕
- Control node
  - Ansible 설치된 시스템 / 다수의 호스트에 명령을 내리는 시스템
- Managed nodes
  - Control node의 통제를 받는 호스트
- playbook
  - 자동화 작업 리스트
  - YAML로 작성한다.
- inventory
  - Managed node 리스트
  - 동적 생성 가능
- task
  - 단일 작업
- projects
  - playbook의 파일 위치 / git으로 관리되는 repository

## Ansible & AWX 설치
> ReferenceURL: <https://meetup.toast.com/posts/258>

### python 3 설치

```cli
sudo yum install gcc zlib-devel openssl openssl-devel libffi-devel -y
wget https://www.python.org/ftp/python/3.6.8/Python-3.6.8.tar.xz
tar -xvf Python-3.6.8.tar.xz
cd Python-3.6.8
mkdir -p /home/ansible/package
./configure --prefix=/home/ansible/package/Python-3.6.8
make && make install
```

### pip 설치

```cli
wget https://bootstrap.pypa.io/get-pip.py
/home/ansible/package/Python-3.6.8/bin/python3 ./get-pip.py
```

### virtualenv 설치. (option)
> 필수 진행 항목은 아니며, python 패키지의 별도 관리를 위해 진행

```cli
cd /home/ansible/package/Python-3.6.8/bin
./pip3 install virtualenv
./pip3 install virtualenvwrapper

vi ~/.bashrc
export VIRTUALENVWRAPPER_PYTHON=/home/ansible/package/Python-3.6.8/bin/python3
source /home/ansible/package/Python-3.6.8/bin/virtualenvwrapper.sh

vi ~/.bash_profile
PATH=$HOME/package/Python-3.6.8/bin:$PATH:$HOME/bin
export WORKON_HOME=$HOME/.virtualenvs
export PATH

source ~/.bash_profile
mkvirtualenv -p /home/ansible/package/Python-3.6.8/bin/python3 ansible
```

### ansible 설치

```cli
pip install ansible
```

### workon

```cli
# 가상환경 ansible 접속
workon ansible

# 가상환경 ansible 종료
deactivate
```

### AWX 설치 및 실행

```cli
# docker

sudo yum install -y yum-utils device-mapper-persistent-data lvm2
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install -y docker-ce docker-ce-cli containerd.io
# docker compose

sudo curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
docker-compose --version
# docker 구동 및 등록

sudo systemctl start docker
sudo systemctl enable docker
# root 외에 계정도 docker 를 이용할 수 있도록 usermod 변경.
sudo usermod -a -G docker ansible

# ansible 계정으로 재 접속 (꼭 재접속) 후, 테스트
docker ps
# python docker / docker-compose

pip install docker
pip install docker-compose
# git

sudo yum install -y curl-devel expat-devel gettext-devel perl-ExtUtils-MakeMaker
cd ~/downloads
wget https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.22.0.tar.xz
tar -xvf git-2.22.0.tar.xz
cd git-2.22.0
make prefix=/home/ansible/package/git all
make prefix=/home/ansible/package/git install
# vi ~/.bash_profile

PATH=$HOME/package/git/bin:$PATH
export PATH
# node.js (LTS 10)

curl -sL https://rpm.nodesource.com/setup_10.x | sudo bash -
sudo yum install -y nodejs
# https://github.com/ansible/awx/releases

cd ~/downloads
wget https://github.com/ansible/awx/archive/15.0.0.tar.gz
tar -xvf 15.0.0.tar.gz
ln -s awx-15.0.0 awx

# 혹은
# use github
cd ~/downloads
git clone https://github.com/ansible/awx.git awx_git
ln -s awx_git awx
# vi /home/ansible/downloads/awx/installer/inventory
# 아래 항목들을 찾아 주석을 제거 하거나, 값을 변경해 줍니다.
# use_docker_compose=true 의 경우, 추가.

localhost ansible_connection=local ansible_python_interpreter="/home/ansible/.virtualenvs/ansible/bin/python"
postgres_data_dir=/home/ansible/package/awx/pgdocker
host_port=10080
host_port_ssl=10081
use_docker_compose=true
docker_compose_dir=/home/ansible/package/awx/awxcompose
project_data_dir=/home/ansible/package/awx/projects
# awx 최초 실행 (docker-compose.yml 파일 생성)
# 이때, "Create Docker Compose Configuration" 에서 libselinux-python 에러가 날 경우, selinux 설정을 disabled 시키고 해볼 것.

cd /home/ansible/downloads/awx/installer
/home/ansible/.virtualenvs/ansible/bin/ansible-playbook -i inventory install.yml
# awx 중지 / 시작

cd /home/ansible/package/awx/awxcompose
docker-compose stop
docker-compose start
# awx 로그 확인
docker logs -f awx_task

# docker process 확인
# awx_task, awx_web, awx_postgres, awx_redis 가 동작하고 있어야 함.
docker ps
# awx 접속
# 초기 비번 : admin / password

http://{서버IP}:10080
```

## ansible 설치

```cli
yum -y install epel-release git gcc yum-utils
yum -y install ansible python-pip
```

## AWX 설치

```cli

yum-config-manager --add-repo  https://download.docker.com/linux/centos/docker-ce.repo
yum -y install docker-ce

pip install -U pip docker-compose==1.23.2

systemctl enable docker
systemctl start docker

git clone -b 21.0.0 https://github.com/Ansible/awx.git
make docker-compose-build


pip3 install awxkit
pip3 install sphinx sphinxcontrib-autoprogram
cd awxkit/awxkit/cli/docs
TOWER_HOST=http://180.70.98.134 TOWER_USERNAME=root TOWER_PASSWORD=root make clean html
```

## AWX CLI 설치

```cli

awx --help
```

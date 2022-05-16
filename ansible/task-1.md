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

## ansible 설치

```bash
yum -y install epel-release git gcc yum-utils
yum -y install ansible python-pip
```

## AWX 설치

```bash

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

```bash

awx --help
```

## ansible on azure 


1. azure centos 기준으로 ansible 과 plugin 설치 방법 

```
#!/bin/bash

# Update all packages that have available updates.
sudo yum update -y

# Install Python 3 and pip.
sudo yum install -y python3-pip

# Upgrade pip3.
sudo pip3 install --upgrade pip

# Install Ansible.
pip3 install "ansible==2.9.17"

# Install Ansible azure_rm module for interacting with Azure.
pip3 install ansible[azure]
```

2. 클리덴셜 등록 

```
크리덴션을 아래와 같이 만든다.
mkdir ~/.azure
vi ~/.azure/credentials


[default]
subscription_id=<subscription_id>
client_id=<service_principal_app_id>
secret=<service_principal_password>
tenant=<service_principal_tenant_id>

혹은 환경변수에 등록 하면 된다. 
export AZURE_SUBSCRIPTION_ID=<subscription_id>
export AZURE_CLIENT_ID=<service_principal_app_id>
export AZURE_SECRET=<service_principal_password>
export AZURE_TENANT=<service_principal_tenant_id>

logan-webapp

<subscription_id> 포탈에서 보면 된다.
31f63209-7e06-4251-a32b-60980c77ae7e

<service_principal_app_id> 
4ee93bf1-37b9-4f2a-a322-ab5868bca3dd

<service_principal_tenant_id>
9ab80bfb-f6a5-4b78-9a6f-dd037dc0e745

<service_principal_password>
FrB8Q~1HRDgbcanmpE.poR6afabkQ5EMBUXETcm5



```

3. 동적 인벤토리 구성 
```
plugin: azure_rm
include_vm_resource_groups:
  - logan-rg-005
auth_source: auto
conditional_groups:
  linux: "'cent' in image.offer"
  windows: "'WindowsServer' in image.offer
```

4. winrmtest
```
---
- hosts: windows
  gather_facts: false

  vars_prompt:
    - name: username
      prompt: "Enter local username"
      private: false
    - name: password
      prompt: "Enter password"

  vars:
    ansible_user: "{{ username }}"
    ansible_password: "{{ password }}"
    ansible_port: 5985
    ansible_connection: winrm
    ansible_winrm_transport: basic
    ansible_winrm_server_cert_validation: ignore

  tasks:

  - name: Test connection
    win_command: "hostname"

  - name: Define when to enable Verbose/Debug outputh
    win_shell: get-service
    register: "getservice"

  - name: debug result
    debug:
      msg: "{{getservice.stdout_lines}}"

```


5. 제약사항 
nsg 에서 5985 허용
windows 베이직 크리데셜과 암호화안된 트래픽 허용 
이상하게도 logan  최초 관리자 계정으로는 reject 됨 
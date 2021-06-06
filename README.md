### OpenStack install with DPDK

Links
-----

https://docs.openstack.org/kolla-ansible/rocky/reference/networking-guide.html

https://docs.openstack.org/tacker/latest/install/kolla.html

Turn off biosdevname and udev
-----

```
vi /etc/default/grub
biosdevname=0 net.ifnames=0
```

Trouble Shooting with Kolla
-----

**libvirtd issue:**
-> removed the libvirt-lock file

INSTRUCTION
----

- 노트북/데스크탑 CentOS7/8버전 설치

앤서블 기반으로 랩 구성을 하려면 GITHUB에서 **build-lab**에서 플레이북을 실행 합니다. 플레이북 실행은 해당 디렉터리에 들어가면 나옵니다. 이 플레이북은 Red hat계열에서 테스트가 되었습니다.

설치전 올바른 테스트를 하기 위해서 Host컴퓨터에서 아래와 같이 설정 및 구성을 합니다.

- libvirt기반으로 가상머신 1대 구성
  
  ```
  yum group install "Virtualization Hosts" -y
  yum install virt-manager
  vi /etc/modprobe.d/nested.conf
     options kvm_intel nested=Y
  ```

- 가상머신을 생성합니다. 플레이북을 사용하면 자동으로 가상머신을 구성 및 생성 합니다. 아래 내용은 수동으로 구성 시 아래처럼 구성 합니다.
  
  ```
  vcpu: 4~8
  vmem: least 8GiB, 16GiB max
  disk: least 30GiB, 100Gib recommend
  cpu: hosted-passthrough
  nic1: default, e1000, default
  nic2: default, e1000, internal
  ```
1. 가상머신을 CentOS 8버전으로 설치한다. (플레이북 사용 시 넘어감)

2. 모든 패키지 업데이트 수행. (플레이북 사용 시 넘어감)
   
   ```
   - yum update -y && reboot
   ```

3. 아이피 설정 eth0,eth1. (플레이북 사용 시 넘어감)
   
   ```
   - eth0: 192.168.122.110
   - eth1: 192.168.90.110
   ```

4. Lab에서 필요없는 서비스 중지. (플레이북 사용 시 넘어감)
   
   - systemctl stop firewalld && systemctl disable firewalld && systemctl status firewalld

5. A recode대신, /etc/hosts에 다음과 같은 정보 입력. (플레이북 사용 시 넘어감)
   
   ```textile
   # vi /etc/hosts
   192.168.122.110 allinone.example.com allinone
   ```

위의 작업이 완료가 되면 가상머신에서 다음과 같은 작업을 수행 합니다.

```bash
# git clone https://github.com/openstack/kolla
# git clone https://github.com/openstack/kolla-ansible
```

github에서 콜라에서 앤서블 플레이북을 내려받기 합니다. 내렵받기 후 콜라를 사용 할 수 있도록 환경을 준비 합니다. 이번 구성에는 venv(virtual enviroment)는 사용하지 않습니다. 

콜라에서 플레이북을 사용할 수 있도록 준비 합니다.

```bash
# cd kolla
# pip3 install .
# kolla-ansible
# pip3 install .
# cp -r kolla-ansible/etc/kolla /etc/kolla
# cp kolla-ansible/ansible/inventory/* /etc/kolla/
# ls /etc/kolla/
all-in-one  globals.yml  multinode  passwords.yml
```

이번 랩에서는 "multinode"를 사용하지 않고, "all-in-one"파일을 사용합니다. 그 이유는 자원이 넉넉하지 않기 때문에 입니다. 자원이 넉넉하신 경우 "multinode"를 사용하셔도 됩니다.

콜라 gloabals에 다음처럼 수정 및 변경 합니다. all-in-one위한 설정이며, 실제 **실무환경(production)** 으로는 적합하지 않습니다.

```bash
# vi /etc/kolla/gloabals.yml 
---
kolla_base_distro: "centos"
kolla_install_type: "source"
kolla_internal_vip_address: "192.168.122.250"
network_interface: "enp1s0"
neutron_external_interface: "enp1s0"
enable_openvswitch: "{{ enable_neutron | bool and neutron_plugin_agent != 'linuxbridge' }}"
kuryr_install_type: source
```

콜라에서 사용할 서비스 비밀번호를 생성 합니다. 생성시 사용할 ssh공개 및 비공개키를 생성 합니다.

```bash
# kolla-genpwd
# vi /etc/kolla/passwords.yml
    - keystone_admin_password: openstack
# ssh-keygen -t rsa -N '' 
# ssh-copy-id root@node
```

all-in-one설정에서 필요 없는 설정 내용을 인벤토리(inventory)에서 제거 합니다.

```bash
# vi /etc/kolla/all-in-one
    :%s/localhost/node/g
    :%s/ansible_connection=local//g
```

위의 기본적인 구성이 완료가 되면 부트-스트랩핑 및 환경 검사후 설치를 진행 합니다.

```bash
# kolla-ansible -i all-in-one bootstrap-servers
# kolla-ansible -i /etc/kolla/all-in-one prechecks
# kolla-ansible -i /etc/kolla/all-in-one pull
# kolla-ansible -i /etc/kolla/all-in-one deploy
# kolla-ansible -i /etc/kolla/all-in-one post-deploy
```

```bash
# cat /etc/kolla/admin-openrc.sh
```

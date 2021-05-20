OpenStack install with DPDK


Links
-----
https://docs.openstack.org/kolla-ansible/rocky/reference/networking-guide.html

https://docs.openstack.org/tacker/latest/install/kolla.html


turn off biosdevname
-----
```
vi /etc/default/grub
biosdevname=0 net.ifnames=0
```


T/S
-----
libvirtd issue:
-> removed the libvirt-lock file

- 노트북에 CentOS7/8버전 설치
- libvirt기반으로 가상머신 1대 구성
  * yum group install "Virtualization Hosts" -y
  * yum install virt-manager
  * vi /etc/modprobe.d/nested.conf
    options kvm_intel nested=Y
    
- 가상머신을 생성한다
  * vcpu: 4~8
  * vmem: least 8GiB, 16GiB max
  * disk: least 30GiB, 100Gib recommend
  * cpu: hosted-passthrough
  * nic1: default, e1000, default
  * nic2: default, e1000, internal
  
1. 가상머신을 CentOS 8버전으로 설치한다. 
2. 모든 패키지 업데이트 수행
  - yum update -y && reboot
3. 아이피 설정 eth0,eth1 
  - eth0: 192.168.122.110
  - eth1: 192.168.90.110

4. Lab에서 필요없는 서비스 중지
  - systemctl stop firewalld && systemctl disable firewalld && systemctl status firewalld
5. A recode대신, /etc/hosts에 다음과 같은 정보 입력
  - vi /etc/hosts
    192.168.122.110 node.example.com node
6. yum install git 
7. git clone https://github.com/openstack/kolla 
8. git clone https://github.com/openstack/kolla-ansible
9. cd kolla
10. pip3 install .
11. cd  kolla-ansible
12. pip3 install .
13. cp -r kolla-ansible/etc/kolla /etc/kolla
14. cp kolla-ansible/ansible/inventory/* /etc/kolla/
15. ls /etc/kolla/
all-in-one  globals.yml  multinode  passwords.yml
16. vi /etc/kolla/gloabals.yml 
---
```
kolla_base_distro: "centos"
kolla_install_type: "source"
kolla_internal_vip_address: "192.168.122.250"
network_interface: "enp1s0"
neutron_external_interface: "enp1s0"
enable_openvswitch: "{{ enable_neutron | bool and neutron_plugin_agent != 'linuxbridge' }}"
kuryr_install_type: source
```
17. kolla-genpwd
18. vi /etc/kolla/passwords.yml
  - keystone_admin_password: openstack
19. ssh-keygen -t rsa -N '' 
20. ssh-copy-id root@node
21. vi /etc/kolla/all-in-one
```
:%s/localhost/node/g
:%s/ansible_connection=local//g
```

22. kolla-ansible -i all-in-one bootstrap-servers
23. kolla-ansible -i /etc/kolla/all-in-one prechecks
24. kolla-ansible -i /etc/kolla/all-in-one pull
25. kolla-ansible -i /etc/kolla/all-in-one deploy
26. kolla-ansible -i /etc/kolla/all-in-one post-deploy
27. cat /etc/kolla/admin-openrc.sh

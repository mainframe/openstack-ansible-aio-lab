# Table of Contents

* [Build and use an OpenStack\-Ansible All\-in\-One lab](#build-and-use-an-openstack-ansible-all-in-one-lab)
  * [Install the KVM virtualized OpenStack\-Ansible All\-in\-One guest on an Ubuntu 16\.04 LTS desktop host](#install-the-kvm-virtualized-openstack-ansible-all-in-one-guest-on-an-ubuntu-1604-lts-desktop-host)
    * [Install KVM on an Ubuntu 16\.04 LTS desktop](#install-kvm-on-an-ubuntu-1604-lts-desktop)
    * [Reserve IP address on a KVM virtualization host for the osaaioubuntu01 KVM VM](#reserve-ip-address-on-a-kvm-virtualization-host-for-the-osaaioubuntu01-kvm-vm)
    * [Create the OSA AiO KVM VM on an Ubuntu 16\.04 LTS desktop](#create-the-osa-aio-kvm-vm-on-an-ubuntu-1604-lts-desktop)
    * [Enable nested virtualization on a KVM virtualization host for the osaaioubuntu01 KVM VM](#enable-nested-virtualization-on-a-kvm-virtualization-host-for-the-osaaioubuntu01-kvm-vm)
    * [Install OpenStack\-Ansible All\-in\-One](#install-openstack-ansible-all-in-one)
  * [Install OpenStack\-Ansible All\-in\-One on the FUJITSU PRIMERGY RX300 S8 (ABN:K1457\-V101\-724)](#install-openstack-ansible-all-in-one-on-the-fujitsu-primergy-rx300-s8-abnk1457-v101-724)
    * [Hardware](#hardware)
    * [Network Addresses](#network-addresses)
    * [Initial FUJITSY PRIMERGY RX300 S8 setup via IPMI console](#initial-fujitsy-primergy-rx300-s8-setup-via-ipmi-console)
    * [Over the SSH](#over-the-ssh)
* [ETAIS images and flavors setup](#etais-images-and-flavors-setup)
  * [Verify](#verify)
  * [Create VM Flavors in OpenStack](#create-vm-flavors-in-openstack)
    * [Create the script for adding flavors](#create-the-script-for-adding-flavors)
    * [Create flavors](#create-flavors)
    * [Verify](#verify-1)
* [HOWTOs, tips and tricks](#howtos-tips-and-tricks)
  * [Find Horizon admin user password](#find-horizon-admin-user-password)
  * [Use Openstack\-Ansible openstack CLI](#use-openstack-ansible-openstack-cli)
  * [Use network namespaces](#use-network-namespaces)
  * [Restart OpenStack\-Ansible](#restart-openstack-ansible)
  * [Customize OpenStack\-Ansible](#customize-openstack-ansible)
  * [Set OpenStack\-Ansible project scope](#set-openstack-ansible-project-scope)
  * [Set permanent DNS servers on CentOS instance](#set-permanent-dns-servers-on-centos-instance)
* [OpenStack\-Ansible documentation](#openstack-ansible-documentation)

# Build and use an OpenStack-Ansible All-in-One lab


## Install the KVM virtualized OpenStack-Ansible All-in-One guest on an Ubuntu 16.04 LTS desktop host


### Install KVM on an Ubuntu 16.04 LTS desktop

    sudo apt install -y qemu-kvm \
      libvirt-bin \
      cloud-utils \
      virtinst

### Reserve IP address on a KVM virtualization host for the osaaioubuntu01 KVM VM

    virsh net-update default \
      add ip-dhcp-host \
      "<host mac='52:54:00:99:1b:ae' name='osaaioubuntu01' \
      ip='192.168.122.100' />" \
      --live --config

### Create the OSA AiO KVM VM on an Ubuntu 16.04 LTS desktop

    cloudimgdom='https://cloud-images.ubuntu.com'
    release='bionic'
    cloudimg='bionic-server-cloudimg-amd64.img'
    sudo wget "$cloudimgdom/$release/current/$cloudimg" \
      -O "/var/lib/libvirt/images/$cloudimg"
    sudo qemu-img create \
      -f qcow2 \
      -b /var/lib/libvirt/images/$cloudimg \
      /var/lib/libvirt/images/osaaioubuntu01.qcow2 100G
    cat << EOF > /tmp/osaaioubuntu01_cidata
    #cloud-config
    password: password
    chpasswd: { expire: False }
    ssh_pwauth: True
    hostname: osaaioubuntu01
    EOF
    sudo cloud-localds /var/lib/libvirt/images/cidata_osaaioubuntu01.img /tmp/osaaioubuntu01_cidata
    virt-install --import --name osaaioubuntu01 \
      --ram 12000 \
      --vcpus 8 \
      --disk /var/lib/libvirt/images/osaaioubuntu01.qcow2 \
      --disk /var/lib/libvirt/images/cidata_osaaioubuntu01.img,device=cdrom \
      --network bridge=virbr0,mac=52:54:00:99:1b:ae \
      --graphics vnc,listen=127.0.0.1 \
      --noautoconsole \
      --import \
      --hvm

### Enable nested virtualization on a KVM virtualization host for the osaaioubuntu01 KVM VM

    # change "cpu mode" like follows
    # <cpu mode='host-passthrough'/>
    $ virsh edit osaaioubuntu01

### Install OpenStack-Ansible All-in-One

    ssh-copy-id ubuntu@192.168.122.100
    ssh ubuntu@192.168.122.100
      sudo apt update -y && sudo apt upgrade -y
        exit
    virsh shutdown osaaioubuntu01
    virsh snapshot-create-as osaaioubuntu01 after_update_and_upgade
    virsh start osaaioubuntu01
    ssh ubuntu@192.168.122.100
      sudo su -
        git clone https://git.openstack.org/openstack/openstack-ansible \
          /opt/openstack-ansible
        cd /opt/openstack-ansible
        git tag -l
        #Checkout latest stable version
        git checkout 18.1.0
          exit
            exit
    virsh shutdown osaaioubuntu01
    virsh snapshot-create-as osaaioubuntu01 after_osa_latest_stable_checkout
    virsh start osaaioubuntu01
    ssh ubuntu@192.168.122.100
      sudo su -
        export ANSIBLE_ROLE_FETCH_MODE=git-clone
        cd /opt/openstack-ansible
        ./scripts/bootstrap-ansible.sh
          exit
            exit
    virsh shutdown osaaioubuntu01
    virsh snapshot-create-as osaaioubuntu01 after_bootstrap_ansible
    virsh start osaaioubuntu01
    ssh ubuntu@192.168.122.100
      sudo su -
        export ANSIBLE_ROLE_FETCH_MODE=git-clone
        cd /opt/openstack-ansible
        ./scripts/bootstrap-aio.sh
          exit
            exit
    virsh shutdown osaaioubuntu01
    virsh snapshot-create-as osaaioubuntu01 after_bootstrap_aio
    virsh start osaaioubuntu01
    ssh ubuntu@192.168.122.100
      sudo su -
        export ANSIBLE_ROLE_FETCH_MODE=git-clone
        cd /opt/openstack-ansible/playbooks/
        openstack-ansible setup-hosts.yml
          exit
            exit
    virsh shutdown osaaioubuntu01
    virsh snapshot-create-as osaaioubuntu01 after_setup_hosts
    virsh start osaaioubuntu01
    ssh ubuntu@192.168.122.100
      sudo su -
        export ANSIBLE_ROLE_FETCH_MODE=git-clone
        cd /opt/openstack-ansible/playbooks/
        openstack-ansible setup-infrastructure.yml
          exit
            exit
    virsh shutdown osaaioubuntu01
    virsh snapshot-create-as osaaioubuntu01 after_setup_infrastructure
    virsh start osaaioubuntu01
    ssh ubuntu@192.168.122.100
      sudo su -
        export ANSIBLE_ROLE_FETCH_MODE=git-clone
        cd /opt/openstack-ansible/playbooks/
        openstack-ansible setup-openstack.yml
          exit
            exit
    virsh shutdown osaaioubuntu01
    virsh snapshot-create-as osaaioubuntu01 after_setup_openstack

## Install OpenStack-Ansible All-in-One on the FUJITSU PRIMERGY RX300 S8 (ABN:K1457-V101-724)

### Hardware

Hostname | Model | CPU cores | Memory | Disk | Network
---------|-------|-----------|--------|------|--------
osa | FUJITSY PRIMERGY RX300 S8 | 12 (< 120 vCore) | 128 | 2 x 558 | 2 x Gbit/s and 2 x 10 Gbit/s (SFP+)

### Network Addresses


Hostname | MAC | IP 
---------|-----|----
osa | 90:1b:0e:0b:75:8b | 193.40.248.32/24 

### Initial FUJITSY PRIMERGY RX300 S8 setup via IPMI console

FUJITSY PRIMERGY RX300 S8 IPMI console requires Java 8 Update 121 (http://www.oracle.com/technetwork/java/javase/downloads/java-archive-javase8-2177648.html)

    sudo su -
      ip addr del 193.40.248.32/24 dev eno1
      ip link set eno1 master br-vlan
      ip addr add 193.40.248.32/24 dev br-vlan
      ip addr del 172.29.248.100/22 dev br-vlan
      ip route add default via 193.40.248.254

### Over the SSH

    sudo su -
      lxc-attach -n $(sudo lxc-ls aio1_utility_container)
        source /root/openrc
        openstack port \
          list -f yaml
        - Fixed IP Addresses: ip_address='192.168.74.2', subnet_id='67796d68-a687-4e69-9bc2-fef39956b232'
          ID: 015c5a78-4273-4f22-8af2-5658ad612ad9
          MAC Address: fa:16:3e:44:c8:ab
          Name: ''
          Status: ACTIVE
        - Fixed IP Addresses: ip_address='192.168.74.1', subnet_id='67796d68-a687-4e69-9bc2-fef39956b232'
          ID: 67a6bd7a-ab21-4246-870b-210b4e8c55e3
          MAC Address: fa:16:3e:43:46:cf
          Name: ''
          Status: ACTIVE
        - Fixed IP Addresses: ip_address='172.29.249.113', subnet_id='ec7ceb4c-813c-4a45-a9e9-52b87bae99b5'
          ID: a43e8192-9d59-4f17-a8a7-f852df2263f8
          MAC Address: fa:16:3e:45:ab:56
          Name: ''
          Status: ACTIVE
        - Fixed IP Addresses: ip_address='172.29.249.110', subnet_id='ec7ceb4c-813c-4a45-a9e9-52b87bae99b5'
          ID: d00ced03-674f-4653-9a3d-6659d9e7d0fd
          MAC Address: fa:16:3e:52:4a:52
          Name: ''
          Status: ACTIVE
        openstack port \
          delete d00ced03-674f-4653-9a3d-6659d9e7d0fd
        openstack router \
          unset router --external-gateway
        openstack port \
          list -f yaml
        - Fixed IP Addresses: ip_address='192.168.74.2', subnet_id='67796d68-a687-4e69-9bc2-fef39956b232'
          ID: 015c5a78-4273-4f22-8af2-5658ad612ad9
          MAC Address: fa:16:3e:44:c8:ab
          Name: ''
          Status: ACTIVE
        - Fixed IP Addresses: ip_address='172.29.249.110', subnet_id='ec7ceb4c-813c-4a45-a9e9-52b87bae99b5'
          ID: 643cbf44-f402-4d4e-8b9a-4776d08e2589
          MAC Address: fa:16:3e:57:1d:57
          Name: ''
          Status: ACTIVE
        - Fixed IP Addresses: ip_address='192.168.74.1', subnet_id='67796d68-a687-4e69-9bc2-fef39956b232'
          ID: 67a6bd7a-ab21-4246-870b-210b4e8c55e3
          MAC Address: fa:16:3e:43:46:cf
          Name: ''
          Status: ACTIVE
        openstack port \
          delete 643cbf44-f402-4d4e-8b9a-4776d08e2589
        openstack network \
          delete public
        openstack network \
          create --provider-physical-network flat \
            --provider-network-type flat public --project demo --external
        openstack subnet \
          create --network public \
            --subnet-range 193.40.248.32/28 --gateway 193.40.248.254 --no-dhcp public-subnet
        openstack router set router --external-gateway public
        cat << EOF >> /etc/openstack_deploy/user_variables.yml
        
        
        # User-provided certificates for HAProxy
        haproxy_user_ssl_cert: /etc/openstack_deploy/ssl/osa_ttu_ee.crt
        haproxy_user_ssl_key: /etc/openstack_deploy/ssl/osa_ttu_ee.key
        haproxy_user_ssl_ca_cert: /etc/openstack_deploy/ssl/DigiCertCA.crt
        EOF
        cd /opt/openstack-ansible/playbooks/
        openstack-ansible haproxy-install.yml

# ETAIS images and flavors setup

http://images.etais.ee

    sudo lxc-attach -n $(sudo lxc-ls aio1_utility_container)
      source /root/openrc
      read images_etais_username
      read images_etais_password
        IMAGENAME="centos7-minimal-1711"
          curl -u ${images_etais_username}:${images_etais_password} \
            -O http://images.opnd.org/${IMAGENAME}.qcow2
          openstack image create "CentOS 7 x86_64" \
            --disk-format qcow2 \
            --min-disk 8 --min-ram 256 \
            --file ${IMAGENAME}.qcow2 \
            --public
        IMAGENAME="centos7-docker-1711"
          curl -u ${images_etais_username}:${images_etais_password} \
            -O http://images.opnd.org/${IMAGENAME}.qcow2
          openstack image create "CentOS 7 Docker Host x86_64" \
            --disk-format qcow2 \
            --min-disk 8 --min-ram 512 \
            --file ${IMAGENAME}.qcow2 \
            --public
        IMAGENAME="ubuntu1604-minimal-20171221"
          curl -u ${images_etais_username}:${images_etais_password} \
            -O http://images.opnd.org/${IMAGENAME}.qcow2
          openstack image create "Ubuntu 16.04 x86_64" \
            --disk-format qcow2 \
            --min-disk 8 --min-ram 512 \
            --file ${IMAGENAME}.qcow2 \
            --public
        IMAGENAME="debian8-minimal-20171223"
          curl -u ${images_etais_username}:${images_etais_password} \
            -O http://images.opnd.org/${IMAGENAME}.qcow2
          openstack image create "Debian 8 x86_64" \
            --disk-format qcow2 \
            --min-disk 8 --min-ram 256 \
            --file ${IMAGENAME}.qcow2 \
            --public
        IMAGENAME="debian9-minimal-20171224"
          curl -u ${images_etais_username}:${images_etais_password} \
            -O http://images.opnd.org/${IMAGENAME}.qcow2
          openstack image create "Debian 9 x86_64" \
            --disk-format qcow2 \
            --min-disk 8 --min-ram 256 \
            --file ${IMAGENAME}.qcow2 \
            --public
    
## Verify

    openstack image list


## Create VM Flavors in OpenStack

###  Create the script for adding flavors

     cat << 'EOF' > add-flavor
     #!/bin/bash
     FLAVOR_NAME=$1
     RAM_IN_MB=$2
     ROOT_DISK_IN_GB=$3
     NUMBER_OF_VCPUS=$4
     
     openstack flavor create \
       $FLAVOR_NAME --id auto \
         --ram $RAM_IN_MB --disk $ROOT_DISK_IN_GB --vcpus $NUMBER_OF_VCPUS
     exit 0
     EOF
     chmod 700 add-flavor

### Create flavors

    source /root/openrc
    ./add-flavor m1.xsmall 1024 10 1
    ./add-flavor m1.small 2048 20 1
    ./add-flavor c1.small 2048 20 2
    ./add-flavor m1.medium 4096 20 2
    ./add-flavor c1.medium 4096 20 4
    ./add-flavor m1.large 8192 20 4
    ./add-flavor c1.large 8192 20 8
    ./add-flavor m1.xlarge 16384 20 8
    ./add-flavor c1.xlarge 16384 20 16
    ./add-flavor m1.xxlarge 32768 20 16

### Verify

    openstack flavor list
      
# HOWTOs, tips and tricks 

## Find Horizon admin user password

    sudo grep keystone_auth_admin_password /etc/openstack_deploy/user_secrets.yml
    keystone_auth_admin_password: 2231d8957a78d1804c827ba9b5d5cffc4f142370c39e649deba26c2c8e1

## Use Openstack-Ansible openstack CLI

    sudo lxc-attach -n $(sudo lxc-ls aio1_utility_container)
      source /root/openrc

## Use network namespaces

    sudo lxc-attach -n $(sudo lxc-ls aio1_neutron_agents_container)
    root@aio1-neutron-agents-container-32670885:/# ip netns
    qrouter-ca9fc9af-c538-4d90-a800-66bc0d560ec1 (id: 3)
    qdhcp-ccf975fd-463b-4b27-9949-9abae843e32d (id: 2)
    qdhcp-88f7f240-0895-4050-80fb-605e51f98753 (id: 1)
    root@aio1-neutron-agents-container-32670885:/# \
      ip netns exec qrouter-ca9fc9af-c538-4d90-a800-66bc0d560ec1 \
      ping 8.8.8.8
    PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
    64 bytes from 8.8.8.8: icmp_seq=1 ttl=57 time=8.16 ms
    64 bytes from 8.8.8.8: icmp_seq=2 ttl=57 time=7.87 ms
    64 bytes from 8.8.8.8: icmp_seq=3 ttl=57 time=7.86 ms
    ^C
    --- 8.8.8.8 ping statistics ---

## Restart OpenStack-Ansible

As the OSA AiO includes all three cluster members of MariaDB/Galera, the cluster has to be re-initialized after the host is rebooted.

    sudo su -
      cd /opt/openstack-ansible/playbooks
      openstack-ansible -e galera_ignore_cluster_state=true galera-install.yml

## Customize OpenStack-Ansible

    sudo su -
      cat << EOF >> /etc/openstack_deploy/user_variables.yml 
      
      # Set  TMOUT in seconds for sessions
      security_rhel7_session_timeout: 28800
      EOF
      cd /opt/openstack-ansible/playbooks/
      openstack-ansible security-hardening.yml 

## Set OpenStack-Ansible project scope

    sudo lxc-attach -n $(sudo lxc-ls aio1_utility_container)
    cp -i /root/openrc /root/openrc_TTUtunniplaan
    sed -i 's/export OS_PROJECT_NAME=admin/export OS_PROJECT_NAME=TTUtunniplaan/' \
      /root/openrc_TTUtunniplaan
    sed -i 's/export OS_TENANT_NAME=admin/export OS_TENANT_NAME=TTUtunniplaan/' \
      /root/openrc_TTUtunniplaan
    source /root/openrc_TTUtunniplaan

Confirm that project is successfully scoped

    openstack token issue -f yaml

## Set permanent DNS servers on CentOS instance

    echo "supersede domain-name-servers 1.1.1.1, 8.8.8.8;" | \
      sudo tee -a /etc/dhcp/dhclient.conf 

and after VM reboot:

    cat /etc/resolv.conf 
    ; generated by /usr/sbin/dhclient-script
    search openstacklocal novalocal
    nameserver 1.1.1.1
    nameserver 8.8.8.8

# OpenStack-Ansible documentation

* https://docs.openstack.org/openstack-ansible/latest/reference/architecture/service-arch.html
* https://docs.openstack.org/openstack-ansible/latest/reference/inventory/openstack-user-config-reference.html
* https://docs.openstack.org/openstack-ansible/latest/user/aio/quickstart.html
* https://wiki.openstack.org/wiki/OpenStackAnsible
* https://git.openstack.org/cgit/openstack/openstack-ansible/


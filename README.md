# VNX Kolla-OpenStack

OpenStack scenario deployed with Kolla-ansible. The OpenStack platform is provisioned on a VNX-based virtual scenario - inspired by the [VNX Openstack Stein Lab](https://web.dit.upm.es/vnxwiki/index.php/Vnx-labo-openstack-4nodes-classic-ovs-stein).

> **IMPORTANT NOTE:**
>
> This scenario installs OpenStack **Wallaby** release.
>
> Upgrading to Xena release is expected in the future (once kolla-ansible rolls out the stable 13.00 release)

## Setup

### Pre-requisites

- Ubuntu 20.04 LTS (aka "focal")
- Python 3 (tested with Python 3.8)
- VNX ([Installation guide for Ubuntu](https://web.dit.upm.es/vnxwiki/index.php/Vnx-install-ubuntu3))

### Installing Kolla-Ansible (using virtual environments)

First, Kolla-ansible must be installed in the host. The recommended option is installing Kolla-ansible in a Python virtual environment. Execute the following commands to create a virtual environment within `ansible/.kolla-venv` folder, and then install kolla-ansible and its dependencies, i.e., ansible:

```bash
python3 -m venv ansible/.kolla-venv
source ansible/.kolla-venv/bin/activate
pip install -U pip
pip install kolla-ansible==12.2.0
pip install 'ansible<2.10'
deactivate
```

### SSH Configuration

Set proper read/write permissions for the SSH private key that Ansible will use to configure the OpenStack nodes.

```bash
sudo chown 644 conf/ssh/id_rsa
```

### Reduce virtual machines MTU

Modify dnsmasq configuration template to reduce virtual machines MTU to 1400:

```bash
echo 'dhcp-option-force=26,1400' >> ./ansible/.kolla-venv/share/kolla-ansible/ansible/roles/neutron/templates/dnsmasq.conf.j2
```

For further details on the virtual environment configuration, please visit [Kolla-ansible Virtual Environments](https://docs.openstack.org/kolla-ansible/xena/user/virtual-environments.html)

## Quickstart

### Create virtual scenario

Create a virtual scenario with VNX as follows:
```bash
sudo vnx -f openstack_lab.xml -v --create
export VNX_SCENARIO_ROOT_PATH=$(pwd)
```

### OpenStack provisioning

Activate python venv where kolla-ansible was installed:
```bash
source ansible/.kolla-venv/bin/activate
```

To provision Openstack with kolla-ansible, first prepare the target servers:
```bash
cd $VNX_SCENARIO_ROOT_PATH/ansible
kolla-ansible -i inventory/multinode --configdir kolla-config bootstrap-servers
```

Then run the `precheks` playbook to make sure that servers were properly configured with the previous playbook:
```bash
kolla-ansible -i inventory/multinode --configdir kolla-config prechecks
```

Lastly, install Openstack services in the target servers. This process will take 20 minutes roughly:
```bash
kolla-ansible -i inventory/multinode --configdir kolla-config deploy
```

Additionally, to allow external Internet access from the VMs you must configure a NAT in the host. You can easily do it using `vnx_config_nat` command distributed with VNX. Just find out the name of the public network interface of your host (i.e eth0) and execute:
```bash
sudo vnx_config_nat ExtNet eth0
```
> **WARNING:**
>
> The previous command might fail depending on your configuration of Ubuntu 20.04 LTS.
> In that case make sure `iptables` can be found under `/sbin` and execute the `vnx_config_nat` again:
 ```bash
sudo ln -s /usr/sbin/iptables /sbin/iptables
sudo vnx_config_nat ExtNet eth0
 ```

### Start demo scenario in OpenStack

Install the Python OpenStack client:
```bash
pip install python-openstackclient
```

Import credentials of Openstack admin tenant:
```bash
cd $VNX_SCENARIO_ROOT_PATH
source conf/admin-openrc.sh
```

Run the `init-runonce` utility to create demo setup - creates everything but servers.
```bash
cd $VNX_SCENARIO_ROOT_PATH
EXT_NET_CIDR='10.0.10.0/24' EXT_NET_RANGE='start=10.0.10.100,end=10.0.10.200' EXT_NET_GATEWAY='10.0.10.1' ./conf/init-runonce
```

Now you are ready to instantiate servers in the demo setup.

### Connect to Openstack Dashboard

To connect to OpenStack Dashboard, just open a web broser to http://10.0.0.11 and login with user 'admin'. The password can be obtained from conf/admin-openrc.sh script (OS_PASSWORD variable).

### Stopping the scenario

To stop the scenario preserving the configuration and the changes made:
```bash
cd $VNX_SCENARIO_ROOT_PATH
sudo vnx -f openstack_lab.xml -v --shutdown
```

## Teardown

Destroy the VNX scenario:
```bash
cd $VNX_SCENARIO_ROOT_PATH
sudo vnx -f openstack_lab.xml -v --destroy
```

To unconfigure the NAT, just execute (change eth0 by the name of your external interface):
```bash
sudo vnx_config_nat -d ExtNet eth0
```

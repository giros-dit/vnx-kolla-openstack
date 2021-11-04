# VNX Kolla-Openstack

OpenStack scenario deployed with Kolla-ansible. The OpenStack platform is provisioned on a VNX-based virtual scenario - inspired by the [VNX Openstack Stein Lab](https://web.dit.upm.es/vnxwiki/index.php/Vnx-labo-openstack-4nodes-classic-ovs-stein).

> **IMPORTANT NOTE:**
>
> This scenario installs OpenStack **Wallaby** release.
>
> Upgrading to Xena release is expected in the future (once kolla-ansible rolls out the stable 13.00 release)

## Requirements

- Ubuntu 20.04 LTS (aka "focal"
- Python 3 (tested with Python 3.8)

## Quickstart

### Create virtual scenario

Create a virtual scenario with VNX as follows:
```bash
vnx -f openstack_lab.xml -v --create
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

Run the `init-runonce` utility to create demo setup - deploys everything but servers.
```bash
cd $VNX_SCENARIO_ROOT_PATH
EXT_NET_CIDR='10.0.10.0/24' EXT_NET_RANGE='start=10.0.10.100,end=10.0.10.200' EXT_NET_GATEWAY='10.0.10.1' ./ansible/.kolla-venv/share/kolla-ansible/init-runonce
```

Now you are ready to instantiate servers in the demo setup.

### Stopping the scenario

To stop the scenario preserving the configuration and the changes made:
```bash
cd $VNX_SCENARIO_ROOT_PATH
vnx -f openstack_lab.xml -v --shutdown
```

## Teardown

Destroy the VNX scenario:
```bash
cd $VNX_SCENARIO_ROOT_PATH
vnx -f openstack_lab.xml -v --destroy
```

To unconfigure the NAT, just execute (change eth0 by the name of your external interface):
```bash
vnx_config_nat -d ExtNet eth0
```

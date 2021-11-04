

Recipe to create and pack the scenario from scratch (Ago-2019)
--------------------------------------------------------------
  
- Create the original rootfs:

cd filesystems
./create_rootfs_lxc_ubuntu64-18.04-ostack-controller 
./create_rootfs_lxc_ubuntu64-18.04-ostack-network 
./create_rootfs_lxc_ubuntu64-18.04-ostack-compute 

- Set the scenario to use the original filesystem:

cd filesystems
rm rootfs_lxc_ubuntu64-ostack-controller
ln -s vnx_rootfs_lxc_ubuntu64-18.04-v025-openstack-controller rootfs_lxc_ubuntu64-ostack-controller
cd ..

- Start the scenario and execute steps 1-6 and 8

vnx -f openstack_lab.xml -v -t  # wait till you see the controller login
vnx -f openstack_lab.xml -v -x step1-6,step8
vnx -f openstack_lab.xml -v --shutdown
# wait for all VMs to stop (all consoles closed)

- Consolidate the changes made to the controller in a new rootfs:

cd filesystems
./create_rootfs_lxc_ubuntu64-18.04-ostack-controller-cfgd 
cd ..
vnx -f openstack_lab.xml -v -P

- Pack the scenario with:

bin/pack-scenario-with-rootfs # including rootfs
bin/pack-scenario             # without rootfs


Check list to see if everything is working (not finished)
---------------------------------------------------------

- Check hypervisor list:

root@controller:~# openstack hypervisor list
+----+---------------------+-----------------+-----------+-------+
| ID | Hypervisor Hostname | Hypervisor Type | Host IP   | State |
+----+---------------------+-----------------+-----------+-------+
|  1 | compute2            | QEMU            | 10.0.0.32 | up    |
|  2 | compute1            | QEMU            | 10.0.0.31 | up    |
+----+---------------------+-----------------+-----------+-------+


- Check registered services list:

root@controller:~# openstack service list
+----------------------------------+-----------+-----------+
| ID                               | Name      | Type      |
+----------------------------------+-----------+-----------+
| 08c00e6cfcf54c04bc9f2d90110c168e | nova      | compute   |
| 711c18bafb6c4aa2b6334cfc242c838c | keystone  | identity  |
| 9210277327b34c3f92e94d46aa8113dd | glance    | image     |
| e2dec0f8630d44a39557baefdec43661 | neutron   | network   |
| e2f1845ea8d64eb6933f83012c19f453 | placement | placement |
+----------------------------------+-----------+-----------+

If any of the services is duplicated there are problems. For example, if glance is duplicated, horizon won't be able to show available images.

- If you get a "no valid host found" when creating a vm, check disk space.

- Services running on each node:

  + Controller:

service nova-api status
service nova-conductor status
service nova-consoleauth status 
service nova-novncproxy status
service nova-scheduler status 
service neutron-server status
service apache2 status



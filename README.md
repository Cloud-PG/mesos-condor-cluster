Complete Condor batch system on Mesos cluster
=========

How to deploy a Condor cluster within a Mesos cluster on Openstack via Heat


Via CLI
--------

<pre>
> git clone https://github.com/Cloud-PG/mesos-condor-ams-cluster.git
> cd mesos-condor-ams-cluster
> vim env_heat_condor
> heat stack-create -f mesoscluster-condor.yaml -e env_heat_condor CLUSTER_NAME</pre>

In "env_heat" modify this parameters:
- `network` is the id of the Openstack network used in your project;
- `ssh_key_name` is the name of the ssh key to inject into your cluster machines;
- `master_flavor`, `loadbalancer_flavor` and `slave_flavor` are the names or ids of the flavors to be used to create the mesos master/slave/loadbalancer VMs;
- `number_of_slaves` and `number_of_masters` is the number of VMs to spawn;
- `server_image` is the name/id of the virtual image to be used to launch the VMs;
- `public_key_url` is an url to download the public key used to connect to the schedd.


Via OS Web Interface
-----------
ToDo

Tests
==================

ToDo

<pre>root@mesos-s1:~# docker exec -it "schedd-docker-id" bash

root@f329f012e05f:/# yum install vim -y
root@f329f012e05f:/# useradd -m -s /bin/bash asd
root@f329f012e05f:/# su - asd

asd@f329f012e05f:~$ vim sleep.sh
#!/bin/bash
/bin/sleep 20
asd@f329f012e05f:~$ vim sleep.sub
executable              = sleep.sh
log                     = sleep.log
output                  = outfile.txt
error                   = errors.txt
should_transfer_files   = Yes
when_to_transfer_output = ON_EXIT
queue

asd@f329f012e05f:~$ condor_status
asd@f329f012e05f:~$ condor_submit sleep.sub
asd@f329f012e05f:~$ condor_q
asd@f329f012e05f:~$ condor_status</pre>

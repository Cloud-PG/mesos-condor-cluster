Complete Condor batch system on Mesos cluster
=========

How to deploy Condor within a Mesos cluster on Openstack via Heat

<pre>
> git clone https://github.com/Cloud-PG/mesos-condor-cluster.git
> cd mesos-condor-cluster
> vim env_heat_condor
> heat stack-create -f mesoscluster-condor.yaml -e env_heat_condor CLUSTER_NAME</pre>

In "env_heat" modify this parameters:
- `network` is the id of the Openstack network used in your project;
- `ssh_key_name` is the name of the ssh key to inject into your cluster machines;
- `master_flavor`, `loadbalancer_flavor` and `slave_flavor` are the names or ids of the flavors to be used to create the mesos master/slave/loadbalancer VMs;
- `number_of_slaves` and `number_of_masters` is the number of VMs to spawn;
- `server_image` is the name/id of the virtual image to be used to launch the VMs;

You now have an empty Mesos/Marathon cluster.

Run Condor dockers manually
--------------

<pre>
> docker run  -d --name=master2 dscnaf/htcondor-debian -m</pre>

Using Marathon
--------------
Master json:
<pre>{
  "id": "htcondor-master-debian",
  "args": ["-m"],
  "cpus": 1,
  "mem": 512.0,
  "container": {
    "type": "DOCKER",
    "docker": {
      "image": "ciangom/htcondor-docker-debian",
      "network": "BRIDGE",
      "portMappings": [
        {"containerPort": 9618, "servicePort": 9618}
      ]
    }
  }
}</pre>

Submitter json:
<pre>{
  "id": "htcondor-submitter-debian",
  "args": ["-s", "192.168.150.65"],
  "cpus": 1,
  "mem": 512.0,
  "container": {
    "type": "DOCKER",
    "docker": {
      "image": "ciangom/htcondor-docker-debian",
      "network": "BRIDGE"
    }
  }
}</pre>

Executor json:
<pre>{
  "id": "htcondor-executor-debian",
  "args": ["-e", "192.168.150.65"],
  "cpus": 1,
  "mem": 512.0,
  "instances": 3,
  "container": {
    "type": "DOCKER",
    "docker": {
      "image": "ciangom/htcondor-docker-debian",
      "privileged": true,
      "network": "BRIDGE"
    }
  }
}</pre>

Tests
==================

From the slave containing schedd-docker:

<pre>root@mesos-s1:~# docker exec -it "schedd-docker-id" bash

root@f329f012e05f:/# apt-get install vim -y
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

Docker problems (mail Marica)
==================

In /etc/condor/condor_config wrong path for "libexec"; correct one is /usr/lib/condor/libexec, current is /usr/libexec


Update: Riesco a specificare le porte e gli indirizzi, ma lo schedd cerca di collegarsi all'indirizzo locale del master (172. ecc) mentre il master è in ascolto sull'indirizzo ip dell'host. Da risolvere

Update2: Risolto problema porte. Rimodificato il condor_config riguardo alle policy di ALLOW

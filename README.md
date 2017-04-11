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
docker run  -d --name=master2 dscnaf/htcondor-debian -m</pre>

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
      "image": "dscnaf/htcondor-debian"
    }
  }
}</pre>

Submitter json:
<pre>{
  "id": "htcondor-submitter-debian",
  "args": ["-s", "MASTER_IP_ADDRESS"],
  "cpus": 1,
  "mem": 512.0,
  "container": {
    "type": "DOCKER",
    "docker": {
      "image": "dscnaf/htcondor-debian"
    }
  }
}</pre>

Executor json:
<pre>{
  "id": "htcondor-executor-debian",
  "args": ["-e", "MASTER_IP_ADDRESS"],
  "cpus": 1,
  "mem": 512.0,
  "instances": 3,
  "container": {
    "type": "DOCKER",
    "docker": {
      "image": "dscnaf/htcondor-debian",
      "privileged": true
    }
  }
}</pre>

Docker problems (mail Marica)
==================

In /etc/condor/condor_config wrong path for "libexec"; correct one is /usr/lib/condor/libexec, current is /usr/libexec

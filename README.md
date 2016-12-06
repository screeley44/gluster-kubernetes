# gluster-kubernetes

## Dynamic, persistent storage for Kubernetes with GlusterFS

This is the home of the **gluster-kubernetes** project,
a new project which brings dynamically provisioned persistent
storage volumes to kubernetes, using a hyperconverged Gluster
deployment scheme.

### The components in this project are:

* **Kubernetes** (https://github.com/kubernetes/kubernetes/), the container management system.
* **GlusterFS** (https://www.gluster.org/), the scale-out storage system, running as pods inside the kubernetes cluster.
* **heketi** (https://github.com/heketi/heketi), Gluster's volume management service interface, running in a pod inside kubernetes.

### Quickstart

You can start with your own Kubernetes installation ready to go, or you can
use the vagrant setup in the `vagrant/` directory to spin up a Kubernetes
VM cluster for you. To run the vagrant setup, you'll need to have the
following installed:

 * ansible
 * vagrant
 * libvirt or VirtualBox

To spin up the cluster, simply run `./up.sh` in the `vagrant/` directory.

Next, copy the `deploy/` directory to the master node of the cluster.

You will have to provide your own topology file. A sample topology file is
included in the `deploy/` directory (default location that gk-deploy expects)
which can be used as the topology for the vagrant libvirt setup. When
creating your own topology file:

 * Make sure the topology file only lists block devices intended for heketi's
 use. heketi needs access to whole block devices (e.g. /dev/sdb, /dev/vdb)
 which it will partition and format.

 * The `hostnames` array is a bit misleading. `manage` should be a list of
 hostnames for the node, but `storage` should be a list of IP addresses on
 the node for backend storage communications.

If you used the provided vagrant libvirt setup, you can run:

```bash
$ vagrant ssh-config > ssh-config
$ scp -rF ssh-config ../deploy master:
$ vagrant ssh master
[vagrant@master]$ cd deploy
[vagrant@master]$ mv topology.json.sample topology.json
```

The following commands are meant to be run with administrative privileges.

For ease of use in the the vagrant setup, we recommend you run the following:

```bash
$ export KUBECONFIG="/etc/kubernetes/admin.conf"
```

To verify the Kubernetes installation, run the following command:

```bash
$ kubectl get nodes
NAME      STATUS    AGE
master    Ready     22h
node0     Ready     22h
node1     Ready     22h
node2     Ready     22h
```

***NOTE***: To see the version of Kubernetes (which will change based on latest official releases) simply do `kubectl version`


Next, to deploy heketi, run the following:

```bash
$ ./gk-deploy -g
```

If you already have GlusterFS deployed in your cluster, you do not need the
`-g` option.


After this completes, to aid in testing and following the rest of the examples, the following export 
should be run and will set the HEKETI_CLI_SERVER env variable and will also give an
easy means to echo the REST URL for the Heketi Server

```bash
$ export HEKETI_CLI_SERVER=$(kubectl describe svc/heketi | grep "Endpoints:" | awk '{print "http://"$2}')

$ echo $HEKETI_CLI_SERVER
http://10.42.0.0:8080
```

kubernetes and heketi should now be installed and ready to go. 
To verify Kubernetes installation run the following commands:

```bash
$ kubectl get nodes
NAME      STATUS    AGE
master    Ready     22h
node0     Ready     22h
node1     Ready     22h
node2     Ready     22h

$ kubectl get pods
NAME                               READY     STATUS              RESTARTS   AGE
glusterfs-node0-2509304327-vpce1   1/1       Running             0          1d
glusterfs-node1-3290690057-hhq92   1/1       Running             0          1d
glusterfs-node2-4072075787-okzjv   1/1       Running             0          1d
heketi-3017632314-yyngh            1/1       Running             0          1d
```

Notice that we have 3 gluster pods running and 1 heketi pod running

To verify Heketi installation and REST Url is working:
```bash
curl <result of echo of $HEKETI_CLI_SERVER>/hello

$ curl http://10.42.0.0:8080/hello
Hello from Heketi
```

### Try It Out

* Continue to an example of the [Hello World application using GlusterFS Dynamic Provisioning](./docs/examples/hello_world/README.md)

Alternatively, you should now also be able to use `heketi-cli` to create/manage volumes and then mount
those volumes to verify they're working.

### Demo

**>>> [Video demo of the technology!](https://drive.google.com/file/d/0B667S2caJiy7QVpzVVFNQVdyaVE/view?usp=sharing) <<<**


### Documentation

* [Setup Guide](./docs/setup-guide.md)

* [Hello World with GlusterFS Dynamic Provisioning](./docs/examples/hello_world/README.md)

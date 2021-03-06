---
page_title: Docker Swarm discovery
page_description: Swarm discovery
page_keywords: docker, swarm, clustering, discovery
---

# Discovery

`Docker Swarm` comes with multiple Discovery backends

## Examples

### Using the hosted discovery service

```bash
# create a cluster
$ swarm create
6856663cdefdec325839a4b7e1de38e8 # <- this is your unique <cluster_id>

# on each of your nodes, start the swarm agent
#  <node_ip> doesn't have to be public (eg. 192.168.0.X),
#  as long as the swarm manager can access it.
$ swarm join --addr=<node_ip:2375> token://<cluster_id>

# start the manager on any machine or your laptop
$ swarm manage -H tcp://<swarm_ip:swarm_port> token://<cluster_id>

# use the regular docker cli
$ docker -H tcp://<swarm_ip:swarm_port> info
$ docker -H tcp://<swarm_ip:swarm_port> run ...
$ docker -H tcp://<swarm_ip:swarm_port> ps
$ docker -H tcp://<swarm_ip:swarm_port> logs ...
...

# list nodes in your cluster
$ swarm list token://<cluster_id>
<node_ip:2375>
```

### Using a static file describing the cluster

```bash
# for each of your nodes, add a line to a file
#  <node_ip> doesn't have to be public (eg. 192.168.0.X),
#  as long as the swarm manager can access it.
$ echo <node_ip1:2375> >> /tmp/my_cluster
$ echo <node_ip2:2375> >> /tmp/my_cluster
$ echo <node_ip3:2375> >> /tmp/my_cluster

# start the manager on any machine or your laptop
$ swarm manage -H tcp://<swarm_ip:swarm_port> file:///tmp/my_cluster

# use the regular docker cli
$ docker -H tcp://<swarm_ip:swarm_port> info
$ docker -H tcp://<swarm_ip:swarm_port> run ...
$ docker -H tcp://<swarm_ip:swarm_port> ps
$ docker -H tcp://<swarm_ip:swarm_port> logs ...
...

# list nodes in your cluster
$ swarm list file:///tmp/my_cluster
<node_ip1:2375>
<node_ip2:2375>
<node_ip3:2375>
```

### Using etcd

```bash
# on each of your nodes, start the swarm agent
#  <node_ip> doesn't have to be public (eg. 192.168.0.X),
#  as long as the swarm manager can access it.
$ swarm join --addr=<node_ip:2375> etcd://<etcd_ip>/<path>

# start the manager on any machine or your laptop
$ swarm manage -H tcp://<swarm_ip:swarm_port> etcd://<etcd_ip>/<path>

# use the regular docker cli
$ docker -H tcp://<swarm_ip:swarm_port> info
$ docker -H tcp://<swarm_ip:swarm_port> run ...
$ docker -H tcp://<swarm_ip:swarm_port> ps
$ docker -H tcp://<swarm_ip:swarm_port> logs ...
...

# list nodes in your cluster
$ swarm list etcd://<etcd_ip>/<path>
<node_ip:2375>
```

### Using consul

```bash
# on each of your nodes, start the swarm agent
#  <node_ip> doesn't have to be public (eg. 192.168.0.X),
#  as long as the swarm manager can access it.
$ swarm join --addr=<node_ip:2375> consul://<consul_addr>/<path>

# start the manager on any machine or your laptop
$ swarm manage -H tcp://<swarm_ip:swarm_port> consul://<consul_addr>/<path>

# use the regular docker cli
$ docker -H tcp://<swarm_ip:swarm_port> info
$ docker -H tcp://<swarm_ip:swarm_port> run ...
$ docker -H tcp://<swarm_ip:swarm_port> ps
$ docker -H tcp://<swarm_ip:swarm_port> logs ...
...

# list nodes in your cluster
$ swarm list consul://<consul_addr>/<path>
<node_ip:2375>
```

### Using zookeeper

```bash
# on each of your nodes, start the swarm agent
#  <node_ip> doesn't have to be public (eg. 192.168.0.X),
#  as long as the swarm manager can access it.
$ swarm join --addr=<node_ip:2375> zk://<zookeeper_addr1>,<zookeeper_addr2>/<path>

# start the manager on any machine or your laptop
$ swarm manage -H tcp://<swarm_ip:swarm_port> zk://<zookeeper_addr1>,<zookeeper_addr2>/<path>

# use the regular docker cli
$ docker -H tcp://<swarm_ip:swarm_port> info
$ docker -H tcp://<swarm_ip:swarm_port> run ...
$ docker -H tcp://<swarm_ip:swarm_port> ps
$ docker -H tcp://<swarm_ip:swarm_port> logs ...
...

# list nodes in your cluster
$ swarm list zk://<zookeeper_addr1>,<zookeeper_addr2>/<path>
<node_ip:2375>
```

### Using a static list of ips

```bash
# start the manager on any machine or your laptop
$ swarm manage -H <swarm_ip:swarm_port> nodes://<node_ip1:2375>,<node_ip2:2375>
# or
$ swarm manage -H <swarm_ip:swarm_port> <node_ip1:2375>,<node_ip2:2375>

# use the regular docker cli
$ docker -H <swarm_ip:swarm_port> info
$ docker -H <swarm_ip:swarm_port> run ...
$ docker -H <swarm_ip:swarm_port> ps
$ docker -H <swarm_ip:swarm_port> logs ...
...
```

### Range pattern for IP addresses

The `file` and `nodes` discoveries support a range pattern to specify IP addresses, i.e., `10.0.0.[10:200]` will be a list of nodes starting from `10.0.0.10` to `10.0.0.200`.

For example,

```bash
# file example
$ echo "10.0.0.[11:100]:2375"   >> /tmp/my_cluster
$ echo "10.0.1.[15:20]:2375"    >> /tmp/my_cluster
$ echo "192.168.1.2:[2:20]375"  >> /tmp/my_cluster

# start the manager
$ swarm manage -H tcp://<swarm_ip:swarm_port> file:///tmp/my_cluster
```

```bash
# nodes example
$ swarm manage -H <swarm_ip:swarm_port> "nodes://10.0.0.[10:200]:2375,10.0.1.[2:250]:2375"
```

## Contributing a new discovery backend

Contributing a new discovery backend is easy,
simply implements this interface:

```go
type DiscoveryService interface {
     Initialize(string, int) error
     Fetch() ([]string, error)
     Watch(WatchCallback)
     Register(string) error
}
```

### Initialize
The parameters are `discovery` location without the scheme and a heartbeat (in seconds)

### Fetch
Returns the list of all the nodes from the discovery

### Watch
Triggers an update (`Fetch`). This can happen either via
a timer (like `token`) or use backend specific features (like `etcd`)

### Register
Add a new node to the discovery service.

## Docker Swarm documentation index

- [User guide](./index.md)
- [Sheduler strategies](./scheduler/strategy.md)
- [Sheduler filters](./scheduler/filter.md)
- [Swarm API](./API.md)

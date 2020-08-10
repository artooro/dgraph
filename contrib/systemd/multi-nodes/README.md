# systemd configuration for multiple nodes

This following document describes how to configure several nodes that are managed through [systemd](https://systemd.io/).

## Overview

You will configure the following types of Dgraph nodes:

* zero nodes
  * zero leader node - an initial leader node configured at start of cluster, e.g. `zero0`
  * zero peer nodes - peer nodes, , e.g. `zero1`, `zero2`, that point to the zero leader
* alpha nodes - configured similarly, e.g. `alpha0`, `alpha1`, `alpha2`, that point to list of all zero nodes


> **NOTE** These commands are run as root using bash shell.

## All Nodes (Zero and Alpha)

On all systems that will run a dgraph service, create `dgraph` group and user.

```bash
groupadd --system dgraph
useradd --system --home-dir /var/lib/dgraph --shell /bin/false --gid dgraph dgraph
```

## All Zero Nodes (Leader and Peers)

On all Zero Nodes, create the these directory paths that are owned by `dgraph` user:

```bash
mkdir --parents /var/{log/dgraph,lib/dgraph/zw}
chown --recursive dgraph:dgraph /var/{lib,log}/dgraph
```

### Configure Zero Leader Node

Edit the file [dgraph-zero-leader.service](dgraph-zero-leader.service) as necessary.  There are two parameters to pay particular attention:

* `--replicas` - total number of zeros
* `--idx` - initial zero node

Copy the edited file to `/etc/systemd/system/dgraph-zero.service` and run the following:

```bash
systemctl enable dgraph-zero
systemctl start dgraph-zero
```

### Configure Zero Peer Nodes

This process is similar to previous step. Edit the file [dgraph-zero-peer.service](dgraph-zero-peer.service) as required. Replace the string `{{ zero0 }}` to match the hostname of the zero leader, such as `zero0.mycompany.com`.  You can also adjust `idx` and `replicas` if needed.

Copy the edited file to `/etc/systemd/system/dgraph-zero.service` and run the following:

```bash
systemctl enable dgraph-zero
systemctl start dgraph-zero
```

### Configure Firewall for Zero Ports

For zero you will want to open up ports `5080` and `6080`.  This process will vary depending on firewall you are using.  Some examples below:


On **Ubuntu 18.04**:

```bash
ufw allow from any to any port 5080 proto tcp
ufw allow from any to any port 6080 proto tcp
```

On **CentOS 8**:


```bash
# NOTE: public zone is the default and includes NIC used to access service
firewall-cmd --zone=public --permanent --add-port=5080/tcp
firewall-cmd --zone=public --permanent --add-port=6080/tcp
firewall-cmd --reload
```

## Configure Alpha Nodes

On all Alpha Nodes, create the these directory paths that are owned by `dgraph` user:

```bash
mkdir --parents /var/{log/dgraph,lib/dgraph/{w,p}}
chown --recursive dgraph:dgraph /var/{lib,log}/dgraph
```

Edit the file [dgraph-alpha.service](dgraph-alpha.service) as required.  For the `--zero` prameter, you want to create a list that matches all the zeros in your cluster, so that when `{{ zero0 }}`, `{{ zero1 }}`, and `{{ zero2 }}` are replaced, you will have a string something like this (adjusted to your organization's domain):

```
--zero zero0.mycompany.com:5080,zero0.mycompany.com:5080,zero0.mycompany.com:5080
```

Copy the edited file to `/etc/systemd/system/dgraph-alpha.service` and run the following:

```bash
systemctl enable dgraph-alpha
systemctl start dgraph-alpha
```

### Configure Firewall for Alpha Ports

For alpha you will want to open up ports `7080`, `8080`, and `9080`. This process will vary depending on firewall you are using.  Some examples below:


On **Ubuntu 18.04**:

```bash
ufw allow from any to any port 7080 proto tcp
ufw allow from any to any port 8080 proto tcp
ufw allow from any to any port 9080 proto tcp
```

On **CentOS 8**:


```bash
# NOTE: public zone is the default and includes NIC used to access service
firewall-cmd --zone=public --permanent --add-port=7080/tcp
firewall-cmd --zone=public --permanent --add-port=8080/tcp
firewall-cmd --zone=public --permanent --add-port=9080/tcp
firewall-cmd --reload
```

## Verifying Services

Below are examples of checking the health of the nodes and cluster.

> **NOTE** Replace mycompany.com to your domain or use the IP address.

### Zero Nodes

You can check the health and state endpoints of the service:

```bash
curl zero0.mycompany.com:6080/health
curl zero0.mycompany.com:6080/state
```

On the system itself, you can check the serivce status and logs:

```bash
systemctl status dgraph-zero
journalctl -u dgraph-zero
```

### Alpha Nodes

You can check the health and state endpoints of the service:

```bash
curl alpha0.mycompany.com:6080/health
curl alpha0.mycompany.com:6080/state
```

On the system itself, you can check the serivce status and logs:

```bash
systemctl status dgraph-alpha
journalctl -u dgraph-alpha
```
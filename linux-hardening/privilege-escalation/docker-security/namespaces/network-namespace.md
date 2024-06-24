# Network Namespace


## Basic Information

A network namespace is a Linux kernel feature that provides isolation of the network stack, allowing **each network namespace to have its own independent network configuration**, interfaces, IP addresses, routing tables, and firewall rules. This isolation is useful in various scenarios, such as containerization, where each container should have its own network configuration, independent of other containers and the host system.

### How it works:

1. When a new network namespace is created, it starts with a **completely isolated network stack**, with **no network interfaces** except for the loopback interface (lo). This means that processes running in the new network namespace cannot communicate with processes in other namespaces or the host system by default.
2. **Virtual network interfaces**, such as veth pairs, can be created and moved between network namespaces. This allows for establishing network connectivity between namespaces or between a namespace and the host system. For example, one end of a veth pair can be placed in a container's network namespace, and the other end can be connected to a **bridge** or another network interface in the host namespace, providing network connectivity to the container.
3. Network interfaces within a namespace can have their **own IP addresses, routing tables, and firewall rules**, independent of other namespaces. This allows processes in different network namespaces to have different network configurations and operate as if they are running on separate networked systems.
4. Processes can move between namespaces using the `setns()` system call, or create new namespaces using the `unshare()` or `clone()` system calls with the `CLONE_NEWNET` flag. When a process moves to a new namespace or creates one, it will start using the network configuration and interfaces associated with that namespace.

## Lab:

### Create different Namespaces

#### CLI

```bash
sudo unshare -n [--mount-proc] /bin/bash
# Run ifconfig or ip -a
```

By mounting a new instance of the `/proc` filesystem if you use the param `--mount-proc`, you ensure that the new mount namespace has an **accurate and isolated view of the process information specific to that namespace**.


#### Docker

```bash
docker run -ti --name ubuntu1 -v /usr:/ubuntu1 ubuntu bash
# Run ifconfig or ip -a
```

### &#x20;Check which namespace is your process in

```bash
ls -l /proc/self/ns/net
lrwxrwxrwx 1 root root 0 Apr  4 20:30 /proc/self/ns/net -> 'net:[4026531840]'
```

### Find all Network namespaces

{% code overflow="wrap" %}
```bash
sudo find /proc -maxdepth 3 -type l -name net -exec readlink {} \; 2>/dev/null | sort -u | grep "net:"
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name net -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```
{% endcode %}

### Enter inside a Network namespace

```bash
nsenter -n TARGET_PID --pid /bin/bash
```

Also, you can only **enter in another process namespace if you are root**. And you **cannot** **enter** in other namespace **without a descriptor** pointing to it (like `/proc/self/ns/net`).

## References
* [https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory](https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory)


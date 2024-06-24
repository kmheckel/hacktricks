# CGroup Namespace


## Basic Information

A cgroup namespace is a Linux kernel feature that provides **isolation of cgroup hierarchies for processes running within a namespace**. Cgroups, short for **control groups**, are a kernel feature that allows organizing processes into hierarchical groups to manage and enforce **limits on system resources** like CPU, memory, and I/O.

While cgroup namespaces are not a separate namespace type like the others we discussed earlier (PID, mount, network, etc.), they are related to the concept of namespace isolation. **Cgroup namespaces virtualize the view of the cgroup hierarchy**, so that processes running within a cgroup namespace have a different view of the hierarchy compared to processes running in the host or other namespaces.

### How it works:

1. When a new cgroup namespace is created, **it starts with a view of the cgroup hierarchy based on the cgroup of the creating process**. This means that processes running in the new cgroup namespace will only see a subset of the entire cgroup hierarchy, limited to the cgroup subtree rooted at the creating process's cgroup.
2. Processes within a cgroup namespace will **see their own cgroup as the root of the hierarchy**. This means that, from the perspective of processes inside the namespace, their own cgroup appears as the root, and they cannot see or access cgroups outside of their own subtree.
3. Cgroup namespaces do not directly provide isolation of resources; **they only provide isolation of the cgroup hierarchy view**. **Resource control and isolation are still enforced by the cgroup** subsystems (e.g., cpu, memory, etc.) themselves.

For more information about CGroups check:

{% content-ref url="../cgroups.md" %}
[cgroups.md](../cgroups.md)
{% endcontent-ref %}

## Lab:

### Create different Namespaces

#### CLI

```bash
sudo unshare -C [--mount-proc] /bin/bash
```

By mounting a new instance of the `/proc` filesystem if you use the param `--mount-proc`, you ensure that the new mount namespace has an **accurate and isolated view of the process information specific to that namespace**.


#### Docker

```bash
docker run -ti --name ubuntu1 -v /usr:/ubuntu1 ubuntu bash
```

### &#x20;Check which namespace is your process in

```bash
ls -l /proc/self/ns/cgroup
lrwxrwxrwx 1 root root 0 Apr  4 21:19 /proc/self/ns/cgroup -> 'cgroup:[4026531835]'
```

### Find all CGroup namespaces

{% code overflow="wrap" %}
```bash
sudo find /proc -maxdepth 3 -type l -name cgroup -exec readlink {} \; 2>/dev/null | sort -u
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name cgroup -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```
{% endcode %}

### Enter inside an CGroup namespace

```bash
nsenter -C TARGET_PID --pid /bin/bash
```

Also, you can only **enter in another process namespace if you are root**. And you **cannot** **enter** in other namespace **without a descriptor** pointing to it (like `/proc/self/ns/cgroup`).

## References
* [https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory](https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory)


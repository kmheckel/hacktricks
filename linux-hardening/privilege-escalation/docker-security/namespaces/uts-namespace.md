# UTS Namespace


## Basic Information

A UTS (UNIX Time-Sharing System) namespace is a Linux kernel feature that provides i**solation of two system identifiers**: the **hostname** and the **NIS** (Network Information Service) domain name. This isolation allows each UTS namespace to have its **own independent hostname and NIS domain name**, which is particularly useful in containerization scenarios where each container should appear as a separate system with its own hostname.

### How it works:

1. When a new UTS namespace is created, it starts with a **copy of the hostname and NIS domain name from its parent namespace**. This means that, at creation, the new namespace s**hares the same identifiers as its parent**. However, any subsequent changes to the hostname or NIS domain name within the namespace will not affect other namespaces.
2. Processes within a UTS namespace **can change the hostname and NIS domain name** using the `sethostname()` and `setdomainname()` system calls, respectively. These changes are local to the namespace and do not affect other namespaces or the host system.
3. Processes can move between namespaces using the `setns()` system call or create new namespaces using the `unshare()` or `clone()` system calls with the `CLONE_NEWUTS` flag. When a process moves to a new namespace or creates one, it will start using the hostname and NIS domain name associated with that namespace.

## Lab:

### Create different Namespaces

#### CLI

```bash
sudo unshare -u [--mount-proc] /bin/bash
```

By mounting a new instance of the `/proc` filesystem if you use the param `--mount-proc`, you ensure that the new mount namespace has an **accurate and isolated view of the process information specific to that namespace**.


#### Docker

```bash
docker run -ti --name ubuntu1 -v /usr:/ubuntu1 ubuntu bash
```

### &#x20;Check which namespace is your process in

```bash
ls -l /proc/self/ns/uts
lrwxrwxrwx 1 root root 0 Apr  4 20:49 /proc/self/ns/uts -> 'uts:[4026531838]'
```

### Find all UTS namespaces

{% code overflow="wrap" %}
```bash
sudo find /proc -maxdepth 3 -type l -name uts -exec readlink {} \; 2>/dev/null | sort -u
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name uts -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```
{% endcode %}

### Enter inside an UTS namespace

```bash
nsenter -u TARGET_PID --pid /bin/bash
```

Also, you can only **enter in another process namespace if you are root**. And you **cannot** **enter** in other namespace **without a descriptor** pointing to it (like `/proc/self/ns/uts`).

### Change hostname

```bash
unshare -u /bin/bash
hostname newhostname # Hostname won't be changed inside the host UTS ns
```

## References
* [https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory](https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory)


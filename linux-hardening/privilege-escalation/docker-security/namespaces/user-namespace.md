# User Namespace


## Basic Information

A user namespace is a Linux kernel feature that **provides isolation of user and group ID mappings**, allowing each user namespace to have its **own set of user and group IDs**. This isolation enables processes running in different user namespaces to **have different privileges and ownership**, even if they share the same user and group IDs numerically.

User namespaces are particularly useful in containerization, where each container should have its own independent set of user and group IDs, allowing for better security and isolation between containers and the host system.

### How it works:

1. When a new user namespace is created, it **starts with an empty set of user and group ID mappings**. This means that any process running in the new user namespace will **initially have no privileges outside of the namespace**.
2. ID mappings can be established between the user and group IDs in the new namespace and those in the parent (or host) namespace. This **allows processes in the new namespace to have privileges and ownership corresponding to user and group IDs in the parent namespace**. However, the ID mappings can be restricted to specific ranges and subsets of IDs, allowing for fine-grained control over the privileges granted to processes in the new namespace.
3. Within a user namespace, **processes can have full root privileges (UID 0) for operations inside the namespace**, while still having limited privileges outside the namespace. This allows **containers to run with root-like capabilities within their own namespace without having full root privileges on the host system**.
4. Processes can move between namespaces using the `setns()` system call or create new namespaces using the `unshare()` or `clone()` system calls with the `CLONE_NEWUSER` flag. When a process moves to a new namespace or creates one, it will start using the user and group ID mappings associated with that namespace.

## Lab:

### Create different Namespaces

#### CLI

```bash
sudo unshare -U [--mount-proc] /bin/bash
```

By mounting a new instance of the `/proc` filesystem if you use the param `--mount-proc`, you ensure that the new mount namespace has an **accurate and isolated view of the process information specific to that namespace**.


#### Docker

```bash
docker run -ti --name ubuntu1 -v /usr:/ubuntu1 ubuntu bash
```

To use user namespace, Docker daemon needs to be started with **`--userns-remap=default`**(In ubuntu 14.04, this can be done by modifying `/etc/default/docker` and then executing `sudo service docker restart`)

### &#x20;Check which namespace is your process in

```bash
ls -l /proc/self/ns/user
lrwxrwxrwx 1 root root 0 Apr  4 20:57 /proc/self/ns/user -> 'user:[4026531837]'
```

It's possible to check the user map from the docker container with:

```bash
cat /proc/self/uid_map 
         0          0 4294967295  --> Root is root in host
         0     231072      65536  --> Root is 231072 userid in host
```

Or from the host with:

```bash
cat /proc/<pid>/uid_map 
```

### Find all User namespaces

{% code overflow="wrap" %}
```bash
sudo find /proc -maxdepth 3 -type l -name user -exec readlink {} \; 2>/dev/null | sort -u
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name user -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```
{% endcode %}

### Enter inside a User namespace

```bash
nsenter -U TARGET_PID --pid /bin/bash
```

Also, you can only **enter in another process namespace if you are root**. And you **cannot** **enter** in other namespace **without a descriptor** pointing to it (like `/proc/self/ns/user`).

### Create new User namespace (with mappings)

{% code overflow="wrap" %}
```bash
unshare -U [--map-user=<uid>|<name>] [--map-group=<gid>|<name>] [--map-root-user] [--map-current-user]
```
{% endcode %}

```bash
# Container
sudo unshare -U /bin/bash
nobody@ip-172-31-28-169:/home/ubuntu$ #Check how the user is nobody

# From the host
ps -ef | grep bash # The user inside the host is still root, not nobody
root       27756   27755  0 21:11 pts/10   00:00:00 /bin/bash
```

### Recovering Capabilities

In the case of user namespaces, **when a new user namespace is created, the process that enters the namespace is granted a full set of capabilities within that namespace**. These capabilities allow the process to perform privileged operations such as **mounting** **filesystems**, creating devices, or changing ownership of files, but **only within the context of its user namespace**.

For example, when you have the `CAP_SYS_ADMIN` capability within a user namespace, you can perform operations that typically require this capability, like mounting filesystems, but only within the context of your user namespace. Any operations you perform with this capability won't affect the host system or other namespaces.

{% hint style="warning" %}
Therefore, even if getting a new process inside a new User namespace **will give you all the capabilities back** (CapEff: 000001ffffffffff), you actually can **only use the ones related to the namespace** (mount for example) but not every one. So, this on its own is not enough to escape from a Docker container.
{% endhint %}

```bash
# There are the syscalls that are filtered after changing User namespace with:
unshare -UmCpf  bash

Probando: 0x067 . . . Error
Probando: 0x070 . . . Error
Probando: 0x074 . . . Error
Probando: 0x09b . . . Error
Probando: 0x0a3 . . . Error
Probando: 0x0a4 . . . Error
Probando: 0x0a7 . . . Error
Probando: 0x0a8 . . . Error
Probando: 0x0aa . . . Error
Probando: 0x0ab . . . Error
Probando: 0x0af . . . Error
Probando: 0x0b0 . . . Error
Probando: 0x0f6 . . . Error
Probando: 0x12c . . . Error
Probando: 0x130 . . . Error
Probando: 0x139 . . . Error
Probando: 0x140 . . . Error
Probando: 0x141 . . . Error
Probando: 0x143 . . . Error
```

## References
* [https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory](https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory)


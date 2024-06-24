# IPC Namespace


## Basic Information

An IPC (Inter-Process Communication) namespace is a Linux kernel feature that provides **isolation** of System V IPC objects, such as message queues, shared memory segments, and semaphores. This isolation ensures that processes in **different IPC namespaces cannot directly access or modify each other's IPC objects**, providing an additional layer of security and privacy between process groups.

### How it works:

1. When a new IPC namespace is created, it starts with a **completely isolated set of System V IPC objects**. This means that processes running in the new IPC namespace cannot access or interfere with the IPC objects in other namespaces or the host system by default.
2. IPC objects created within a namespace are visible and **accessible only to processes within that namespace**. Each IPC object is identified by a unique key within its namespace. Although the key may be identical in different namespaces, the objects themselves are isolated and cannot be accessed across namespaces.
3. Processes can move between namespaces using the `setns()` system call or create new namespaces using the `unshare()` or `clone()` system calls with the `CLONE_NEWIPC` flag. When a process moves to a new namespace or creates one, it will start using the IPC objects associated with that namespace.

## Lab:

### Create different Namespaces

#### CLI

```bash
sudo unshare -i [--mount-proc] /bin/bash
```

By mounting a new instance of the `/proc` filesystem if you use the param `--mount-proc`, you ensure that the new mount namespace has an **accurate and isolated view of the process information specific to that namespace**.


#### Docker

```bash
docker run -ti --name ubuntu1 -v /usr:/ubuntu1 ubuntu bash
```

### &#x20;Check which namespace is your process in

```bash
ls -l /proc/self/ns/ipc
lrwxrwxrwx 1 root root 0 Apr  4 20:37 /proc/self/ns/ipc -> 'ipc:[4026531839]'
```

### Find all IPC namespaces

{% code overflow="wrap" %}
```bash
sudo find /proc -maxdepth 3 -type l -name ipc -exec readlink {} \; 2>/dev/null | sort -u
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name ipc -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```
{% endcode %}

### Enter inside an IPC namespace

```bash
nsenter -i TARGET_PID --pid /bin/bash
```

Also, you can only **enter in another process namespace if you are root**. And you **cannot** **enter** in other namespace **without a descriptor** pointing to it (like `/proc/self/ns/net`).

### Create IPC object

```bash
# Container
sudo unshare -i /bin/bash
ipcmk -M 100
Shared memory id: 0
ipcs -m

------ Shared Memory Segments --------
key        shmid      owner      perms      bytes      nattch     status      
0x2fba9021 0          root       644        100        0    

# From the host
ipcs -m # Nothing is seen
```

## References
* [https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory](https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory)




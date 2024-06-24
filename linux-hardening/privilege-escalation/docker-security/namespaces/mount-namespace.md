# Mount Namespace


## Basic Information

A mount namespace is a Linux kernel feature that provides isolation of the file system mount points seen by a group of processes. Each mount namespace has its own set of file system mount points, and **changes to the mount points in one namespace do not affect other namespaces**. This means that processes running in different mount namespaces can have different views of the file system hierarchy.

Mount namespaces are particularly useful in containerization, where each container should have its own file system and configuration, isolated from other containers and the host system.

### How it works:

1. When a new mount namespace is created, it is initialized with a **copy of the mount points from its parent namespace**. This means that, at creation, the new namespace shares the same view of the file system as its parent. However, any subsequent changes to the mount points within the namespace will not affect the parent or other namespaces.
2. When a process modifies a mount point within its namespace, such as mounting or unmounting a file system, the **change is local to that namespace** and does not affect other namespaces. This allows each namespace to have its own independent file system hierarchy.
3. Processes can move between namespaces using the `setns()` system call, or create new namespaces using the `unshare()` or `clone()` system calls with the `CLONE_NEWNS` flag. When a process moves to a new namespace or creates one, it will start using the mount points associated with that namespace.
4. **File descriptors and inodes are shared across namespaces**, meaning that if a process in one namespace has an open file descriptor pointing to a file, it can **pass that file descriptor** to a process in another namespace, and **both processes will access the same file**. However, the file's path may not be the same in both namespaces due to differences in mount points.

## Lab:

### Create different Namespaces

#### CLI

```bash
sudo unshare -m [--mount-proc] /bin/bash
```

By mounting a new instance of the `/proc` filesystem if you use the param `--mount-proc`, you ensure that the new mount namespace has an **accurate and isolated view of the process information specific to that namespace**.


#### Docker

```bash
docker run -ti --name ubuntu1 -v /usr:/ubuntu1 ubuntu bash
```

### &#x20;Check which namespace is your process in

```bash
ls -l /proc/self/ns/mnt
lrwxrwxrwx 1 root root 0 Apr  4 20:30 /proc/self/ns/mnt -> 'mnt:[4026531841]'
```

### Find all Mount namespaces

{% code overflow="wrap" %}
```bash
sudo find /proc -maxdepth 3 -type l -name mnt -exec readlink {} \; 2>/dev/null | sort -u
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name mnt -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```
{% endcode %}

### Enter inside a Mount namespace

```bash
nsenter -m TARGET_PID --pid /bin/bash
```

Also, you can only **enter in another process namespace if you are root**. And you **cannot** **enter** in other namespace **without a descriptor** pointing to it (like `/proc/self/ns/mnt`).

Because new mounts are only accessible within the namespace it's possible that a namespace contains sensitive information that can only be accessible from it.

### Mount something

```bash
# Generate new mount ns
unshare -m /bin/bash
mkdir /tmp/mount_ns_example
mount -t tmpfs tmpfs /tmp/mount_ns_example
mount | grep tmpfs # "tmpfs on /tmp/mount_ns_example"
echo test > /tmp/mount_ns_example/test
ls /tmp/mount_ns_example/test # Exists

# From the host
mount | grep tmpfs # Cannot see "tmpfs on /tmp/mount_ns_example"
ls /tmp/mount_ns_example/test # Doesn't exist
```

## References
* [https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory](https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory)



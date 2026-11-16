# Container internals

## Namespaces: what a process can see

A container is a regular Linux process. The kernel isolates it with namespaces and limits it with cgroups.

| Namespace | Isolates |
|---|---|
| pid | Process IDs |
| uts | Hostname |
| mnt | Filesystem mounts |
| net | Network interfaces, routes, ports |
| ipc | Shared memory |
| user | User and group IDs |
| cgroup | Cgroup view |

## PID namespace, verified

`sudo unshare --pid --fork --mount-proc bash` starts bash in a new PID namespace.

- Inside: bash is PID 1 and sees only itself and its children.
- On the host: the same process has a normal PID and a different namespace ID in `/proc/<pid>/ns/pid`.
- The host can signal or kill it at any time. This is what `docker kill` and pod deletion do underneath.
- If PID 1 exits, every process in the namespace is killed. That is why a container stops when its main process exits.

## Root filesystem: what a process sees as its disk

A container image is a folder holding a small Linux filesystem. I used Alpine's minirootfs (about 4 MB).

- `chroot` alone changes the files a process sees, but it still shares processes, hostname and network with the host. `ps` shows nothing because `/proc` is not mounted.
- Combined with namespaces, it becomes a container:

    sudo unshare --pid --fork --mount --uts --ipc --net chroot ~/rootfs /bin/sh

- Inside: Alpine files, `/bin/sh` as PID 1, its own hostname, only a loopback interface that is down.
- From the host: the hostname is unchanged, the container's `/proc` mount is invisible, and the shell runs under a normal host PID.
- Real runtimes such as runc use `pivot_root` instead of `chroot`, because a root process can escape a chroot.

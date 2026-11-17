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

## cgroups: how much a process can use

cgroups (v2) live under `/sys/fs/cgroup`. Every folder is a group. Creating a group is `mkdir`, setting a limit is writing to a file, and a process joins by writing its PID to `cgroup.procs`. Children inherit the group.

Test: a 50 MB limit on the hand-built container, then a process that keeps growing.

    echo 50M > /sys/fs/cgroup/box1/memory.max
    echo 0   > /sys/fs/cgroup/box1/memory.swap.max

- The kernel killed the process at about 49 MB. The container's PID 1 survived because it stayed small.
- `memory.events` showed `max 34`, `oom 1`, `oom_kill 1`. Kubernetes reads `oom_kill` to mark a pod `OOMKilled`, and the process exits with code 137 (128 plus SIGKILL).
- `dmesg` showed `constraint=CONSTRAINT_MEMCG`, which means a cgroup limit was hit, not a whole-node out-of-memory event.

Kubernetes mapping: `limits.memory` becomes `memory.max`, and `limits.cpu` becomes `cpu.max`.

## A real container, taken apart (containerd 2.2)

    sudo ctr run -d --memory-limit 52428800 docker.io/library/alpine:latest box2 sleep 3600

- **Process chain:** `containerd-shim-runc-v2` is the parent of the container process, and the shim's parent is PID 1, not containerd. Restarting containerd does not kill running containers. runc sets up namespaces and cgroups, starts the process, then exits.
- **Namespaces:** new pid, mnt, net, uts and ipc. Shared with the host: user (root inside is root on the host) and time (containers use the host clock).
- **Files:** overlayfs. `lowerdir` holds the read-only image layers, shared by every container from that image. `upperdir` is the container's own writable layer.
- **cgroup:** `/sys/fs/cgroup/default/box2/memory.max` holds the 52428800 limit. `default` is a containerd namespace. Kubernetes uses the `k8s.io` containerd namespace, so on a node use `ctr -n k8s.io`.
- **Exec:** `nsenter -t <PID> -a sh` joins every namespace of a running container. That is what `kubectl exec` does underneath.
- **Hostname:** a new UTS namespace starts as a copy of the host's hostname. Kubernetes sets it to the pod name.

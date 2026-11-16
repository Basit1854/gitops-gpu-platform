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

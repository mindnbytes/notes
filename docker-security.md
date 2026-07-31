# Docker Security

Docker security related notes.

Containers are very well isolated through kernel's namespaces and control groups. There are still some vectors of attack.
Kernel vulnerabilities - not something we actively care about, need to patch consistently.

Docker Daemon running as a root + services running under root inside a container lead to potential escape and privilege
escalation problems. So Docker daemon running under root is a significant surface of attack and we don't always need it
to be root. I want to explore and better understand rootless Docker daemon here.

## Rootless Docker

Rootless mode runs the daemon and containers from a user namespace. It works similar to `userns-remap` mode.
They differ as noted in how daemon runs and how they map container UIDs and GIDs to the host.

`dockerd` runs under user with their own `docker.sock`. Daemon still needs to create PID, network, mount namespaces,
which requires `CAP_SYS_ADMIN`. It utilizes user namespace, the only one, which can be created by non-root user.
One problem in user namespace user maps to nobody UID. Rootless Docker uses helper RootlessKit to bootstrap user namespace,
RootlessKit does the remapping of UID and GID with `newuidmap` and `newgidmap`, such that the process inside the container
appears to be root. Those helpers installed with `setuid` bit, which grants them permissions to temporarily run an executable
with permissions of file's owner - typically `root` - instead of the calling user. With those privileges granted, the daemon
can create all the nested namespaces it needs, because the kernel permits nested namespace creation when a user namespace is
the outermost boundary. Read this great [blogpost](https://www.kenmuse.com/blog/rootless-docker-and-its-hidden-security-trade-offs/)
for more details and clear explanations. Rootless BuildKit operates in the similar manner utilizing user namespace bootstrapped
by RootlessKit.

### User Remap

This part is for conceptual understanding of user remap. We shouldn't do it for the rootless install, as daemon itself is
remapped.

The main concept of `userns-remap` is to try to run as user inside a container when possible, but when it has to be root,
remap this user into a less-privileged user on the host. Conceptually, the mapped user is assigned a range of UIDs, which
function in the namespace as normal UIDs between 0 and 65536, but have no privileges on the host machine.

Remapping is handled by two files: `/etc/subuid` and `/etc/subgid` for the user IDs and group IDs respectively.
Example of the entry in such file is `testuser:231072:65536` means that `testuser` is assigned a subordinate user ID range
starting from 231072 and the next 65536 integers in sequence. UID 231072 is mapped within a namespace (within a container)
as UID 0 (root). If the process trying to escalate privilege outside the namespace, the process is running as a high-number
UID on the host, which doesn't even have corresponding user, meaning doesn't have any privileges on the host at all.

### Downsides of Rootless

- Running Docker requires logged-in user, work around with enabled lingering
- It doesn't allow exposing privileged ports, work around with [`cap_net_bind_service`](https://docs.docker.com/engine/security/rootless/tips/#exposing-privileged-ports)
- [Limiting resources](https://docs.docker.com/engine/security/rootless/tips/#limiting-resources) is supported only when running with `cgroup v2` and `systemd`
- If you need [troubleshooting](https://docs.docker.com/engine/security/rootless/troubleshoot/)

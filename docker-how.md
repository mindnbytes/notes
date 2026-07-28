# Docker How

Notes taken while working through Docker docs.

## Docker Engine

Note: On Debian and Ubuntu Docker routes the traffic around the ufw rules!
Consider publishing minimum required ports, reverse proxy can handle routing
the traffic to the localhost bound services.

### Install
- Uninstall old versions or unofficial packages
- Install according to official instructions
- Go rootless path: check security section of the documentation
- Follow post-installation steps for Linux

## Docker Storage

Writable container layer on top of read-only image layer. Data doesn't persist.

### Storage Mount Options

Doesn't matter what option you choose, date looks the same to the container. It is
exposed as a file or folder.

- Volume mounts:
    - Persistent storage, managed by Docker daemon,
    - On the host file system,
    - Mount to a container to interact,
    - Direct access is *unsupported* and *undefined* behavior,
    - Ideal for performance-critical data processing and long-term storage needs,
    - Gives raw, host level performance
- Bind mounts:
    - Direct link to the path on the host system,
    - Both, container, and host can modify it
    - Use when you need access from container and host
- Tmpfs mounts:
    - Ephemeral, in-memory,
    - Use when you don't need to persist data, 
    - Caching, sensitive, avoids I/O
- Named pipes - communicate between host and container

#### Volumes

Use cases:
- Easier to back up or migrate
- Work on Linux and Win
- More safely shared among multiple containers
- Can be pre populated by container or build
- High-performance I/O

It is better to write to the volume compared to the container writable layer.
It doesn't increase the size of container and doesn't require storage driver.
Storage driver provides union filesystem, using Linux kernel. This additional
abstraction reduces performance.

Volume can be mounted into multiple containers simultaneously.

Remove unused volumes with `docker volume prune`

Note: preexisting folder structure is obscured by the mount if you mount nonempty volume into it

Note: mounting empty volume into the folder propagates (copy) folders content into volume

You can create read-only volumes. You can mount into volume sub path.

See documentation on driver usage, backup, migrate, restore using volumes, etc.

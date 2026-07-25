# Notes on Containers and Docker

It is all about virtualization of a singular resource like CPU, RAM, storage, network into multiple of resources.
Why? You can probably share (multi-tenancy) and better utilize not used resources. But also, it is a separation of concerns into
different environments, especially if there is no need for app A and app B to sit in one shared environment.

## Virtual Machine

Virtual machine emulates an entire computer. It uses software to make this emulation possible (like hypervisor).
They are nice to run many OSes on, to safely test in the sandbox as VM is completely isolated. VM is a big file, so
one can easily snapshot it for backing up and recover to this snapshot later if something goes wrong. Or even bring
your VM on a different computer. Difference: VM has its own app, bin/lib, and OS. Container only app and bin/lib.

Full isolation from other VMs, still can be exploited. Interactive: which can be pro and cons. Think of VMs are more
prone to be used imperatively, and containers are more a declarative style. VM cons are speed and size.

## What Is a Container?

Virtual machine (VM) - emulates an entire computer. Container - only pretends to be a VM, but it is a sandboxed environment
rather than a full VM. Usually, containers run a stripped down (smallest) version of software (e.g. OS) to do a one job only.

Containers are lightweight and fast to develop and iterate. It is still vulnerable to exploits, which can potentially affect the
shared hardware. Public images are potential security risk. They have smaller attack surface and prevent configuration drift,
due to the lightweight, smaller size, fewer responsibilities, and declarative configuration.

Containers' main goal was granular control over processes using global resources (cgroups to limit, control, account for CPU, memory,
storage, network) with a virtualization as a mean to isolate the process/program into Linux namespaces. We can think of namespaces
and cgroups as a minimal and foundational instruments for the future development of container technology.

## How Containers?

Cgroups is mostly about control as a name implies, think limit and manage resources.
Namespaces are all about isolation, they make it possible for the process/program to have their own isolated view of resources as
if that process/program were an exclusive user of resources.

Container runtime = software managing container lifetime. Docker runtime uses containerd (high level) - container life cycle manager
and runc (low level, interacting with kernel cgroups and namespaces) for execution.

## Why Containers?

- Efficient use of shared resources
- Isolation improves security, less opportunity to corrupt shared data
- Isolation improves reproducibility: no dependency hell, no concerns of underlying operation system
- Isolation gives sane and granular management of the process/program
- Isolation allows pain-free testing (e.g. version upgrade) - you can duplicate your container and see how the new one is doing
- Isolation gives dependency resolution peace of mind - no shared dependencies and consequently dependency conflicts

## History

1979 - chroot, virtualizing/segregating the process/program. 2000 - FreeBSD jails for separation of its services and those of
its customers for security and ease of administration. 2001 - Linux VServer - jail mechanism which can partition resources
(file systems, network addresses, memory). 2006 - Google developed process containers for limiting, accounting, and isolating
resource usage of a collection of processes; renamed to control groups (cgroups) and merged into Linux kernel. 2008 - Linux
Containers - first container manager, uses Linux namespaces (isolate global resources: mounts, process trees, network) and
cgroups (limit and isolate CPU, memory etc.) 2013 - Docker - started from Linux Containers - moved to their own libcontainer lib
(Google). Ecosystem for container management has exploded the popularity.

## Docker

Docker is a way to build, run these containers, save them to templates etc. Docker container can run on many underlying operating
systems and don't care much on which one, Docker engine will translate. Container runs a service and usually exposes only required network
ports, making it hard target to exploit.

Saved containers called images. Same image is supposed to behave always the same way independently on the underlying environment.

Dockerfile is an instruction on how to build image, can be as simple as to download some other image and install a few packages on top.

Docker as an open-source platform allows building, deploying, running, updating, and managing containers.

Architecture: client-server. Docker client `docker` talks to the Docker daemon `dockerd`, which does building, running, distributing. Docker client can
connect to remote Docker daemon, client can communicate with more than one daemon. Docker Compose is a Docker client for an applications consisting of the
set of containers. Docker registries store Docker images. Docker Hub is public default Docker registry.

Objects. Image - read only template for creating the container. Dockerfile lets you define a set of steps needed to create an image and run it.
Dockerfile steps create layers, when changed and rebuilt, only changed layers are rebuilt. Container is a runnable instance of image.

## Apple `container`

- It is an Apple native Open Container Initiative (OCI) compatible container runtime.
- It runs every container in the lightweight Linux machine built on native Apple's Containerization and Virtualization frameworks.
- One VM for one container reduces the need to mount only container required host data paths.

Architecture: `container` CLI command communicates via `container-apiserver` - launch agent (daemon, service). It provides client API.
It launches `container-core-images` API for image management + manage local content store. It launches `container-network-vmnet` for virtual
network. For each container it launches another helper - `container-runtime-linux` which exposes a management API for the specific container.

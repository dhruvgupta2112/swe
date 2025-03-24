# _Core Concepts Behind Containers_ | CS305 (Software Engineering) | March 2025 

Containers are essentially lightweight, isolated environments running on a single host OS kernel. They achieve this through a combination of:

1.	Namespaces – Provide isolation
2.	Cgroups (Control Groups) – Manage resource allocation
3.	Union File Systems – Provide lightweight, layered filesystems
4.	Container Runtime – Manages container lifecycle


## Namespaces – Isolation of Processes

Namespaces are a Linux kernel feature that allows containers to have their own isolated view of system resources, such as:

| Namespace	 | Description 
| ---------- | ------------
| PID        | Isolates process IDs (container sees only its own processes)|
| NET        | Provides isolated network interfaces (container has its own IP stack)
| MNT        | Provides isolated filesystem mount points
| UTS        | Isolates hostname and domain name
| IPC        | Isolates interprocess communication
| USER       | Isolates user IDs and group IDs

When a container is started, Docker creates new namespaces so that the container operates in its own sandboxed environment.


## Cgroups – Resource Allocation and Limiting

Control groups (cgroups) allow the kernel to limit and monitor resource usage by containers:
- CPU – Limit CPU cores or time
- Memory – Set maximum memory usage
- I/O – Limit disk and network throughput
- Network – Control network bandwidth usage

When you run a container with `--memory 512M`, Docker creates a cgroup for the container and assigns the limit to it.


## Union Filesystem – Layered Filesystem

Docker uses a union filesystem (e.g., [OverlayFS](https://docs.kernel.org/filesystems/overlayfs.html), AUFS) to manage container images and files:
- Base image forms the bottom layer.
- User changes (like installed packages) are stored as read-write layers on top.
- Changes are stored as copy-on-write (COW) — minimizing disk usage and improving speed.

When you start a container, Docker creates a new thin writable layer on top of the image's filesystem.


## Container Runtime – Lifecycle Management

Docker itself doesn't directly run containers — it uses a container runtime like:

- Low-Level Runtimes:
    - [runc](https://github.com/opencontainers/runc) – Implements the OCI (Open Container Initiative) runtime standard.
    - [containerd](https://github.com/containerd/containerd/blob/main/docs/getting-started.md) – Manages the lifecycle of containers (start, stop, restart, etc.).

- High-Level Management:
    - Docker Engine talks to containerd (or cri-o in Kubernetes).
    - containerd launches runc to set up namespaces, cgroups, and filesystem layers.


## Docker Workflow Overview

When you run `docker run my-image`:
1. Docker CLI sends the request to Docker Daemon.
2. Docker Daemon:
    - Pulls the image (if not present).
    - Calls containerd to manage the container.
3.	containerd:
    - Creates new namespaces.
    - Assigns cgroups.
    - Sets up the filesystem using union FS.
    - Calls `runc` to execute the container.
4.	The containerized process runs within its isolated namespace and cgroup.

## Networking – How Containers Connect

Docker sets up an isolated virtual network using:
- Bridge Network – Containers communicate within the same network.
- Host Network – Container shares the host's network stack.
- Overlay Network – Multi-host networking using vxlan.

For example, Docker creates a virtual Ethernet interface for each container and connects it to a bridge (like `docker0`).


## Container Image – What Makes Up an Image

A Docker image is made up of:
- Base Layer – OS + minimal tools (ubuntu, alpine)
- Additional Layers – Installed packages, configurations
- Manifest – Metadata about the image

When you docker build:
- Each `RUN`, `COPY`, `ADD` creates a new layer.
- Layers are cached to speed up builds.

### Summary Flow
1.	Docker CLI → Docker Daemon
2.	Docker Daemon → containerd → runc
3.	runc sets up:
    - Namespaces
    - Cgroups
    - Filesystem
4.	Kernel isolates and manages the container
5.	Container runs as a standard Linux process under isolation


### Why Are Containers Fast?
- No hardware emulation like VMs.
- Directly uses host OS kernel.
- Fast creation and deletion due to layered filesystem and copy-on-write.
- Lightweight because of shared kernel.


### Difference Between Containers and VMs

| Feature | Containers	| Virtual Machines
| -------- | ------ |-----
| Isolation	| Process-level (namespace, cgroup) | 	Hardware-level (hypervisor)
| Startup Time	| Milliseconds |	Minutes
| Size	| MBs	| GBs
| Performance	| Near-native |	Slight overhead
| Compatibility	| Same kernel as host	| Any OS



## Deep Dive Example

You can simulate how Docker works manually:
1.	Create a namespace:
`unshare -p -f --mount-proc bash`

2.	Assign cgroups:
`echo 512M > /sys/fs/cgroup/memory/demo/memory.limit_in_bytes`

3.	Mount filesystem:
`mount -t overlay overlay -o lowerdir=/lower,upperdir=/upper,workdir=/work /merged`

4.	Start process in isolated namespace:
`chroot /merged /bin/bash`

## How containers  safely share the host kernel
1. Containers Share the Kernel, Not the User Space

### What is shared:
- The container uses the host kernel for system calls (like process scheduling, memory management, file I/O).
- Since the kernel is shared, containers don't have to ship their own kernel — which makes them lightweight and fast.

### What is NOT shared:
- User space (libraries, binaries, etc.) is NOT shared.
- Each container has its own isolated root filesystem.
- Libraries and binaries inside the container are separate from those on the host.


### How Conflicts Are Avoided

Containers avoid conflicts through a combination of:

**(a) Namespaces – Isolate System-Level Resources**

Each container operates in its own set of namespaces, so it "thinks" it's the only process running:

| Namespace | Effect
| --------- | -------
| PID   | Containers see their own process IDs only.
| MNT   | Containers have their own filesystem view.
| NET   | Containers have their own network stack (IP, ports).
| UTS   | Containers can have unique hostnames.
| IPC   | Containers have separate inter-process communication.
| USER  | Containers have their own user IDs and groups.

Example:
- Container A can have PID 1 (`init` process) even if the host already has a PID 1.
- Container A can have hostname foo, and container B can have hostname bar — independent of the host's hostname.


**(b) Cgroups – Control Resource Usage**

Cgroups ensure that containers don't hog system resources or interfere with each other:
- Set limits on CPU, memory, network, and I/O.
- Prevent a container from consuming all available resources and affecting the host.

_Example:_
Container A can be limited to 2 CPU cores and 1 GB of memory, even if the host has 8 cores and 16 GB of memory.


**(c) Union Filesystem – Isolated Filesystem View**

Containers use a union filesystem (e.g., overlayfs) to provide an isolated root filesystem:
- Each container starts from a base image (like ubuntu or alpine).
- The container's filesystem is composed of:
- Base layer → Immutable shared files
- Read-write layer → Container-specific changes
- Changes in the container are only visible to that container.

_Example:_ Container A and B can both be based on ubuntu, but if A installs vim, it won't appear in B because they have separate read-write layers.

**(d) Root Filesystem is Mounted in a Private Namespace**

When a container starts, Docker mounts the container's root filesystem using a private namespace:

`mount --make-rprivate /`

This ensures that the container's filesystem changes don't affect the host or other containers.

_Example:_ If container A deletes `/bin/bash`, the host and other containers are unaffected.


**(e) Library Version Conflicts Are Avoided by Layering**

Each container brings its own libraries (like `glibc`, `openssl`) within its filesystem:
- Containers are built from an image that includes the necessary libraries and binaries.
- The container uses the libraries inside its own filesystem, not the host's libraries.

_Example:_ Host may have `glibc` 2.28, but container can run `glibc` 2.34 without conflict because it's using the container's version.


## Why No Kernel Conflicts?

The kernel provides backward compatibility and ABI (Application Binary Interface) stability:
- User-space programs interact with the kernel using system calls (like `read()`, `write()`), not direct kernel code.
- As long as the kernel supports the system calls, containers with different libraries or binaries will work.

_Example:_ Container A can use `glibc` 2.28 and container B can use `musl` — as long as the kernel supports the underlying syscalls, both will work.


### Example Scenario

- Host Setup:
    - Host kernel: Linux 5.10
    - Host glibc: 2.31

- Container A:
    - Based on Alpine Linux
    - glibc: 2.34
    - OpenSSL: 1.1.1

- Container B:
    - Based on Ubuntu
    - glibc: 2.28
    - OpenSSL: 1.0.2

- What Happens:
    - Both containers run without conflict because they use their own `glibc` and their own OpenSSL.
    - Both containers use the same kernel for syscalls.
    - Kernel handles the system calls independently for each container using namespace isolation.


### Testing This Isolation

You can prove this isolation:
1.	List host processes: `ps aux`

2.	List container processes: `docker exec -it container_id ps aux`

    - The container will only show processes inside its namespace.

3.	Check glibc version on the host: `ldd --version`

4.	Check glibc version in the container: `docker exec -it container_id ldd --version`

    - The versions can differ without conflict!



## Containers Can be better than VMs

| Feature | Containers | Virtual Machines
| --- | --- | ----
| Kernel    | Shared with host  | Separate per VM
| Size  | MBs   | GBs
| Startup Time  | Milliseconds  | Minutes
| Performance   | Near native   | Hardware emulation overhead
| Compatibility | Same kernel as host   | 	Different OS possible



## What does it mean for a container to have its own IP stack?

1. **Separate Network Namespace**
- Each container runs in its own network namespace (created using `CLONE_NEWNET`).
- A network namespace creates a separate virtual instance of:
- IP addresses
- Network interfaces (like `eth0`, `lo`)
- Routing tables
- Firewall rules
- Port bindings

2. **Independent IP Address**
- The container gets an IP address that is:
- Assigned by Docker (or Kubernetes).
- Can be from a private network (172.x.x.x, 10.x.x.x) or an overlay network (if using Swarm or Kubernetes).

3. **Own Interfaces (Virtual)**
- The container has its own network interface (usually `eth0` inside the container).
- The container also has a local loopback (`lo`) interface for local communication.
- The interface is connected to the host via a virtual Ethernet pair (`veth`).

4. **Own Routing Table**
- The container has its own routing table, which determines:
- How traffic flows between containers.
- How traffic flows to the host or external network.

5. **Own Firewall Rules**
- Containers can have their own iptables or nftables rules inside the network namespace.
- They can block or allow traffic independently of the host.

### How It Works Internally

**Step 1: Virtual Ethernet Pair**

When a container starts, Docker creates a virtual Ethernet pair (veth):
- One end (`veth0`) is attached to the container's `eth0` interface.
- The other end is attached to a bridge on the host (like `docker0`).

**Step 2: IP Assignment**

Docker assigns an IP address to the container's `eth0` interface, e.g.,:

`docker network inspect bridge`

Example output:
```
"Containers": {
    "abcdef1234": {
        "IPv4Address": "172.17.0.2/16"
    }
}
```
**Step 3: Routing and NAT**
- Traffic from the container is routed via the virtual Ethernet pair to the host.
- If the container wants to reach the internet, the host performs NAT (Network Address Translation) using iptables:
    - Container (172.17.0.2) → veth0 → docker0 (bridge) → Host NIC → Internet

**Step 4: Bridge or Overlay Network**
- Containers can talk to each other directly over a bridge (local) or overlay (multi-host).
- Docker creates a docker0 bridge by default and attaches containers to it.

**Step 5: Firewall and Isolation**
- Containers are firewalled from each other using iptables or nftables.
- A container cannot access other containers unless explicitly allowed.


## Example Scenario

### Setup:
1.	Host IP: 192.168.1.100
2.	Container A IP: 172.17.0.2
3.	Container B IP: 172.17.0.3

### How They Talk:
- If Container A sends a packet to Container B:
- Traffic goes through the virtual Ethernet pair to the docker0 bridge.
- Docker bridge forwards the packet to Container B using its routing table.

### How They Talk to the Outside World:
- If Container A wants to access google.com:
- Traffic goes from eth0 → veth0 → docker0 → Host NIC.
- Host NATs the traffic to its own IP (192.168.1.100).
- Return traffic is NATed back to the container's IP (172.17.0.2).

### Why This Works Without Conflict
1.	Separate network namespace = Containers "think" they have their own network stack.
2.	Virtual Ethernet pair = Isolated pipe to the host.
3.	NAT = Host remaps container traffic to avoid IP conflicts.
4.	Separate firewall = Prevents containers from interfering with each other.


## Proof of Isolation

You can check the container's network stack using the following:

1. List the container's network interfaces:
`docker exec -it <container_id> ip a`

2. Check container's routing table:
`docker exec -it <container_id> ip route`

3. Check open ports inside the container:
`docker exec -it <container_id> netstat -tuln`

4. Block network access inside a container using firewall rules:
`docker exec -it <container_id> iptables -A INPUT -p tcp --dport 80 -j DROP`

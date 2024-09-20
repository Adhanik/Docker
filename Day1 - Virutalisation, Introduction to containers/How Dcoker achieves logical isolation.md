

To provide a better picture of files and folders that containers base images have and files and folders that containers use from host operating system (not 100 percent accurate -> varies from base image to base image). Refer below.

### Files and Folders in containers base images

    /bin: contains binary executable files, such as the ls, cp, and ps commands.

    /sbin: contains system binary executable files, such as the init and shutdown commands.

    /etc: contains configuration files for various system services.

    /lib: contains library files that are used by the binary executables.

    /usr: contains user-related files and utilities, such as applications, libraries, and documentation.

    /var: contains variable data, such as log files, spool files, and temporary files.

    /root: is the home directory of the root user.

### Files and Folders that containers use from host operating system

    The host's file system: Docker containers can access the host file system using bind mounts, which allow the container to read and write files in the host file system.

    Networking stack: The host's networking stack is used to provide network connectivity to the container. Docker containers can be connected to the host's network directly or through a virtual network.

    System calls: The host's kernel handles system calls from the container, which is how the container accesses the host's resources, such as CPU, memory, and I/O.

    Namespaces: Docker containers use Linux namespaces to create isolated environments for the container's processes. Namespaces provide isolation for resources such as the file system, process ID, and network.

    Control groups (cgroups): Docker containers use cgroups to limit and control the amount of resources, such as CPU, memory, and I/O, that a container can access.
    

It's important to note that while a container uses resources from the host operating system, it is still isolated from the host and other containers, so changes to the container do not affect the host or other containers.

Note: There are multiple ways to reduce your VM image size as well, but I am just talking about the default for easy comparision and understanding.

so, in a nutshell, container base images are typically smaller compared to VM images because they are designed to be minimalist and only contain the necessary components for running a specific application or service. VMs, on the other hand, emulate an entire operating system, including all its libraries, utilities, and system files, resulting in a much larger size.

I hope it is now very clear why containers are light weight in nature.



Docker achieves logical isolation between containers primarily through the use of several key technologies and concepts:

1. **Namespaces**: Docker uses Linux namespaces to provide isolation. Each container runs in its own namespace, which ensures that resources like process IDs, network interfaces, and user IDs are unique to that container. The main types of namespaces Docker uses include:
   - **PID namespace**: Isolates process IDs.
   - **Network namespace**: Provides separate network stacks for each container.
   - **Mount namespace**: Ensures each container has its own file system view.

2. **Control Groups (cgroups)**: Docker employs cgroups to limit and prioritize resource usage (CPU, memory, disk I/O) for each container. This prevents any single container from consuming too many resources, helping maintain performance and stability across the system.

3. **Union Filesystems**: Docker uses a union filesystem (such as OverlayFS) to provide a layered file system. Each container can have its own file system layer that can be built on top of a base image, allowing for efficient storage and isolation of file changes.

4. **Networking**: Docker provides virtual networks that allow containers to communicate with each other while remaining isolated from the host network and other containers unless explicitly allowed. This is managed through the use of virtual Ethernet bridges.

5. **Security Features**: Docker incorporates several security mechanisms, such as:
   - **AppArmor and SELinux**: These provide mandatory access controls that restrict how containers interact with the host system and each other.
   - **User namespaces**: Allow for mapping container users to different users on the host, enhancing security.

By leveraging these technologies, Docker ensures that each container operates in its own isolated environment, making it possible to run multiple applications on the same host without interference.
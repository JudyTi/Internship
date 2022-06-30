# General Concepts
### Network
+ **Protocol**: a set of rules for data transm ission which are agreed by sender and receiver
+ TCP/IP is th e dominant protocol su ite for Internet usage.

<img src="./images/network_model.png" alt=network_model width="500"/>

### OS
+ Difference between **Operating system(OS)** and **Kernel**:
   - **Operating system(OS)** 
      * Def: system program that runs on the computer to provide an interface to the computer user so that they can easily operate on the computer
      * The package of data and software that manages the resources of the system
      * In addition to the responsibilities of Kernel, OS is also responsible for **protection and security of the computer**
   - **Kernel**
      * Def: a system program that controls all programs running on the computer
      * The important program in the OS
      * **A bridge between software and hardware of the system**

### Container
Container is just a **process** on the machine uses **namespaces** and **control groups (cgroup)** to provide isolation  

![container evolution](/images/container_evolution.svg)

+ Difference between **namespaces** and **control groups (cgroup)**:
   - **namespaces**
      * Def: a feature of the Linux kernel that partitions kernel resources such that one set of processes **sees** one set of resources while another set of processes sees a different set of resources
      * Limits **what you can see**
   - **control groups (cgroup)**
      * Def: a Linux kernel feature that limits, accounts for, and isolates the **resource usage** (CPU, memory, disk I/O, network, and so on) of a collection of processes
      * Limits **how much you can use**
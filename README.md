# Environment Install

In this lab virtual machines running on VMware workstation will be used. These machines will be considered as baremetal computers. Two machines will be installed as:

- A single node openshift cluster
- A bastion and development machine

Following table summarizes system and software properties for both:

|  | Development | Single Node OpenShift |
| --- | --- | --- |
| CPU [Cores] | 4 | 8 |
| Memory [GB] | 8 - no swap | 32 |
| Storage [GB] | 1000 - mounted as / | 400 |
| OS | Red Hat Enterprise Linux 9.2 | RHCOS |
| Users | root | core |
| Installed Software | coredns [container]<br>podman<br>go<br>git<br>openshift cli | NA |
| Hostname | trip-dev | trip-ocp |
| Domain | tuff.local | tuff.local |
| IP | 192.168.1.140/24 10.10.221.254/24 | 10.10.221.5/24 |
| Gateway | 192.168.1.1 | 10.10.221.254 |
| DNS | 10.10.221.254 | 10.10.221.254 |

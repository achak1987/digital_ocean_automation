# Setting up remote host machines on digital ocean
In this project we setup the relevent infrastructure for prroject fabric-as-code and mysome_gluster
Please see the following subsections for details

## Setting up host machines for fabric-as-code [https://github.com/achak1987/fabric_as_code]
- Please navigate to the file `inventory/hosts_template`
- It looks as follows:
```
[all:children]
swarm_manager_prime
swarm_managers
swarm_workers

[swarm_manager_prime]


[swarm_managers]


[swarm_workers]


```
- In order the specify the host machines, you need to populate this file `inventory/hosts_template` with the names of the host that you want to create. Each line/row in the file would represent a host machine. The lines with square brackets  `[]` represents groups for internal reference in the project and **must not be changed**. Please fill each line under a group in the format: `hostname"`
  - `swarm_manager_prime`: represents the host machine that will act as swarm master
  - `swarm_managers`: represents the backup masters
  - `swarm_workers`: represents the workers
  - `hostname`: can be any name. Must be unique for each machine. The project will internally refer to the machines with this name
- The following *example* defines 5 machines as remote hosts, with hlf0 acting as master, hlf1 as backup master and hlf[2-4] as swarm workers
```
[all:children]
swarm_manager_prime
swarm_managers
swarm_workers

[swarm_manager_prime]
hlf0

[swarm_managers]
hlf0
hlf1

[swarm_workers]
hlf2
hlf3
hlf4
```

## Setting up host machines for mysome_glusterfs [https://github.com/achak1987/mysome_glusterfs]
- Please navigate to the file `inventory/hosts_template`
- It looks as follows:
```
[gfscluster]



```
- In order the specify the host machines, you need to populate this file `inventory/hosts_template` with the names of the host that you want to create. Each line/row in the file would represent a host machine. The root/first line `[gfscluster]` gives a name to the cluster for internal reference in the project and **must not be changed**. Please fill each line in the format: `hostname ansible_python_interpreter="/path/to/python"`
  - `hostname`: can be any name. Must be unique for each machine. The project will internally refer to the machines with this name
  - `ansible_python_interpreter`: In order for ansible to work, we need python 2.7.x or above available on each remote machine. Here we specify the **path of python on the remote machine** so that our local ansible project know where to find python on these machines.
- The following *example* defines 3 machines as remote hosts
```
[gfscluster]
gfs1 ansible_python_interpreter="/usr/bin/python3"
gfs2 ansible_python_interpreter="/usr/bin/python3"
gfs2 ansible_python_interpreter="/usr/bin/python3"
```


## Creaing Droplets on DO

- Next we would run the following playbooks to create
    - the number of VMs / Droplets
    - a block mount on each machines
- Playbook: `000.init.yml`
    - Execute: `ansible-playbook -v 000.init.yml`
    - Create a host file  inside inventory that would be filled up with the ips of the created host machines
- Playbook: `001.spawn_droplets.yml`
  - Execute: `ansible-playbook -v 001.spawn_droplets.yml`
  - Creates the specified number of host machines on digital ocean.

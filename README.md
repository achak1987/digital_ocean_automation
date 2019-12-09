# Setting up remote host machines on digital ocean
In this project we setup the relevent infrastructure for prroject fabric-as-code and mysome_gluster.

**All the remote hosts machines are setup in Digital Ocean**

Please see the following subsections for details

## Local Configuration
There are very few parameters to be configured currently. All configurations are made inside `group_vars/all.yml`. 
- If you are using the automated process for host setup (*see bellow*), it needs few steps to enable ansible to setup the remote environment
  - **API token**
    - In order to connect with Digital Ocean. 
    - Please see [https://www.digitalocean.com/docs/api/create-personal-access-token/]
    - Once you get the token, please find `do_oauth_token` in `all.yml` and set its value there
  - **SSH Keys**
    - In order for setting up the cluster, ansible needs ssh password less login to the host machines.
    - Your ssh public key from your local machines should be registered with digital ocean, see [https://www.digitalocean.com/docs/droplets/how-to/add-ssh-keys/to-account/] 
    - Once you have the ssh keys registered with Digital Ocean, you need to retrived ths ssh key ids from digital ocean. You can execute the following command on the bash on your local machine to get  this ids: `curl -X GET -silent "https://api.digitalocean.com/v2/account/keys" -H "Authorization: Bearer API token`
    - This will get a list of all ssh keys stored to your Digital Ocean account. You need to find the **numeric id** associated with your ssh key name. 
    - Once you get the ssh key id, please find `ssh_keys` in `all.yml` and set its value there within the `[]` brackets

## Configurations for Host Machines 
For each of the projects (fabric-as-code or mysome_glusterfs) we need to select one of the following *configutation* and run the playbooks defined in the *Creaing Droplets on DO*  section bellow

### for **mysome_glusterfs** [https://github.com/achak1987/mysome_glusterfs]
- Please navigate to the file `inventory/hosts_template`
- fill this file with the following structure:
```
[gfscluster]



```
- In order the specify the host machines, you need to populate this file `inventory/hosts_template` with the names of the host that you want to create. Each line/row in the file would represent a host machine. The root/first line `[gfscluster]` gives a name to the cluster for internal reference in the project and **must not be changed**. Please fill each line in the format: 

`hostname ansible_python_interpreter="/path/to/python"`
  - `hostname`: can be any name. Must be unique for each machine. The project will internally refer to the machines with this name
  - `ansible_python_interpreter`: In order for ansible to work, we need python 2.7.x or above available on each remote machine. Here we specify the **path of python on the remote machine** so that our local ansible project know where to find python on these machines.
- The following *example* defines 3 machines (gfs[1-3]) as remote hosts
```
[gfscluster]
gfs1 ansible_python_interpreter="/usr/bin/python3"
gfs2 ansible_python_interpreter="/usr/bin/python3"
gfs3 ansible_python_interpreter="/usr/bin/python3"
```
### for **fabric-as-code** [https://github.com/achak1987/fabric_as_code]
- Please navigate to the file `inventory/hosts_template`
- fill this file with the following structure:
```
[all:children]
swarm_manager_prime
swarm_managers
swarm_workers

[swarm_manager_prime]


[swarm_managers]


[swarm_workers]


```
- In order the specify the host machines, you need to populate this file `inventory/hosts_template` with the names of the host that you want to create. Each line/row in the file would represent a host machine. The lines with square brackets  `[]` represents groups for internal reference in the project and **must not be changed**. Please fill each line under a group in the format: 

`hostname ansible_python_interpreter="/path/to/python"`
  - `swarm_manager_prime`: represents the host machine that will act as swarm master
  - `swarm_managers`: represents the backup masters
  - `swarm_workers`: represents the workers
  - `hostname`: can be any name. Must be unique for each machine. The project will internally refer to the machines with this name

- The following *example* defines 5 machines as remote hosts called hlf[0-5], with hlf0 acting as master, hlf1 as backup master and hlf[2-4] as swarm workers
```
[all:children]
swarm_manager_prime
swarm_managers
swarm_workers

[swarm_manager_prime]
hlf0 ansible_python_interpreter="/usr/bin/python3"

[swarm_managers]
hlf0 ansible_python_interpreter="/usr/bin/python3"
hlf1 ansible_python_interpreter="/usr/bin/python3"

[swarm_workers]
hlf2 ansible_python_interpreter="/usr/bin/python3"
hlf3 ansible_python_interpreter="/usr/bin/python3"
hlf4 ansible_python_interpreter="/usr/bin/python3"
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

- Copy hosts file to `inventory/hosts` for the respective project

## Removing Droplets on DO
- `inventory/hosts` file should be present
- The created `hostname` should listed in the hosts file
- example:
```
[gfscluster]
gfs0 ansible_host=167.71.42.51 ansible_python_interpreter=/usr/bin/python3
gfs1 ansible_host=165.22.203.52 ansible_python_interpreter=/usr/bin/python3
```
- Playbook: `001.despawn_droplets.yml`
    - Execute: `ansible-playbook -v 001.despawn_droplets.yml`
    - Removes the droplets in Digital Ocean as defined in `inventory/hosts` file

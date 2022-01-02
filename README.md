# Slurm-basic-cluster-configuration

## Basic configs
In this example controller and all nodes will have a user with a username `slurming`.

This is an example of what the hostnames of all nodes should look like:
- _controller_ - main node of the cluster
- _node_[0-x] - other nodes of the cluster
To change node's hostname to _example_:
- change the value to `/etc/hostname` to _example_ and save it (`sudo nano /etc/hostname`)
- change the value from the previous hostname in `/etc/hosts` to `xxx.x.x.x _example_` and save it (`sudo nano /etc/hosts`)
- `sudo reboot`

On the controller node:
- `sudo apt install net-tools`
- `ifconfig`
- copy _inet_ ip address for later

For each node:
- `sudo apt install net-tools`
- `ifconfig`
- copy _inet_ ip address
- add the CONTROLLER's _inet_ to `/etc/hosts` file, like: `192.168.91.134 controller`
- add the _inet_ to the controller's `/etc/hosts` file, like: `192.168.91.135 node1`


## passwordless SSH
On each node:
- `sudo apt install openssh-server`
- `sudo visudo`
- under the _#includedir /etc/sudoers.d_ add `slurming ALL=(ALL) NOPASSWD: ALL`
- save the file

On the controller node:
- `sudo apt install openssh-server`
- `ssh-keygen -t rsa -b 4096 -C "your-email@address.com` where _your-email@address.com_ is your email address
- keep hitting `enter`
- `ssh-copy-id nodeX` for each node, where nodeX is the node's name
- check if everything works via `ssh nodeX sudo hostname`, which should return a hostname of the node without any prompts


## Munge
On the controller node:
- `sudo apt install munge`
- `create-munge-key`
- `sudo systemctl start munge`

On each node:
- `sudo apt install munge`
- `sudo systemctl start munge`

Now we have to copy the CONTROLLER's munge key to each node. We will use some tricks to avoid permission problems.
On the controller node:
- `cd ~`
- `sudo cp /etc/munge/munge.key /`
- `sudo chown slurming:slurming munge.key`
- `sudo chmod 777 munge.key`
- `scp munge.key nodeX:~` for each node, where nodeX is the node's name

On each node:
- `cd ~`
- `sudo chown root:root munge.key`
- `sudo chmod 400 munge.key`
- `sudo chown 128 munge.key`
- `sudo mv munge.key /etc/munge`
- `sudo systemctl restart munge`


## Slurm
On the controller AND each node: 
- `sudo apt install slurm-wlm`
- copy the _slurm.conf_ file to `/etc/slurm-llnl` (you should edit file's Nodes according to your resources, use `slurmd -C` on each node and the controller)

On the controller node:
- `sudo systemctl start slurmd`
- `sudo systemctl start slurmctld`

On each node:
- `sudo systemctl start slurmd`

# Workshop-Environment

The Ansible playbooks in this repository will create a **demo environment** comparable to the *Ansible Workshop* in the *Red Hat Demo environment*, but instead of virtual machines, it will create Podman containers acting as the *managed nodes*.  
The demo environment will mimic the RH Demo Environment with three managed nodes: **node1**, **node2** and **node3**. The Ansible control node will be known as **ansible-1** in the lab inventory.  
Although containers are used, all managed nodes are accessible via SSH, a couple of ports are exposed with every container:

| Managed node | Port 80 | Port 8080 |
| ------------ | ------- | --------- |
| node1        | 8002    | 8003      |
| node2        | 8005    | 8006      |
| node3        | 8008    | 8009      |

For example, if you want to access a webserver running on port 8080 on node2, you'll need to access it via `http://localhost:8006`.

> NOTE: If any ports in the range of 8001 to 8009 are already occupied, deployment will fail! You can adjust the ports to be used in the `inventory.ini`.  

If you want to resolve the hosts (containers) with their hostname, adjust your `/etc/hosts`:

```ini
127.0.0.1 localhost node1 node2 node3
```

You still need to add the port, but now you can run `curl http://node1:8002`.

The SSH port for every container is also exposed via a high port (*node1* on *8001*, *node2* on *8004* and *node3* on *8007*), but you can access the managed nodes container like this:

```console
ssh node1
```

This is achieved by the addition of a block to your personal `~/.ssh/config`. A SSH keypair is created for the Workshop (stored as `~/.ssh/ansible-workshop-environment` and `~/.ssh/ansible-workshop-environment.pub`).

As with the RH Demo environment, you will find your *Ansible Workshop inventory file* at `~/lab_inventory/hosts`. An Ansible configuration file is created at `~/.ansible.cfg`.

> NOTE: If you already have a config file at this location, a backup is created.

## Create Workshop environment

To create the managed node containers, run the `create-workshop-environment` playbook:

```console
ansible-playbook create-workshop-environment.yml
```

The playbook checks if the Podman container runtime is installed. If it is not present, the playbook will try to install it, this task is the only one where *sudo* permissions are necessary. Either run the playbook with `--ask-become-pass` (if you need to provide a sudo password by default) or install Podman manually. For Ubuntu or Debian, use the following command:

```console
sudo apt-get update && sudo apt-get -y install podman  
```

Take a look at the [Podman documentation](https://podman.io/docs/installation#linux-distributions) for additional information and installation instructions for different distributions.

The playbook will do the following steps:

1. Install Podman (if necessary).
2. Pull a Docker image and start as many containers as are defined in the inventory.
3. Deploy the *Workshop inventory* (not the same as is used by this playbook!) to `~/lab_inventory/hosts`.
4. Add a block with SSH config to access the managed nodes to `~/.ssh/config`.
5. Deploy the Ansible configuration for the Workshop to `~/.ansible.cfg`.
6. Create a SSH keypair for the Workshop (`~/.ssh/ansible-workshop-environment` and `~/.ssh/ansible-workshop-environment.pub`).
7. Prepare all managed nodes.

## Delete Workshop environment

After completing the workshop, you can remove all traces of the demo environment by running the `delete-workshop-environment` playbook:

```console
ansible-playbook create-workshop-environment.yml
```

The playbook will do the following steps:

1. Delete the managed node containers.
2. Delete the folder with the workshop inventory (`~/lab_inventory/hosts`).
3. Remove the managed nodes from `~/.ssh/known_hosts` if they were accessed manually.
4. Delete the block from `~/.ssh/config` which was added for the workshop.
5. Delete the Ansible configuration from `~/.ansible.cfg`, if a backup was created, a hint is shown.
6. Delete the keypair created for the workshop (`~/.ssh/ansible-workshop-environment` and `~/.ssh/ansible-workshop-environment.pub`).

If you want to delete the Podman installation as well, run the playbook like this:

```console
ansible-playbook delete-workshop-environment.yml -e delete_podman=true
```
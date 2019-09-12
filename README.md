# Ansible playbooks to deploy Kubernetes

The ansible playbooks are able to deploy a kubernetes cluster with
Windows minion nodes and Flannel as SDN solution.
It supports docker or containerd as a container runtime.

## Ansible requirements

The recommended version is `2.7.2`.

For Linux: Make sure that you are able to SSH into the target nodes without being
asked for the password. You can read more [here](http://docs.ansible.com/ansible/latest/user_guide/intro_getting_started.html).

For Windows: Follow [this guide](https://docs.ansible.com/ansible/devel/user_guide/windows_setup.html)
to setup the node to be used with ansible.

#### Setup env

##### Populate your [hosts](/inventory/hosts) inventory file with the hostnames and IPs of your machines.

```
[kube-master]
test-master

[kube-minions-windows]
test-min0
test-min1

```

##### Update Linux machies username in [group_vars](/inventory/group_vars) in the kube-master and kube-minions ( if any ) vars files.

```
ansible_ssh_user: ubuntu
```

##### Update Windows machines settings in  [group_vars](/inventory/group_vars) ( kube-minions-windows).

```
ansible_port: 5986 // winrm port
ansible_connection: winrm // type or ansible connection for remoting
ansible_user: 
ansible_password:
CONTAINER_RUNTIME: docker | containerd 
containerd_url: url to zip containerd binaries
```

##### Set the flannel mode in [inventory/group_vars/all](/inventory/group_vars/all):

```
FLANNEL_MODE: "overlay" | "host-gw"
```

##### Set the CNI binaries in [inventory/group_vars/all](/inventory/group_vars/all):

```
CNIBINS: "cniwin" | "sdnms"
```

NOTE: 

cniwin - refers to the win-bridge and win-overlay cni plugins maintainerd in github.com/containernetworking/plugins
sdnms  - refers to the sdnbridge and sdnoverlay cni plugins maintained by Microsoft in github.com/Microsoft/windows-container-networking

#### Verifying the setup

To verify the setup and that ansible has been successfully configured you can run the following:

```
ansible --key-file=ssh_private_key_path -m setup all
```

This will connect to the target hosts and will gather host facts.
If the command succeeds and everything is green, you're good to go with running the playbook.

## How to use

Make sure to update first the [inventory](/contrib/inventory) with
details about the nodes.

To start the playbook, please run the following:
```
ansible-playbook -i inventory/hosts kubernetes-cluster.yml --private-key=ssh_private_key_path
```

## NOTES

- All binaries used by the playbooks will be downloaded from locations specified in the vars. If you want to use your custom
builds ( recommended ) you should have them present on the ansible machine in the ansible tmp folder (ansible_root/tmp). Ansible will take care of copying them to the k8s master / k8s minion nodes.

Currently supported Linux nodes:
- 16.04

Currently supported Windows nodes:
- Windows Server 2019 LTSC and build version 1809 (OS Version 10.0.17763.0)

#### Ports that have to be opened on public clouds when using the playbooks

The following ports need to be opened if we access the cluster machines via the public address.

##### Kubernetes ports

- Kubernetes service ports (deployment specific): UDP and TCP `30000 - 32767`
- Kubelet (default port): TCP `10250`
- Kubernetes API: TCP `8080` for HTTP and TCP `443` for HTTPS

##### Ansible related ports

- WinRM via HTTPS: TCP `5986` (for HTTP also TCP `5985`)
- SSH: TCP `22`

##### Further useful ports/types

- Windows RDP Port: 3389 (TCP)
- ICMP: useful for debugging


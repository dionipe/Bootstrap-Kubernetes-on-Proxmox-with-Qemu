# On-Premise Kubernetes Cluster on Proxmox Ceph Cluster 

<p align="center">
  <img src="https://raw.githubusercontent.com/dionipe/Bootstrap-Kubernetes-on-Proxmox-with-Qemu/master/ss.png" width="800">
</p>
<p align="center">
  <img src="https://raw.githubusercontent.com/dionipe/Bootstrap-Kubernetes-on-Proxmox-with-Qemu/master/ss2.png" width="800">
</p>

## Summary
Declaratively build a 4 node Kubernetes cluster on Proxmox using Ansible and QEMU. Optionally enable advanced features including ingress, load balancing, unattended upgrades, the Dashboard, etc.

**Approximate deployment time:** 25 minutes


## Requirements
1. Proxmox server
2. DNS Server*
3. Ansible 2.7.0+. Known incompatibility with a previous build.  
<sub>*A DNS server is not technically required, it is possible to manually add entries corresponding to your node hostnames to your Proxmox's hosts file. </sub>


## Instructions
**Required:**

1. Modify the `vars.yml` file with values specific to your environment.
2. Provision DNS A records for the IP Addresses & Hostnames you defined for your nodes in the `vars.yml` file.
3. Modify the `inventory.ini` file to reflect your chosen DNS records and the location of the SSH keys used to connect to the nodes.
4. Run the deployment: `ansible-playbook -i inventory.ini site.yml`
5. After deployment, a `~/.kube` directory will be created on your workstation. You can use the `config` file within to interact with your cluster.

**Optional:**

*To enable an optional feature, fill in the additional parameters in `vars.yml` and execute a playbook listed below.*

| Feature | Command | Requirements |
| ------- | ------- | ------------ |
| [Kubernetes Dashboard](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/) | `ansible-playbook -i inventory.ini playbooks/optional/deploy_dashboard.yml` | |
| [NFS backed persistent storage](https://github.com/kubernetes-incubator/external-storage/tree/master/nfs-client) | `ansible-playbook -i inventory.ini playbooks/optional/deploy_nfs_provisioner.yml` | |
| [MetalLB Load Balancer](https://metallb.universe.tf) | `ansible-playbook -i inventory.ini playbooks/optional/deploy_metallb.yml` | | 
| [NGINX Ingress Controller](https://github.com/kubernetes/ingress-nginx) | `ansible-playbook -i inventory.ini playbooks/optional/deploy_ingress-nginx.yml` | [MetalLB](https://metallb.universe.tf/) or other Load Balancer integration |
| [DataDog agents](https://docs.datadoghq.com/integrations/kubernetes/) | `ansible-playbook -i inventory.ini playbooks/optional/deploy_datadog.yml` | |
| [Unattended Upgrades](https://wiki.debian.org/UnattendedUpgrades) | `ansible-playbook -i inventory.ini playbooks/optional/enable_unattended_upgrades.yml` | |


## Tips
1. You can rollback the entire deployment with: `ansible-playbook -i inventory.ini playbooks/optional/delete_all_resources.yml`
2. If Calico isn't deploying correctly it's likely the CIDR you assigned to it in `vars.yml` conflicts with your network. 
3. [There appears to be an issue](https://forum.proxmox.com/threads/has-cloud-init-been-changed-between-5-3-and-6-0.56175/) with Proxmox's `cloud-init` implementation. Perhaps just with Debian? As a result, your VM might not have the correct information in `/etc/resolv.conf` and may also have multiple IP Addresses assigned to the `eth0` network interfaces. Furthermore, if you do not have a DHCP server active in the network that you are provisoning the VMs to, it is entirely possible that nothing will be present at all in `/etc/resolv.conf`.  
4. See [this repository](https://github.com/zimmertr/Bootstrap-Kubernetes-with-LXC) to do this with LXC instead.  Benefits of using LXC include:
```
* No virtualization overhead means better performance
* Ability to directly mount volumes from your server into your containers.
```


## TODO
1. Add better support for multi-node Proxmox clusters.
2. Perform security audit and enhance if necessary.
3. Add info to README about updating inventory file and how to handle SSH key generation and propegation.
4. Create playbook to upgrade kubernetes version for kubeadm cluster.


## Problems
1. The `proxmox_kvm` module is out of date and does not support cloudinit related api calls. Meaning shell commands must be used instead to perform `qm create` tasks. 
2. The `k8s` module does not support applying Kubernetes Deployments from URL. Instead of using `get_url` to download them first, and then apply them with `k8s`, I just use `shell` to run a `kubectl apply -f`. [Feature Request here](https://github.com/ansible/ansible/issues/48402).
3. Miscellaneous `qcow2` image issues:

| OS | Issue |
| -- | ----- |
| CentOS | A nameserver is baked into `/etc/resolv.conf` by default. [Bug Report here](https://bugs.centos.org/view.php?id=15426) |
| CoreOS | Proxmix issued cloud-init does not seem to configure networking properly. |

## Credit 
[Zimmertr Repository](https://github.com/zimmertr/Bootstrap-Kubernetes-with-QEMU)
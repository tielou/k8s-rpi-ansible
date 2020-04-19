# k8s-rpi-ansible
This Ansible playbook deploys a fully featured Kubernetes cluster built on a set of Raspberry Pi's.
It includes the following features:
* Kubernetes v1.18.2
* Flannel-based networking v0.12
* (multiple) NFS shares across the Raspberry Pi's and the necessary client provisioner to create PVC's
* Traefik v2.2 deployment as ingress load balancer for services with automatic Let's Encrypt capabilities
* Deployment mechanism for manifests as defined in my other repository: [tielou/kubernetes-manifests-rpi](https://github.com/tielou/kubernetes-manifests-rpi)
* Full idempotency of the playbook

## Prerequisites
To successfully deploy Kubernetes with this playbook you need to set up some prerequisites before the first run:
* Have a set of at least two Raspberry Pi's running with direct network connectivity
* Fresh install of Raspbian Buster Lite on the RPI's (this was developed with version [raspbian_lite-2020-02-14](http://downloads.raspberrypi.org/raspbian_lite/images/raspbian_lite-2020-02-14/2020-02-13-raspbian-buster-lite.zip) )
* SSH connectivity to the RPI's with public key authentication from the deployment computer
* Ansible 2.9.6 installed on the deployment computer
* Static IP's for the RPI's (and preferrably hostnames). See [this](https://thepihut.com/blogs/raspberry-pi-tutorials/how-to-give-your-raspberry-pi-a-static-ip-address-update)

## Usage
Clone this git repository by running:
```
git clone https://github.com/tielou/k8s-rpi-ansible.git
```
Open the `vars/main.yml` in your favorite text editor and adjust the values as desired. Find a detailed declaration further down.
Afterwards open the `ìnventory` file and adjust the IP's accordingly to the static ones you set for your RPI's. If you want to connect to the RPI's through another user than `pi` change the `ansible_ssh_user` variable.
If you would like to verify the connectivity and the Ansible configuration you can run:
`ansible all -i inventory -m ping`
If this succeeds you can trigger the execution of the playbook by running:
`ansible-playbook -i inventory main.yml`

After the run succeeded you can verify the funtionality by running:
`kubectl get nodes` and `kubectl get pods -A`
The outputs should be similar to:
```
NAME            STATUS   ROLES    AGE   VERSION
tipi-master-1   Ready    master   1m   v1.18.2
tipi-node-1     Ready    <none>   1m   v1.18.2
tipi-node-2     Ready    <none>   1m   v1.18.2
tipi-node-3     Ready    <none>   1m   v1.18.2
```
and respectively:
```
NAMESPACE     NAME                                     READY   STATUS    RESTARTS   AGE
default       nfs-client-provisioner-598449f86-zl8pq   2/2     Running   0          129m
kube-system   coredns-66bff467f8-k8rjd                 1/1     Running   0          26h
kube-system   coredns-66bff467f8-sdhpt                 1/1     Running   0          26h
kube-system   etcd-tipi-master-1                       1/1     Running   0          26h
kube-system   kube-apiserver-tipi-master-1             1/1     Running   0          26h
kube-system   kube-controller-manager-tipi-master-1    1/1     Running   0          26h
kube-system   kube-flannel-ds-arm-7r49k                1/1     Running   0          26h
kube-system   kube-flannel-ds-arm-8k2hb                1/1     Running   0          26h
kube-system   kube-flannel-ds-arm-bmbdx                1/1     Running   0          26h
kube-system   kube-flannel-ds-arm-zxlwl                1/1     Running   0          26h
kube-system   kube-proxy-42zrm                         1/1     Running   0          26h
kube-system   kube-proxy-4dhgz                         1/1     Running   0          26h
kube-system   kube-proxy-7tgc4                         1/1     Running   0          26h
kube-system   kube-proxy-d5hpg                         1/1     Running   0          26h
kube-system   kube-scheduler-tipi-master-1             1/1     Running   0          26h
kube-system   traefik-ingress-lb-b7k29                 1/1     Running   0          172m
kube-system   traefik-ingress-lb-h9k4k                 1/1     Running   0          172m
kube-system   traefik-ingress-lb-l57hf                 1/1     Running   0          172m
```
### Traefik Dashboard
You can find the Traefik dashboard at `http://x.x.x.x:8080/dashboard/` Replace `x.x.x.x` with any of your node IP's

## Variables

* `deploy_user` Define the user as which the whole playbook should be installed. For Raspbian the default user is `pi`.
* `kubernetes_version` Define the desired K8S version here, please be aware this is the Debian specific version number. This was tested with `1.18.2-00`.
* `nfs_shares` Define your desired NFS shares here. The `share` variable reflects the path of the mount on the nfs server whereas `mount` specifies the mount path on the client side.
* `nfs_master_ip` Define the static ip of your NFS server. This is used for the mounts on the client RPI's as well as for the nfs client provisioner deployment.
* `pod_network_cidr` Define your desired cidr range for the pods inside Kubernetes here. If you change the default, make sure to also adjust the variable `Network` in `flannel-networking.yml`, line 129.
* `kubeconfig_path` Define where you would like to store your Kubernetes configuration locally.
* `k8s_user_manifests` If you would like to deploy additional Kubernetes manifests declare them here and put them in `manifests/user`. You can set the namespace here, which will be created if not present. This playbook includes a sample deployment for Nextcloud.
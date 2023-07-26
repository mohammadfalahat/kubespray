# Lunch a kubernetes cluster with KubeSpray in 6 step:




## 1. Pull kubespray in Bastion

check kubespray github here: https://github.com/kubernetes-sigs/kubespray

set latest version tag in variable KUBESPRAY_VERSION.

```
KUBESPRAY_VERSION=v2.22.1
cd /opt
git clone https://github.com/kubernetes-sigs/kubespray.git
cd kubespray
git checkout $KUBESPRAY_VERSION
```



## 2. Copy Bastion SSH Public to nodes

generate a public key in bastion and copy it to all nodes:

```
ssh-keygen -o
cat ~/.ssh/id_rsa.pub
```
copy public key with SSH-COPY-ID
```
ssh-copy-id users@nodes
```



## 3. Installing Ansible in Bastion

```
apt install -y python3-pip 
cd /opt/kubespray
pip install -r requirements-2.12.txt
ansible --version
```



## 4. Configure kubespray in Bastion

Take a copy of inventory/sample and rename it to inventory/yourClusterName.

config inventory/yourClusterName/group_vars/all/all.ini

        # External LB
        # Internal loadbalancers
        # upstream dns
        # http_proxy and https_proxy and no_proxy
        # download container

config inventory/yourClusterName/group_vars/k8s_cluster/k8s_cluster.ini

        # kube_version
        # kube_proxy_mode
        # optionally for kube deamons
        # optionally for OS system deamons
        # LB SAN ip's, vip and hostname in: supplementry address in ssl keys

delete everything in group_vars/all just keep:

        # all.yml
        # containerd.yml
        # coreos.yml
        # etcd.yml

delete everything in group_vars/k8s_cluster just keep:

        # k8s-cluster.yml
        # k8s-net-calico.yml


## 5. Configure LoadBalancers

Load Balancer is out of kubespray. we config it manually.

install on both loadbalancers:

```
apt install -y haproxy keepalived
```

config /etc/keepalived/keepalived.cfg. sample config is here: https://github.com/falahatme/keepalived

config /etc/haproxy/haproxy.cfg. sample config is here: https://github.com/falahatme/haproxy

validate HA-PROXY config before restarting it:

```
 haproxy -c -f /etc/haproxy/haproxy.cfg
```
```
 service haproxy restart
```


## 6. Ansible Check And Start in Bastion

run ansible ping in bastion:
```
ansible all -i inventory/shoniz/inventory.ini -m ping
```

Let's go to install cluster with kubespray ansible:
```
ansible-playbook -i inventory/shoniz/inventory.ini cluster.yml
```



## Changes And Updates

change your configs in inventory.ini  and yml files
for example you added worker2:

```
ansible-playbook -i inventory/shoniz/inventory.ini playbooks/facts.yml
ansible-playbook -i inventory/shoniz/inventory.ini playbooks/scale.yml --limit=worker2
```



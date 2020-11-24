### Install
```sh
kubectl apply -f https://docs.projectcalico.org/v3.6/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml
oc set env ds calico-node IP_AUTODETECTION_METHOD=interface=eth0
oc set env ds calico-node FELIX_HEALTHHOST=127.0.0.1
oc adm policy add-scc-to-user privileged -z calico-node
oc adm policy add-scc-to-user privileged -z calico-upgrade-job
oc adm policy add-scc-to-user privileged -z calico-kube-controllers
ansible all -a "sysctl -w net.ipv4.conf.all.rp_filter=0"
# make sure you have `127.0.0.1  localhost`  entry in your /etc/hosts

```

### Debugging
```sh
docker exec -it <caliconode> sh
cat /etc/os-release
apk add curl
curl -O -L  https://github.com/projectcalico/calicoctl/releases/download/v3.5.1/calicoctl
./calicoctl get nodes
```

### force calico to use different network interface

```sh
oc set env ds calico-node --list | grep IP_AUTODETECTION_METHOD
```

### calicoctl
```
export DATASTORE_TYPE=kubernetes
export KUBECONFIG=~/.kube/config
calicoctl get workloadendpoints

ansible all -a "curl -O -L  https://github.com/projectcalico/calicoctl/releases/download/v3.6.1/calicoctl"
ansible all -a "chmod +x calicoctl"
ansible all -a "cp calicoctl /usr/local/bin/calicoctl"
ansible all -m copy -a "src=~/.kube/config dest=/tmp/.kubeconfig"
ansible all -a "DATASTORE_TYPE=kubernetes KUBECONFIG=/tmp/.kubeconfig calicoctl node status"
```

```sh
oc delete po -l k8s-app=calico-node
```
### While Calico setup has been done in a k8s cluster having two interface (  nic )   
There are chances in your prod environment that you get two nic interface , problem begins from here because vxlan encapsulation defult setting may misbehave with your connectivity ( internal within k8s)   
fix is to use bellow   
``
DATASTORE_TYPE=kubernetes KUBECONFIG=~/.kube/config calicoctl patch ippool default-ipv4-ippool -p '{"spec": {"vxlanMode": "Always"}}'
``   

One more thing is if you wish to direct calico to use which particuler interface . do this.   

``
apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
  name: default
spec:
  calicoNetwork:
    ipPools:
    - blockSize: 26
      cidr: 192.168.0.0/16
      encapsulation: VXLANCrossSubnet
      natOutgoing: Enabled
      nodeSelector: all()
    nodeAddressAutodetectionV4
	  firstFound: false
	  interface: eth1
 ``

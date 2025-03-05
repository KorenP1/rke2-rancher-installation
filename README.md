# rke2-rancher-installation

RKE2 creates the following components (Can be edited with flags in installation script):
* CRI: containerd
* CNI: Canal
* Ingress Controller: Nginx
* > DOES **NOT** COME WITH local-path-provisioner CSI

### Creating Machines
Creating servers, 1 master, at least 1 worker (Choose your prefered OS)

### RKE2 Installation
Use root user!  
If your OS is using NetworkManager, you must exclude the CNI from it.  
```
cat > /etc/NetworkManager/conf.d/rke2-canal.conf << EOF
[keyfile]
unmanaged-devices=interface-name:flannel*;interface-name:cali*;interface-name:tunl*;interface-name:vxlan.calico;interface-name:vxlan-v6.calico;interface-name:wireguard.cali;interface-name:wg-v6.cali
EOF
systemctl restart NetworkManager.service
```  
Edit the hostname of the servers to master-0, master-1, worker-0, worker-1...  `nmtui` OR `hostnamectl hostname <hostname>`  
Needed commands: `apt install curl vim htop`  
Specify version if needed, i have specified v1.30.4+rke2r1  
**Master:** `curl -sfL https://get.rke2.io | INSTALL_RKE2_VERSION=<Version> sh -`  
**Worker:** `curl -sfL https://get.rke2.io | INSTALL_RKE2_VERSION=<Version> INSTALL_RKE2_TYPE="agent" sh -`
  
If worker:  
Insert server ip and token  
```
mkdir -p /etc/rancher/rke2/
cat > /etc/rancher/rke2/config.yaml << EOF
server: https://<server>:9345
token: <token from server node in path /var/lib/rancher/rke2/server/node-token>
EOF
```

Enable and start the service:  
**Master:** `systemctl enable rke2-server.service && systemctl start rke2-server.service`  
**Worker:** `systemctl enable rke2-agent.service && systemctl start rke2-agent.service`

With my installation i wanted to have only one ingress controller pod so i can skip installing load balancer or external reverse proxy for the cluster, i am mentioning that i am giving up on high availability.  
Insert nodeName (e.g. master-0)
`kubectl patch daemonset  -n kube-system rke2-ingress-nginx-controller --patch '{"spec": {"template": {"spec": {"nodeName": "<NODE_NAME>"}}}}'`

### Adding default certificate for the nginx ingress controller
Insert certificate and key  
`kubectl create secret tls -n kube-system ingresscontroller-certificate --cert <CERTIFICATE_FILE> --key <PRIVATE_KEY_FILE>`

`kubectl patch daemonset rke2-ingress-nginx-controller -n kube-system --type=json -p='[{"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--default-ssl-certificate=$(POD_NAMESPACE)/ingresscontroller-certificate"}]'`

### Rancher Installation
Insert Version and Hostname  
`helm install rancher -n cattle-system --create-namespace https://releases.rancher.com/server-charts/stable/rancher-<VERSION>.tgz --set hostname=<INGRESS_HOSTNAME> --set ingress.tls.source=secret`

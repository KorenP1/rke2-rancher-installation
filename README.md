# rke2-rancher-installation

RKE2 creates the following components (Can be edited with flags in installation script):
* CRI: containerd
* CNI: Canal
* Ingress Controller: Nginx
* > DOES **NOT** COME WITH local-provisioner CSI

### Creating Machines
Creating servers, 1 master, at least 1 worker (Choose your prefered OS)

### Installation RKE2
Use root user!  
Edit the hostname of the servers to master-0, master-1, worker-0, worker-1...  `nmtui` OR `hostnamectl hostname <hostname>`  
Needed commands: `apt install curl vim`  
Specify version if needed, i have specified v1.30.4+rke2r1  
**Master:** `curl -sfL https://get.rke2.io | INSTALL_RKE2_VERSION=<Version> sh -`  
**Worker:** `curl -sfL https://get.rke2.io | INSTALL_RKE2_VERSION=<Version> INSTALL_RKE2_TYPE="agent" sh -`
  
If worker:  
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

With my installation i wanted to have only one ingress controller pod so i can skip installing load balancer or external reverse proxy for the cluster, i am mentioning that i am giving up on high availability. my node-name is master-0  
`kubectl patch daemonset  -n kube-system rke2-ingress-nginx-controller --patch '{"spec": {"template": {"spec": {"nodeName": "<node-name>"}}}}'`

### Adding default certificate for the nginx ingress controller
`kubectl create secret tls -n kube-system ingresscontroller-certificate --cert [CERTIFICATE_FILE] --key [PRIVATE_KEY_FILE]`  
`kubectl patch daemonset rke2-ingress-nginx-controller -n kube-system --type=json -p='[{"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--default-ssl-certificate=$(POD_NAMESPACE)/ingresscontroller-certificate"}]'`

### Installing Rancher

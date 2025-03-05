
### rke2-installation

RKE2 creates the following components (Can be edited with flags in installation script):
* CRI: containerd
* CNI: Canal
* CSI: local storage + longhorn (idk what is that)
* Ingress Controller: Traefik
* Helm Controller (Helm Release Object)
* More stuff

Creating servers, at least 1 master, at least 1 worker (Choose your prefered OS)  
Edit the hostname of the servers to master-0, master-1, worker-0, worker-1...
`nmtui` OR `hostnamectl hostname <hostname>`

Use root user!  
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

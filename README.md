# rke2-installation

RKE2 creates the following components (Can be edited with flags in installation script):
* CRI: containerd
* CNI: Canal
* CSI: local storage + longhorn (idk what is that)
* Ingress Controller: Traefik
* Helm Controller (Helm Release Object)
* More stuff
  
Creating servers, at least 1 master, at least one worker.

Edit the hostname of the servers to master-0, master-1, worker-0, worker-1...

Use root user

Master command: ```curl -sfL https://get.rke2.io | sh -```

Worker command: ```curl -sfL https://get.rke2.io | INSTALL_RKE2_TYPE="agent" sh -```

Enable the service: ```systemctl enable rke2-agent.service```

If worker:
```
mkdir -p /etc/rancher/rke2/
cat > /etc/rancher/rke2/config.yaml << EOF
server: https://<server>:9345
token: <token from server node>
EOF
```

Start the service ```systemctl start rke2-agent.service```

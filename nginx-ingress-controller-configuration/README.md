# Nginx Ingress Controller Configuration

There is a configMap that is configuring the nginx ingress controller  
In RKE2 its in the namespace kube-system and called rke2-ingress-nginx-controller

For disabling auto redirect http to https you need to add ssl-redirect=false  
`kubectl patch configmap rke2-ingress-nginx-controller -n kube-system --type merge -p '{"data": {"ssl-redirect": "false"}}'`  

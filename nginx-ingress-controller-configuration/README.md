# Nginx Ingress Controller Configuration

Nginx ingress controller is exposing both http and https for every ingress without asking you. 

There is a configMap that is configuring the nginx ingress controller  
In RKE2 its in the namespace kube-system and called rke2-ingress-nginx-controller

Redirect every http to https.  
`kubectl patch configmap rke2-ingress-nginx-controller -n kube-system --type merge -p '{"data": {"force-ssl-redirect": "true"}}'`  

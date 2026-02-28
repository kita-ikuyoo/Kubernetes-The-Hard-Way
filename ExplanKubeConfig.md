The bash code for creating kubeconfig files is as follows. Let me explain them to you.
set-cluster: this sets the server(API server) the kubelet needs to talk to. 
set-credentials: this sets which user the kubelet uses to talk to the server.
set-context: this sets the pair of server and user
use-context: this sets which context to be used. This command sets current-context entry in the kubeconfig file.
```bash
for host in node-0 node-1; do
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://server.kubernetes.local:6443 \
    --kubeconfig=${host}.kubeconfig

  kubectl config set-credentials system:node:${host} \
    --client-certificate=${host}.crt \
    --client-key=${host}.key \
    --embed-certs=true \
    --kubeconfig=${host}.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:node:${host} \
    --kubeconfig=${host}.kubeconfig

  kubectl config use-context default \
    --kubeconfig=${host}.kubeconfig
done
```
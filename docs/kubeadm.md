# Deploying kube-router with kubeadm

Please follow the [steps](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/) to install Kubernetes cluster with Kubeadm, however must specify `--pod-network-cidr` when you run `kubeadm init`.

Kube-router relies on kube-controller-manager to allocate pod CIDR for the nodes.

Kube-router provides pod networking, network policy and high perfoming IPVS/LVS based service proxy. Depending on you choose to use kube-router for service proxy you have two options.

## kube-router providing pod networking and network policy

For the step #3 **Installing a pod network** install a kube-router pod network and network policy add-on with the following command (Kubernetes version should be at least 1.8):

```sh
KUBECONFIG=/etc/kubernetes/admin.conf kubectl apply -f https://raw.githubusercontent.com/cloudnativelabs/kube-router/master/daemonset/kubeadm-kuberouter.yaml
```

## kube-router providing service proxy, firewall and pod networking.

For the step #3 **Installing a pod network** install a kube-router pod network and network policy add-on with the following command (Kubernetes version should be at least 1.8):

```sh
KUBECONFIG=/etc/kubernetes/admin.conf kubectl apply -f https://raw.githubusercontent.com/cloudnativelabs/kube-router/master/daemonset/kubeadm-kuberouter-all-features.yaml
```

Now since kube-router provides service proxy as well. Run below commands to remove kube-proxy and cleanup any iptables configuration it may have done.

```sh
KUBECONFIG=/etc/kubernetes/admin.conf kubectl -n kube-system delete ds kube-proxy
```

To cleanup kube-proxy we can do this with docker or containerd:

docker:
```sh
docker run --privileged -v /lib/modules:/lib/modules --net=host k8s.gcr.io/kube-proxy-amd64:v1.23.4 kube-proxy --cleanup
```

containerd:
```sh
ctr images pull k8s.gcr.io/kube-proxy-amd64:v1.23.4
ctr run --rm --privileged --net-host --mount type=bind,src=/lib/modules,dst=/lib/modules,options=rbind:ro \
    k8s.gcr.io/kube-proxy-amd64:v1.23.4 kube-proxy-cleanup kube-proxy --cleanup
```
[[kubernetes]]
-   [Debian-based distributions](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#k8s-install-0)



1.  Update the `apt` package index and install packages needed to use the Kubernetes `apt` repository:

    ```bash
    sudo apt-get update
    sudo apt-get install -y apt-transport-https ca-certificates curl

    ```

2.  Download the Google Cloud public signing key:

    ```bash
    sudo curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

    ```

3.  Add the Kubernetes `apt` repository:

    ```bash
    echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

    ```

4.  Update `apt` package index, install kubelet, kubeadm and kubectl, and pin their version:

    ```bash
    sudo apt-get update
    sudo apt-get install -y kubelet kubeadm kubectl
    sudo apt-mark hold kubelet kubeadm kubectl

    ```
- Turn swap off:
 ```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

```
- Copy conf file:
```bash 
mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

```
- Copy conf file from host to workstation:
```bash
sudo scp so2@192.168.2.10:/home/so2/.kube/config $HOME/.kube/config
```
Alternatif
```bash
export KUBECONFIG=/etc/kubernetes/admin.conf

```

Sample init command
```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --cri-socket=unix:///run/containerd/containerd.sock
```

Create a containerd configuration file 
```bash

sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml

```
### Set the cgroup driver for runc to systemd

Set the cgroup driver for runc to systemd which is required for the kubelet.\
For more information on this config file see the containerd configuration docs [here](https://github.com/containerd/cri/blob/master/docs/config.md) and [also here](https://github.com/containerd/containerd/blob/master/docs/ops.md).

At the end of this section in `/etc/containerd/config.toml`

```
      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
      ...

```

Around line 112, change the value for `SystemCgroup` from `false` to `true`.

```
            SystemdCgroup = true

```

If you like, you can use sed to swap it out in the file with out having to manually edit the file.

```bash
sudo sed -i 's/            SystemdCgroup = false/            SystemdCgroup = true/' /etc/containerd/config.toml

```

### Restart containerd with the new configuration

```bash
sudo systemctl restart containerd

```
let pods to be opened in control-plane node

```bash
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```

### Add new certificate
[link for this issue](https://blog.scottlowe.org/2019/07/30/adding-a-name-to-kubernetes-api-server-certificate/)
get the config yml
```
kubectl -n kube-system get configmap kubeadm-config -o jsonpath='{.data.ClusterConfiguration}' > kubeadm.yaml
```

```yaml
apiServer:
  certSANs:
  - "172.29.50.162"
  - "k8s.domain.com"
  - "other-k8s.domain.net"
  extraArgs:
    authorization-mode: Node,RBAC
  timeoutForControlPlane: 4m0
```

First, move the existing API server certificate and key (if `kubeadm` sees that they already exist in the designated location, it won’t create new ones):

```
mv /etc/kubernetes/pki/apiserver.{crt,key} ~
```

Then, use `kubeadm` to _just_ generate a new certificate:

```
kubeadm init phase certs apiserver --config kubeadm.yaml
```

This command will generate a new certificate and key for the API server, using the specified configuration file for guidance. Since the specified configuration file includes a `certSANs` list, then `kubeadm` will automatically add those SANs when creating the new certificate.

The final step is restarting the API server to pick up the new certificate. The easiest way to do this is to kill the API server container using `docker`:

1.  Run `docker ps | grep kube-apiserver | grep -v pause` to get the container ID for the container running the Kubernetes API server. (The container ID will be the very first field in the output.)
2.  Run `docker kill <containerID>` to kill the container.

If your nodes are running containerd as the container runtime, the commands are a bit different:

1.  Run `crictl pods | grep kube-apiserver | cut -d' ' -f1` to get the Pod ID for the Kubernetes API server Pod.
2.  Run `crictl stopp <pod-id>` to stop the Pod.
3.  Run `crictl rmp <pod-id>` to remove the Pod.

The Kubelet will automatically restart the container, which will pick up the new certificate. As soon as the API server restarts, you will immediately be able to connect to it using one of the newly-added IP addresses or hostnames.

[official docs](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

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


[official docs](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

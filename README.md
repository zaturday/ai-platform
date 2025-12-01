# ai-platform
internal project for ai platform container

# project detail
- Ubuntu 24.04 LTS
- K3s 1.33.6 (container platform)
- Rancher
- Longhorn
- GPU Nvidia driver v.X

# Ubuntu
install tools
```
apt install vim
apt install net-tools
```

tunning os
```
echo -e "net.ipv6.conf.all.disable_ipv6 = 1\nnet.ipv6.conf.default.disable_ipv6 = 1\nnet.ipv6.conf.lo.disable_ipv6 = 1" | sudo tee -a /etc/sysctl.conf
echo "fs.file-max = 1048576" | sudo tee -a /etc/sysctl.conf
echo "fs.inotify.max_user_watches=1048576" | sudo tee -a /etc/sysctl.conf
echo "fs.inotify.max_user_instances=512" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```
turnoff swap
```
sudo swapoff -a
sudo sed -i '/swap/d' /etc/fstab
```


# Helm
install helm
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-4
chmod 700 get_helm.sh
./get_helm.sh

```

# K3s
install K3s
```

```



setup KUBCONFIG
```
echo "export KUBECONFIG=/etc/rancher/k3s/k3s.yaml" >> $HOME/.bash_profile

```
# Longhorn CSI
install Longhorn with Helm
```
helm repo add longhorn https://charts.longhorn.io
helm repo update
helm install longhorn longhorn/longhorn   --namespace longhorn-system   --create-namespace   --set defaultSettings.defaultDataPath="/longhorndata"
cat <<EOF | sudo kubectl apply -f -
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: longhorn-single-node
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: driver.longhorn.io
allowVolumeExpansion: true
parameters:
  numberOfReplicas: "1"
  staleReplicaTimeout: "2880"
  fromBackup: ""
  fsType: "ext4"
EOF

```

# Rancher
install rancher with Helm
```

```


``
reference install with helm
https://artifacthub.io/packages/helm/rancher-latest/rancher
``

# NVIDIA Driver
```
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
apt-get update
apt-get install -y nvidia-container-toolkit
nvidia-container-cli --version
nvidia-smi
sudo apt-get install nvidia-driver firmware-misc-nonfree nvidia-smi nvidia-container-runtime
sudo apt-get install -y gpg curl
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg
curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
sudo apt-get update
sudo apt-get install nvidia-driver firmware-misc-nonfree nvidia-smi nvidia-container-runtime
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg   && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list |     sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' |     sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
nvidia-smi
ubuntu-drivers devices
sudo ubuntu-drivers autoinstall
reboot
```

#  NVIDIA GPU Operator
```
https://github.com/NVIDIA/gpu-operator/issues/522#issuecomment-1545574686

helm install --wait nvidiagpu
-n gpu-operator --create-namespace
--set toolkit.env[0].name=CONTAINERD_CONFIG
--set toolkit.env[0].value=/var/lib/rancher/k3s/agent/etc/containerd/config.toml
--set toolkit.env[1].name=CONTAINERD_SOCKET
--set toolkit.env[1].value=/run/k3s/containerd/containerd.sock
--set toolkit.env[2].name=CONTAINERD_RUNTIME_CLASS
--set toolkit.env[2].value=nvidia
--set toolkit.env[3].name=CONTAINERD_SET_AS_DEFAULT
--set-string toolkit.env[3].value=true
nvidia/gpu-operator

```

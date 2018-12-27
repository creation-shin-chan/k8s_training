# Installation
## 1. docker-ceのインストール
```
apt-get update
apt-get install -y \
     apt-transport-https \
     ca-certificates \
     curl \
     gnupg2 \
     software-properties-common
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"
apt-get update
apt-get install -y docker-ce=17.03.3~ce-0~debian-stretch
```
To get all available versions
> `apt-cache madison docker-ce`

dockerのインストール・起動確認
> `docker info`

## 2. kubeadm, kubelet, kubectlのインストール
```
apt-get update && apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl
```

## 3. kubernetesマスターノード/ワーカーノードのインストール
`kubeadm`コマンドを使用してマスターノードを配備します。下記のコマンドはマスターノードのみで実行してください。
> `kubeadm init`

問題なく配備されると

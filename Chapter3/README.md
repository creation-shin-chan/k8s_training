# Installation
## キーノート
- kubelet
- kubeadm
- kubectl
- マスターノード
- ワーカーノード
- taint(Chapter11で解説)

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
インストール可能な全てのバージョンを確認するには下記のコマンドを実行します。
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
`kubeadm`コマンドを使用してマスターノードをインストールします。下記のコマンドはマスターノードで実行してください。
> `kubeadm init`

問題なく配備されると最後に下記のような出力が出てくるはずです。
```
Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run (as a regular user):

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the addon options listed at:
  http://kubernetes.io/docs/admin/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:<hash>
```

上記の指示に従って`kubectl`のコンフィグを配置します。
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

次にワーカーノードをインストールします。`kubeadm init`の最後の行に出力されるコマンドをワーカーノードで実行します。

> 例：  
> `kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:<hash>`

マスターへ戻り下記のコマンドを実行し、設定が完了したか確認してください。成功しているとノードが2つ表示されるはずです。
> `kubectl get nodes`

マスターノードにtaint(Chapter11で解説)が設定されていますので削除します。
まずは確認。
> `kubectl describe node | grep Taint`  

taintを削除。
> `kubectl taint nodes --all node-role.kubernetes.io/master-`  

削除された事を確認
> `kubectl describe node | grep Taint`

ワーカーノードで`not found`と出ますがワーカーノードには`taint`が設定されていない為、`not found`と出力されますが無視してください。

kubectlの補完機能を使用するため下記のコマンドを実行しておきましょう。
> `source <(kubectl completion bash) && echo 'source <(kubectl completion bash)' >> ~/.bashrc`
 
コマンドを途中まで実行し、`tab`を押すと残りが補完されるはずです。


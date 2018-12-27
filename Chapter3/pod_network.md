# Pod Network
### Pod Networkのインストール:
`kubectl get nodes`を実行するとstatusが`Not Ready`になっていると思います。これはまだ、`pod network`がインストールされていないからです。Kubernetesのクラスターには必ず1つ`pod network`をインストールする必要があります。

今回はWeaveNetを使用します。マスター上で下記のコマンドを実行し、`pod network`をインストールしましょう。
> `kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"`

インストールが完了したらもう一度`kubectl get nodes`を実行してください。  
statusが`Ready`になっているはずです。ならない場合は暫くしてもう一度実行してください。
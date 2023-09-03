# 開発環境

## kubectl のインストール

```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
rm -rf kubectl*
```

## stern のインストール

- https://github.com/stern/stern

```
go install github.com/stern/stern@latest
```

## kind のインストール

```
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

## kind の使い方

以下のコマンドで k8s クラスタを作成します。

```
sudo kind create cluster -n kind
```

kubeconfig を設定します。

```
sudo -E kind export -n kind kubeconfig
sudo -E chown -R $USER:$USER ~/.kube
```

クラスタ一覧の確認。

```
sudo kind get clusters
```

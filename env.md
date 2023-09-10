# 開発環境

- ツール類の紹介など

## kind

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

## ctlptl

ctlptl も kind と同様に k8s クラスタを作成管理することができます。  
kind をそのまま使ってもよいですが、ctlptl では k8s を yaml で宣言的に管理することができます。

まず、以下のような cluster.yaml を用意します。

```
apiVersion: ctlptl.dev/v1alpha1
kind: Registry
name: mdview-registry
port: 5000
---
apiVersion: ctlptl.dev/v1alpha1
kind: Cluster
name: kind-mdview-dev
product: kind
kubernetesVersion: v1.27.3
registry: mdview-registry
```

そして、以下のコマンドで k8s クラスタを作成、削除することができます。

```
# k8sクラスタを作成
$ ctlptl apply -f ./cluster.yaml

# k8sクラスタを削除
$ ctlptl delete -f ./cluster.yaml
```

## kustomize

Kubernetes のマニフェスト管理ツールです。

ベースのマニフェストに対して、別の yaml ファイルで上書きして、一つのマニフェストとして扱うことができます。

```
# 環境に適用せず、buildだけしてbaseとoverlaysの合成結果を確認する
$ kubectl kustomize [overlaysのdirpath]

# 環境に適用せず、環境に適用済の定義と、kusomizeで作成した定義の比較する
$ kubectl diff -k [overlaysのdirpath]

# kusomizeで作成したマニフェストを環境に適用する
$ kubectl apply -k [overlaysのdirpath]
```

## Tilt

- 参考
  - [公式](https://docs.tilt.dev/tutorial/index.html)
  - [Tilt でカスタムコントローラーの開発を効率化しよう](https://zenn.dev/zoetro/articles/fba4c77a7fa3fb)

Tilt は、コンテナベースの開発に特化したタスクランナーです。  
Tiltfile に、どのファイルを監視して、どのようにコンテナイメージをビルドし、kubectl apply や docker-compose up までのタスクを管理します。  
また、Tilefile 自体も編集したら自動で読み直してくれます。

Tiltfile は、Starlark という Python に似た言語で記述することができます。  
Starlark については、https://github.com/bazelbuild/starlark/blob/master/README.md を参照してください。

コンテナイメージをビルドせずにバイナリファイルだけ入れ替えて livereload するといったことも可能です。

tilt は UI を持っており、外部からアクセスしたい場合は、--host 0.0.0.0 を付けて起動します。

```
$ tilt up --host 0.0.0.0
```

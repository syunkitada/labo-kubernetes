# kubebuilder の開発環境

## プロジェクトの開発環境構築の流れ

```
# 各種ツールのインストール
$ aqua i

# k8s環境のセットアップ
$ make start

# tiltの起動
$ make tilt-up

# k8s環境のクリーンアップ
$ make stop
```

## tilt の設定について

kubebuilder 用の Tiltfile は以下で公開されているので、これをひな型にするとよいです。  
https://github.com/tilt-dev/tilt-extensions/tree/master/kubebuilder

```
$ wget https://raw.githubusercontent.com/tilt-dev/tilt-extensions/master/kubebuilder/Tiltfile
```

そのままだと動かなかったので以下のように変更しました。

```
$ diff Tiltfile.origin Tiltfile
12c12
<         data = local('cd config/manager; kustomize edit set image controller=' + IMG + '; cd ../..; kustomize build config/default')
---
>         data = local('cd config/manager; bin/kustomize edit set image controller=' + IMG + '; cd ../..; bin/kustomize build config/default')
29c29
<         return 'controller-gen ' + CONTROLLERGEN
---
>         return 'bin/controller-gen ' + CONTROLLERGEN
32c32
<         return 'controller-gen object:headerFile="hack/boilerplate.go.txt" paths="./...";'
---
>         return 'bin/controller-gen object:headerFile="hack/boilerplate.go.txt" paths="./...";'
39c39
<         return 'CGO_ENABLED=0 GOOS=linux GOARCH=amd64 GO111MODULE=on go build -o tilt_bin/manager main.go'
---
>         return 'CGO_ENABLED=0 GOOS=linux GOARCH=amd64 GO111MODULE=on go build -o tilt_bin/manager cmd/main.go'
57c57
<     local_resource('CRD', manifests() + 'kustomize build config/crd | kubectl apply -f -', deps=["api"])
---
>     local_resource('CRD', manifests() + 'bin/kustomize build config/crd | kubectl apply -f -', deps=["api"])
59c59,61
<     k8s_yaml(yaml())
---
>     # for livereload
>     watch_file('./config/')
>     k8s_yaml(kustomize('./config/dev'))
61c63
<     deps = ['controllers', 'main.go']
---
>     deps = ['internal/controllers', 'cmd/main.go']
66c68
<     local_resource('Sample YAML', 'kubectl apply -f ./config/samples', deps=["./config/samples"], resource_deps=[DIRNAME + "-controller-manager"])
---
>     local_resource('Sample YAML', 'kubectl apply -k ./config/samples', deps=["./config/samples"], resource_deps=[DIRNAME + "-controller-manager"])
75a78,79
>
> kubebuilder("zoetrope.github.io", "view", "v1", "MarkdownView")
```

go build のビルド先が tilt_bin となっているので、.gitignore にこれを追加しておきます。

```
$ vim .gitignore
...
tilt_bin/*
```

Makefile に以下を追加しておきます。

```
#! [env]
.PHONY: start
start: ## Start local Kubernetes cluster
	ctlptl apply -f ./cluster.yaml
	kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/latest/download/cert-manager.yaml
	kubectl -n cert-manager wait --for=condition=available --timeout=180s --all deployments

.PHONY: tilt-up
tilt-up: ## Start tilt
	tilt up --host 0.0.0.0

.PHONY: stop
stop: ## Stop local Kubernetes cluster
	ctlptl delete -f ./cluster.yaml
#! [env]
```

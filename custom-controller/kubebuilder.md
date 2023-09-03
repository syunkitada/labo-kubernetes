# kubebuilder

## 各種ツールのインストール方法

- [kubebuilder.io/quick-start](https://book.kubebuilder.io/quick-start)

```
$ curl -L -o kubebuilder "https://go.kubebuilder.io/dl/latest/$(go env GOOS)/$(go env GOARCH)"
$ chmod +x kubebuilder && sudo mv kubebuilder /usr/local/bin/
```

## プロジェクトの作成

以下のコマンドでプロジェクトのひな型を生成します。

```
$ mkdir markdown-view
$ cd markdown-view
$ kubebuilder init --domain zoetrope.github.io --repo github.com/zoetrope/markdown-view
```

- kubebuilder init のオプション説明
  - --domain で指定した名前は CRD のグループ名に使われます。あなたの所属する組織が保持するドメインなどを利用して、ユニークで valid な名前を指定してください。
  - --repo には go modules の module 名を指定します。GitHub にリポジトリを作る場合は github.com/<user_name>/<product_name>を指定します。

kubebuilder init を実行すると、以下のファイルが自動生成されます。

```bash
.
├── Dockerfile
├── Makefile
├── PROJECT
├── README.md
├── cmd
│   └── main.go
        # main.go の中に、 //+kubebuilder:scaffold:imports, //+kubebuilder:scaffold:scheme, //+kubebuilder:scaffold:builder などのコメントが記述されています。
        # kubebuilerはこれらのコメントを目印にソースコードの自動生成を行うので、決して削除しないように注意してください。
├── config
    # configディレクトリ配下には、カスタムコントローラをKubernetesにデプロイするためのマニフェストが生成されます。
    # 実装する機能によっては必要のないマニフェストも含まれるので、適切に取捨選択してください。
│   ├── default
        # マニフェストをまとめて利用するための設定が記述されています。
        # 利用するマニフェストに応じて、kustomization.yamlを編集してください。
│   │   ├── kustomization.yaml
│   │   ├── manager_auth_proxy_patch.yaml
            # kube-auth-proxyを利用するために必要なパッチです。
            # kube-auth-proxyを利用しない場合は削除しても問題ありません。
│   │   └── manager_config_patch.yaml
            # カスタムコントローラのオプションを引数ではなくConfigMapで設定するためのパッチファイルです。
│   ├── manager
        # カスタムコントローラのDeploymentリソースのマニフェストです。
        # カスタムコントローラのコマンドラインオプションの変更をおこなった場合など、必要に応じて書き換えてください。
│   │   ├── kustomization.yaml
│   │   └── manager.yaml
│   ├── prometheus
        # Prometheus Operator用のカスタムリソースのマニフェストです。
        # Prometheus Operatorを利用している場合、このマニフェストを適用するとPrometheusが自動的にカスタムコントローラのメトリクスを収集してくれるようになります。
│   │   ├── kustomization.yaml
│   │   └── monitor.yaml
│   └── rbac
        # 各種権限を設定するためのマニフェストです。
│       ├── auth_proxy_client_clusterrole.yaml
│       ├── auth_proxy_role.yaml
│       ├── auth_proxy_role_binding.yaml
│       ├── auth_proxy_service.yaml
            # auth_proxy_から始まる4つのファイルは、kube-auth-proxy用のマニフェストです。
            # kube-auth-proxyを利用するとメトリクスエンドポイントへのアクセスをRBACで制限できます。
│       ├── kustomization.yaml
│       ├── leader_election_role.yaml
│       ├── leader_election_role_binding.yaml
            # leader_election_から始まる2つのファイルは、リーダーエレクション機能を利用するために必要な権限です。
│       ├── role_binding.yaml
│       └── service_account.yaml
            # role_binding.yaml, service_account.yamlは、コントローラが各種リソースにアクセスするための権限を設定するマニフェストです。
├── go.mod
├── go.sum
└── hack
    └── boilerplate.go.txt
      # boilerplate.go.txt は、自動生成されるソースコードの先頭に挿入されるボイラープレートです。
```

## API のひな型作成

以下のコマンドで、カスタムリソースとカスタムコントローラのひな型を生成します。

```
$ kubebuilder create api --group view --version v1 --kind MarkdownView --resource --controller
$ make manifests
```

- kubebuilder create api のオプション説明
  - --kind: 作成するリソースの名前を指定します。
  - --group: リソースが属するごループ名を指定します。
  - --version: 適切なバージョンを指定します。今後仕様が変わる可能性があるなら v1alpha1, v1beta1 を指定し、安定版であれば v1 を指定します。

kubebuilder create api を実行すると、以下のファイルが自動生成されます。

```
.
├── api
│   └── v1
│       ├── groupversion_info.go
            # これは初期生成後に編集する必要はありません。
│       ├── markdownview_types.go
            # MarkdownViewリソースをGo言語のstructで表現したものです。
            # 今後、MarkdownViewリソースの定義を変更するばあにはこのファイルを編集していくことになります。
│       └── zz_generated.deepcopy.go
            # これはmarkdownview_types.goの内容から自動生成されるファイルなので編集する必要はありません。
├── bin
│   └── controller-gen
├── cmd
│   └── main.go
        # create api実行後に、controllerの初期化処理が追加されています。
├── config
│   ├── crd
        # crdディレクトリにはCRD(Custom Resource Definition)のマニフェストが追加されています。
        # これらのマニフェストはmarkdownview_types.goから自動生成されるものなので、基本的に手動で編集する必要はありません。
        # ただし、Conversion Webhookを利用したい場合は、cainjection_in_markdownView.yamlとwebhook_in_markdownViews.yamlのパッチを利用するようにkustomization.yamlを書き換えてください。
│   │   ├── bases
│   │   │   └── view.zoetrope.github.io_markdownviews.yaml
│   │   ├── kustomization.yaml
│   │   ├── kustomizeconfig.yaml
│   │   └── patches
│   │       ├── cainjection_in_markdownviews.yaml
│   │       └── webhook_in_markdownviews.yaml
│   ├── rbac
        # role.yaml には、MarkdownViewリソースを扱うための権限が追加されています。
        # markdownview_editor_role.yaml と markdownview_viewer_role.yaml は、MarkdownViewリソースの編集・読み取りの権限です。必要に応じて利用しましょう。
│   │   ├── markdownview_editor_role.yaml
│   │   ├── markdownview_viewer_role.yaml
│   │   ├── role.yaml
│   └── samples
        # カスタムリソースのサンプルマニフェストです。テストで利用したり、ユーザ向けに提供できるように記述しておきましょう。
│       ├── kustomization.yaml
│       └── view_v1_markdownview.yaml
└── internal
    └── controller
        ├── markdownview_controller.go
        └── suite_test.go
```

## Webhook の生成

Kubernetes には、Admission Webhook と呼ばれる拡張機能があります。  
これは特定のリソースを作成・更新する際に Webhook API を呼び出し、バリデーションやリソースの書き換えを行うための機能です。

以下のコマンドで、webhook のひな型を生成します。

```
$ kubebuilder create webhook --group view --version v1 --kind MarkdownView --programmatic-validation --defaulting
$ make manifests
```

- kubebuilder create webhook のオプション
  - --programmatic-validation: リソースのバリデーションを行うための Webhook
  - --defaulting: リソースのフィールドにデフォルト値を設定するための Webhook
  - --conversion: カスタムリソースのバージョンアップ時にリソースの変換を行うための Webhook

kubebuilder create webhook を実行すると、以下のファイルが自動生成されます。

```
.
├── api
│   └── v1
│       ├── markdownview_webhook.go
            # Webhook実装のひな型です。
            # このファイルにWebhookの実装を追加していくことになります。
│       ├── webhook_suite_test.go
├── cmd
│   └── main.go
        # create webhook実行後に、webhookの初期化をおこなうためのコードが追加されています。
├── config
│   ├── certmanager
        # Admission Webhook機能を利用するためには証明書が必要となります。
        # cert-managerを利用して証明書を発行するためのカスタムリソースが生成されています。
│   │   ├── certificate.yaml
│   │   ├── kustomization.yaml
│   │   └── kustomizeconfig.yaml
│   ├── default
│   │   ├── manager_webhook_patch.yaml
│   │   └── webhookcainjection_patch.yaml
│   └── webhook
        # webhook機能を利用するために必要なマニフェストファイルです。
│       ├── kustomization.yaml
│       ├── kustomizeconfig.yaml
│       ├── manifests.yaml
            # manifests.yamlファイルは make manifestsで自動生成されるため、基本的に手動で編集する必要はありません。
│       └── service.yaml
```

この時点ではまだ Webhook 機能が利用できるようにはなってません。  
config/default/kustomization.yaml ファイルを編集する必要があります。

生成直後の kustomization.yaml では、以下がコメントアウトされており、これらコメントを外します。

- "resources:" の "- ../webhook", "- ../certmanager"
- "patchesStrategicMerge:" の "- manager_webhook_patch.yaml", "webhookcainjection_patch.yaml"
- "replacements:"

## カスタムコントローラの動作確認

Webhook 用の証明書を発行するために cert-manager が必要となります。 下記のコマンドを実行して cert-manager のデプロイをおこないます。

```
$ kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/latest/download/cert-manager.yaml

$ kubectl get pod -n cert-manager
NAME                                       READY   STATUS    RESTARTS   AGE
cert-manager-cainjector-84cdd9dd4b-jp48q   1/1     Running   0          98s
cert-manager-d6cd78457-glptn               1/1     Running   0          98s
cert-manager-webhook-56ffdd7c44-6r445      1/1     Running   0          98s
```

config/manager/manager.yaml の imagePullPolicy を IfNotPresent に設定します。  
これはコンテナイメージのタグ名に latest を指定した場合、ImagePullPolicy が Always になり、ロードしたコンテナイメージが利用されない場合があるためです。

```
        image: controller:latest
>       imagePullPolicy: IfNotPresent
```

下記のコマンドでコンテナイメージをビルドし、kind 環境にロードし直します。  
開発時もコントローラーの実装が変わった場合は、以下のコマンドを実行します

```
$ sudo make docker-build
$ sudo kind load docker-image controller:latest
```

以下のコマンドで、CRD を Kubernetes クラスターに適用します。  
開発時も CRD に変更がある場合はこのコマンドを実行します。  
ただし、互換性のない変更をおこなった場合はこのコマンドに失敗するため、事前に make uninstall を実行してください。

```
$ sudo make install
```

CRD 以外のマニフェストファイルに変更がある場合は下記のコマンドを実行します。ただし、互換性のない変更をおこなった場合はこのコマンドに失敗するため、事前に make undeploy を実行してください。

```
$ sudo make deploy
```

次のコマンドでカスタムコントローラーを再起動します。

```
$ kubectl rollout restart -n markdown-view-system deployment markdown-view-controller-manager
```

## pod の動作確認

```
$ kubectl get pod -n markdown-view-system
NAME                                                READY   STATUS    RESTARTS   AGE
markdown-view-controller-manager-58fc98c866-fg8lr   2/2     Running   0          37s
```

kubectl apply でリソースを作った際に、以下のようなログが流れれば OK です。

```
$ kubectl logs -f -n markdown-view-system markdown-view-controller-manager-58fc98c866-fg8lr
...
2023-09-03T11:24:25Z    INFO    markdownview-resource   default {"name": "markdownview-sample"}
2023-09-03T11:24:25Z    INFO    markdownview-resource   validate create {"name": "markdownview-sample"}
```

```
$ kubectl apply -f config/samples/view_v1_markdownview.yaml
markdownview.view.zoetrope.github.io/markdownview-sample created
```

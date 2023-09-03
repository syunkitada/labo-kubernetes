# controller-tools

- Kubebuilder では、カスタムコントローラの開発を補助するためのツール軍として controller-tools を提供しています。
- controller-tools には以下のツールが含まれています。
  - controller-gen
  - type-scaffold
  - helpgen

### controller-gen

- controller-gen は、Go のソースコードをもとにしてマニフェストや Go のソースコードの生成を行うツールです。

```
$ bin/controller-gen -h
...
generators

+webhook[:headerFile=<string>][,year=<string>]                                                                                                                                           package  generates (partial) {Mutating,Validating}WebhookConfiguration objects.
+schemapatch[:generateEmbeddedObjectMeta=<bool>],manifests=<string>[,maxDescLen=<int>]                                                                                                   package  patches existing CRDs with new schemata.
+rbac[:headerFile=<string>],roleName=<string>[,year=<string>]                                                                                                                            package  generates ClusterRole objects.
+object[:headerFile=<string>][,year=<string>]                                                                                                                                            package  generates code containing DeepCopy, DeepCopyInto, and DeepCopyObject method implementations.
+crd[:allowDangerousTypes=<bool>][,crdVersions=<[]string>][,generateEmbeddedObjectMeta=<bool>][,headerFile=<string>][,ignoreUnexportedFields=<bool>][,maxDescLen=<int>][,year=<string>]  package  generates CustomResourceDefinition objects.
```

- kubebuilder が生成した Makefile には、make manifests と make generate というターゲットが用意されており、make manifests では webhook, rbac, crd の生成、make generate では object の生成が行われます
- controller-gen がマニフェストの生成を行う際には、Go の struct の構成と、ソースコード中に埋め込まれた // +kubebuilder: から始まるコメント（マーカと呼ばれる）を目印にします。
- 利用可能なマーカーは下記のコマンドで確認できます。(-ww や-www を指定するとより詳細な説明が確認できます）

```
$ controller-gen crd -w
$ controller-gen webhook -w
```

#### CRD マニフェストの生成

- markdownview_types.go のフィールド定義で、CRD を定義し、Validation などの設定を行うことができます。
- 定義が完了したら make manifests でマニフェストを生成します。

```
 27 type MarkdownViewSpec struct {
 28     // Markdowns contain the markdown files you want to display.
 29     // The key indicates the file name and must not overlap with the keys.
 30     // The value is the content in markdown format.
 31     //+kubebuilder:validation:Required
 32     //+kubebuilder:validation:MinProperties=1
 33     Markdowns map[string]string `json:"markdowns,omitempty"`
 34
 35     // Replicas is the number of viewers.
 36     // +kubebuilder:default=1
 37     // +optional
 38     Replicas int32 `json:"replicas,omitempty"`
 39
 40     // ViewerImage is the image name of the viewer.
 41     // +optional
 42     ViewerImage string `json:"viewerImage,omitempty"`
 43 }
```

#### RBAC マニフェストの生成

- markdownview_controller.go の //+kubebuilder:rbac:xxx でコントローラに必要な権限を定義します。
- 定義が完了したら make manifests でマニフェストを生成します。

#### Webhook マニフェストの生成

- markdownview_webhook.go で Webhook の設定を行います。
- 定義が完了したら make manifests でマニフェストを生成します。

## controller-runtime

- カスタムコントローラを開発するためには、Kubernetes が標準で提供している client-go, apimachinery, api などのパッケージを利用することになります。
- 代表的なコンポーネント
  - manager.Manager
    - 複数のコントローラをまとめて管理するためのコンポーネントです。
    - リーダエレクションやメトリクスサーバ機能、カスタムコントローラを実装するための様々な機能をもっています。
  - client.Client
    - Kubernetes の kube-apiserver とやりとりするためのクライアントです。
    - 監視対象のリソースをインメモリにキャッシュする機能などを持ち、カスタムリソースも型安全に扱うことが可能なクライアントとなっています。
  - reconcile.Reconciler
    - カスタムコントローラが実装すべきインターフェイス

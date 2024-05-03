# しゃなりしゃなり マニフェスト

しゃなりしゃなりのマニフェスト部分を構築/管理する
※ Kubernetes を ArgoCD で管理するための構成管理

ArgoCD は GKE 上で運用するとコストが高くなるため、ローカル PC 上で運用することを方針とする。

# 必要ツール

以下の開発ツールを準備する

- [Visual Studio Code](https://azure.microsoft.com/ja-jp/products/visual-studio-code)
- [Docker Desktop](https://www.docker.com/ja-jp/products/personal/)
  - 個人開発のため
  - Kubernetes のプラグインを有効にする
- kubectl
  - Docker Desktop がインストール済みの場合は自動でインストールされている
- argocd CLI
  - Mac の場合は`brew install argocd`でインストールする

## 実行手順

### ArgoCD の起動停止

ローカル PC 上に ArgoCD を起動する。
以下の手順で ArgoCD を起動する

```
kubectl create namespace argocd
kubectl create namespace shanari-shanari
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj-labs/argocd-image-updater/stable/manifests/install.yaml
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

Web Server にアクセスする際は、別のターミナルを立ち上げて、以下を実行しておく

```
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

アクセス後は、username: `admin`とパスワードは下記の実行結果より生成されたものを入力してログインする。

```
argocd admin initial-password -n argocd
```

※ (個人向け) メモより Artifact Registry への secret を登録しておくこと
以下のコマンドで argocd に application をデプロイする

```
kubectl apply -n argocd -f local/argocd/application.yaml
```

停止、削除する場合は以下を実行する。

```
kubectl delete -n argocd -f local/argocd/application.yaml
kubectl delete -n argocd -f https://raw.githubusercontent.com/argoproj-labs/argocd-image-updater/stable/manifests/install.yaml
kubectl delete -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl delete namespace argocd
```

### ローカル環境の Kubernetes でアプリを起動

以下のコマンドを実行してローカル PC 上の Kubernetes でアプリケーションを起動します。

```bash
$ kubectl apply -f local/fe -R
```

以下のコマンドで停止します。

```bash
$ kubectl delete -f local/fe -R
```

`http://localhost`でアクセスできます。  
新しい Image を作成した場合は、各 deployment.yaml のバージョンを変更します。  
ToDo: 自動でバージョンを変更するか、実行時に引数として渡せるかの対応を検討する

## コンテキスト(接続先の Kuberenetes)の切り替え

以下のコマンドでコンテキストの切り替えを実施する。

- 一覧表示

```bash
$ kubectl config get-contexts
```

- 切り替え

```bash
$ kubectl config use-context <Clustername>
```

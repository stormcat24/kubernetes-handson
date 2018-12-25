# kubernetes-handson

## 💻Localでいい感じのKubernetes開発環境を揃える

### 💻kubectl

```bash
$ kubectl cluster-info
```

### 💻Dashboardのデプロイ

```bash
# Dashboardをデプロイ
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml

$ kubectl -n kube-system get pod -l k8s-app=kubernetes-dashboard
```

### 💻Dashboardログイン用ユーザー作成

```bash
# ログイン用ユーザー作成
$ kubectl apply -f https://raw.githubusercontent.com/stormcat24/kubernetes-handson/master/01-setup/admin-user.yaml

# アクセストークンの表示
$ kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}’)
```

### 💻proxyを実行

```bash
$ kubectl proxy
```

http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/

## 💻Kubernetesの基本的なリソース・アプリケーションのデプロイ

### 💻Podを作成

```bash
# Podの作成
$ kubectl apply -f 02-resource/simple-echo.yaml
pod "simple-echo" created

# 作成したPodの確認
$ kubectl get pod
```

### 💻Podの操作

```bash
# echoコンテナのログ（標準出力）の確認
$ kubectl logs -f simple-echo -c echo

# nginxコンテナにSSHログインっぽいことをする（shの実行）
$ kubectl exec -it simple-echo sh -c nginx

# Podを削除
$ kubectl delete pod simple-echo
```

### 💻ReplicaSetの作成

```bash
# ReplicaSetの作成
$ kubectl apply -f 02-resource/simple-echo-rs.yaml

# 作成したReplicaSet, Podの確認
$ kubectl get rs,pod --selector app=echo
```

### 💻ReplicaSetの削除

```bash
# ReplicaSetの削除
$ kubectl delete -f 02-resource/simple-echo-rs.yaml

# 削除したリソースの確認（ReplicaSet、Podともに削除されている）
$ kubectl get rs,pod --selector app=echo
```

### 💻Deploymentの作成

```bash
# ReplicaSetの作成
$ kubectl apply -f 02-resource/simple-echo-deployment.yaml

# 作成したDeployment, ReplicaSet, Podの確認
$ kubectl get deploy,rs,pod --selector app=echo
```

### 💻Deploymentの削除

```bash
# Deploymentの作成
$ kubectl delete -f 02-resource/simple-echo-deployment.yaml

# 削除したDeployment, ReplicaSet, Podの確認
$ kubectl get deploy,rs,pod --selector app=echo
```

### 💻ディスカバリの実験

```bash
# 2種類のDeploymentを作成
$ kubectl apply -f 02-resource/simple-echo-deployment-spring.yaml
$ kubectl apply -f 02-resource/simple-echo-deployment-summer.yaml

# 作成したDeploymentの確認
$ kubectl get deploy --selector app=echo
```

### 💻Serviceの作成

```bash
# Serviceを作成
$ kubectl apply -f 02-resource/simple-service.yaml

# 作成したServiceの確認
$ kubectl get svc --selector app=echo
```

### 💻デバッグ用Podから確認

```bash
# デバッグ用コンテナ
$ kubectl run -i --rm --tty debug --image=gihyodocker/fundamental:0.1.0 --restart=Never -- bash -il

# 作成したServiceに対してHTTP Request
debug:/# curl http://simple-echo/

# Pod名はそれぞれの環境で異なります

$ kubectl logs -f simple-echo-spring-xxxxx -c echo

$ kubectl logs -f simple-echo-summer-xxxxx -c echo
```

### 💻Serviceの変更

```bash
# Serviceの変更
$ kubectl apply -f 02-resource/simple-echo-service.yaml

# 再びデバッグコンテナからHTTP Request
debug:/# curl http://simple-echo/

# 今度はsummerの方にリクエストが来る
$ kubectl logs -f simple-echo-summer-7549994875-6g5vv -c echo
```

### 💻ingress-nginx導入

```bash
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.21.0/deploy/mandatory.yaml

$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.21.0/deploy/provider/cloud-generic.yaml
```

### 💻Ingress作成とHTTP Request

```bash
# Ingressを作成
$ kubectl apply -f 02-resource/simple-echo-ingress.yaml

# 作成したIngressを確認
$ kubectl get ing -l app=echo

# クラスタ外（ローカル）から、公開しているIngressに対してリクエストを送る　
$ curl http://localhost -H 'Host: echo.gihyo.local’
```

## 💻Helmでアプリケーションをパッケージングする

### 💻Helmセットアップ

```bash
# Homebrewならば
$ brew install kubernetes-helm

# バイナリダウンロードならば
https://github.com/helm/helm/releases/tag/v2.12.1

# helm初期化（TillerというPodがデプロイされる）
$ helm init —wait

# helm確認
$ helm version
Client: &version.Version{SemVer:"v2.12.1", GitCommit:”02a47c7...”, GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.12.1", GitCommit:”02a47c7...”, GitTreeState:"clean"}
```

### 💻Chartの検索

```bash
# Chartのリポジトリ確認
$ helm repo list

# リポジトリから適当にChart検索
$ helm search redmine
```

### 💻redmineデプロイ

```bash
# デフォルト値を上書きしてデプロイ
$ helm install -f 03-helm/redmine-values.yaml --name redmine stable/redmine

# helmで作られたリソースを確認
$ kubectl get ing,svc,pod -l release=redmine
```

### 💻redmine撤去

```bash
# helmでdeployしたリリース一覧
$ helm ls

# hostsでIngressに設定したドメインを追加
$ helm del --purge redmine
```

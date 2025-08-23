| 設定              | 値          |
| ----------------- | ----------- |
| トンネル名        | k8s         |
| namespace         | cloudflared |
| 作成される pod 数 | 2           |

## CloudFlaredをインストールする

以下を参考に自分にあったものを探してください

- https://pkg.cloudflare.com/index.html
- https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/downloads/

## トンネルを作成します。

```sh
cloudflared tunnel create k8s
```

作成後に `*.json` で終わるパスが表示されます。それをメモしてください。

## クレデンシャルを cloudflared ネームスペース上の secret に追加する

認証情報を今回の場合 cloudflared ネームスペースの secret に追加します。

```sh
kubectl create secret generic tunnel-credentials --from-file=credentials.json=/Users/yourusername/.cloudflared/<tunnel ID>.json -n cloudflared
```

## deployment.yaml の ignress.hostname を編集する

ご自身が所有しているドメインに hostname を置き換えてください。サブドメイン等は自由です。

## DNS に登録する

DNS の次に来る k8s は作成したトンネル名です。今回の構成では `k8s` となります。次に 先ほど編集した `ingress.hostname` に指定したドメインを入力し DNS を登録します。

```
cloudflared tunnel route dns k8s _k8s.akarinext.org
```

## apply する

kubectl を用いて apply します。しばらく待ったのち `cloudflared` pod が作成されていれば完了です。

```
kubectl apply -f ./deployment.yaml

# READYになってるか確認
kubectl get deployment cloudflared -n cloudflared
```

## 別のサービスを ingress に追加する

### ingress の service にどのような値を記述するか

大事な知識なので記述しておきます。

> クラスター内(DNS サーバーそれ自体も含む)で定義された全ての Service は DNS 名を割り当てられます。デフォルトでは、クライアント Pod の DNS サーチリストは Pod 自身のネームスペースと、クラスターのデフォルトドメインを含みます。
> 下記の例でこの仕組みを説明します。

> Kubernetes の bar というネームスペース内で foo という名前の Service があると仮定します。bar ネームスペース内で稼働している Pod は、foo に対して DNS クエリを実行するだけでこの Service を探すことができます。bar とは別の quux ネームスペース内で稼働している Pod は、foo.bar に対して DNS クエリを実行するだけでこの Service を探すことができます。

https://kubernetes.io/ja/docs/concepts/services-networking/dns-pod-service/

今回の構成では、`kubernetes-dashboard` という namespace で `kubernetes-dashboard` サービスが 443 ポートで動作しているため、 `https://kubernetes-dashboard.kubernetes-dashboard` という形式になります。

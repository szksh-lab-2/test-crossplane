# test-crossplane

## kind で cluster 作成

```sh
kind create cluster -n crossplane
```

## crossplane のインストール

https://docs.crossplane.io/v2.2/get-started/install/

```sh
helm repo add crossplane-stable https://charts.crossplane.io/stable
helm repo update
helm install crossplane \
    --namespace crossplane-system \
    --create-namespace crossplane-stable/crossplane
```

## upbound CLI のインストール

`up` command

install script がある。

https://docs.upbound.io/

## GitHub Provider のインストール

https://github.com/crossplane-contrib/provider-upjet-github

```sh
up ctp provider install xpkg.upbound.io/crossplane-contrib/provider-upjet-github:v0.19.0
```

認証情報の作成
GitHub App を使う。

改行を含む private key を JSON で書くのは難しいので、 YAML で書いてから JSON に変換する。

gh.yaml

```yaml
owner: szksh-lab-2
app_auth:
  - id: 0000000
    installation_id: 000000000
    pem_file: |
      -----BEGIN RSA PRIVATE KEY-----
      ...
      -----END RSA PRIVATE KEY-----
```

```sh
yaml2json < gh.yaml | pbcopy
```

```yaml
---
apiVersion: v1
kind: Secret
metadata:
  name: provider-secret
  namespace: crossplane-system
type: Opaque
stringData:
  credentials: |
    {"app_auth":[{"id":"0000000","installation_id":"000000000","pem_file":"-----BEGIN RSA PRIVATE KEY-----\nxxx\n-----END RSA PRIVATE KEY-----\n"}],"owner":"szksh-lab-2"}

---
apiVersion: github.upbound.io/v1beta1
kind: ProviderConfig
metadata:
  name: default
spec:
  credentials:
    source: Secret
    secretRef:
      name: provider-secret
      namespace: crossplane-system
      key: credentials
```

## GitHub Issue Label を crossplane で管理してみる

example は https://github.com/crossplane-contrib/provider-upjet-github/tree/main/examples にある。

issuelabel: https://github.com/crossplane-contrib/provider-upjet-github/blob/main/examples/cluster/repo/v1alpha1/issuelabels.yaml

[label.yaml](label.yaml)

```sh
k apply -f label.yaml
```

deploy すると定義した label が作成され、それ以外が削除された。

https://github.com/szksh-lab-2/test-github-action/labels

本当は label 単体で管理したい。

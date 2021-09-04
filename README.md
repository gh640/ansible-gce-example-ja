# Ansible で GCP Compute Engine インスタンスを操作するサンプル

Ansible で Compute Engine インスタンスに SSH 接続するサンプルです。

接続には IAP (Identity-Aware Proxy) TCP Forwarding を使用します。
IAP TCP Forwarding だとパブリックな IP を持たないインスタンスにも接続が可能です。

参考:

- [Identity-Aware Proxy (IAP) | Google Cloud](https://cloud.google.com/iap)
- [Using IAP for TCP forwarding | Identity-Aware Proxy | Google Cloud](https://cloud.google.com/iap/docs/using-tcp-forwarding)

## 必須

- Python 3.9
- Poetry 1.x
- Ansible 4.x

## ディレクトリ構成

```text
collections/
group_vars/
playbooks/
secrets/
ansible.cfg
prod.gcp.yml
pyproject.toml
```

## セットアップ

### Python パッケージをインストール

```bash
poetry install
```

### Ansible コレクションをインストール

```bash
poetry run ansible-galaxy collection install -r collections/requirements.yml
```

このコマンドを実行すると `requirements.yml` で定義されている `google.cloud` プラグイン（ `gcp_compute` インベントリプラグインを含む）が `collections/ansible_collections` 以下にインストールされます。

### インベントリファイルを作成

インベントリファイルの中身を実際の情報に書き換えます。

`prod.gcp.yml`:

```bash
plugin: gcp_compute
zones:
  - asia-northeast1-a
projects:
  - my_project
auth_kind: serviceaccount
service_account_file: ./secrets/gce_key_file.json
hostnames:
  - name
groups:
  prod: "name == 'my_instance'"
strict: yes
```

最低限 `zones` `projects` `groups` を変更する必要があります。

### サービスアカウントファイルを取得

対象のインスタンスの操作ができるサービスアカウントのキーファイルを生成・ダウンロードして `secrets/gce_key_file.json` に保存します。

`secrets/gce_key_file.json`:

```json
{
  "type": "...",
  "project_id": "...",
  "private_key_id": "...",
  "private_key": "...",
  "client_email": "...",
  "client_id": "...",
  "auth_uri": "...",
  "token_uri": "...",
  "auth_provider_x509_cert_url": "...",
  "client_x509_cert_url": "..."
}
```

## 利用

インベントリプラグイン `gcp_compute` でインスタンス情報が正しく取得できる確認します。

```bash
# すべて表示
poetry run ansible-inventory --list 

# `prod` グループのみ表示
poetry run ansible-inventory --host prod
```

サンプルプレイブックを実行します。

```bash
poetry run ansible-playbook playbooks/pwd.yml
```

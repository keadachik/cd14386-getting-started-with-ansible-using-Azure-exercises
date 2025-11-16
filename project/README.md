## 実行手順

事前準備:

```bash
ansible-galaxy collection install ansible.posix
# YAML出力を使いたい場合（任意）
# ansible-galaxy collection install community.general
```

接続確認:

```bash
cd project
ansible -i inventory.ini azure -m ping
```

Playbook 実行:

```bash
ansible-playbook -i inventory.ini playbook.yml
```


## 実行結果の証跡（任意・推奨）

- Playbook 実行ログ要約（例）: `failed=0` が表示されていること
- 2回目実行時に以下のメッセージが表示されること
  - `/var/www/html/www.companyplus.com exists.`


## バージョン情報（任意）

```bash
ansible --version
```

- 注意: 一部の環境で `ansible.posix` コレクションが Ansible 2.14 系に対して警告を出すことがあります。多くの場合は警告のみで実行は可能です。


## 追加の加点要素（任意）

- `document_root/app_root`（デフォルト: `/var/www/html/html_demo_site-main`）に `index.html` を配置して HTTP 疎通確認

```bash
# 例: いずれかのターゲット上で
echo "hello" | sudo tee /var/www/html/html_demo_site-main/index.html

# ローカルから疎通確認
curl http://<VMのIP>/
```

- 使った追加モジュール・条件分岐・フィルタなどの説明を追記（任意）


## 構成の要点

- `inventory.ini`: `azure` グループに 3 ホストを定義
- `group_vars/azure/vars.yml`: `ports: [80, 443]` を定義（firewalld に適用）
- `playbook.yml`:
  - firewalld の導入・起動
  - `ansible.posix.firewalld` を用いて `ports` でループ開放（permanent/immediate）
  - `/var/www/html/www.companyplus.com` の存在チェックと条件メッセージ表示
  - `webserver` ロールを以下の変数で呼び出し
    - `app_root: html_demo_site-main`
    - `server_name: "{{ ansible_default_ipv4.address }}"`
    - `document_root: /var/www/html`
- `roles/webserver`:
  - `vars/main.yml`: `webserver: nginx`
  - `tasks/main.yml`: nginx インストール、2ディレクトリ作成（owner/group=nginx, mode=0770）、テンプレート配置
  - `handlers/main.yml`: テンプレート更新後に nginx 再起動
  - `templates/nginx.conf.j2`: `app_root`, `server_name`, `document_root` を使用



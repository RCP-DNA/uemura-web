# 140.227.126.196 デプロイ手順書

## サーバー情報

- **IPアドレス**: 140.227.126.196
- **現在のWebサーバー**: Apache (httpd)
- **ドキュメントルート**: `/home/uemuraganka-jp/www/uemuraganka.jp/`
- **ポート使用状況**: 80, 443 (Apache使用中)
- **Docker**: 未インストール

---

## 移行の全体像

```
【現在】
www.uemuraganka.jp → 140.227.126.196 (Apache) → HTMLファイル

【移行後】
www.uemuraganka.jp → 140.227.126.196 (Nginx Proxy) → Vercel
                                                    └→ uemura-web-53ym.vercel.app
```

---

## 前提条件チェック

- [ ] rootまたはsudo権限を持つユーザーでログイン可能
- [ ] サーバーにSSH接続可能
- [ ] インターネット接続が安定している
- [ ] メンテナンス時間を確保（推奨：1-2時間）

---

## フェーズ1: Docker環境のセットアップ

### 1-1. サーバーにSSH接続

```bash
ssh -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedKeyTypes=+ssh-rsa root@140.227.126.196
```

### 1-2. システムの更新

```bash
# CentOS/RHEL系の場合
sudo yum update -y

# または、Ubuntu/Debian系の場合
# sudo apt update && sudo apt upgrade -y
```

### 1-3. Dockerのインストール

#### CentOS/RHEL系の場合：

```bash
# 古いバージョンを削除（もしあれば）
sudo yum remove docker docker-client docker-client-latest docker-common \
    docker-latest docker-latest-logrotate docker-logrotate docker-engine

# 必要なパッケージをインストール
sudo yum install -y yum-utils device-mapper-persistent-data lvm2

# Docker公式リポジトリを追加
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# Dockerをインストール
sudo yum install -y docker-ce docker-ce-cli containerd.io

# Dockerを起動
sudo systemctl start docker
sudo systemctl enable docker

# 動作確認
sudo docker --version
sudo docker run hello-world
```

#### Ubuntu/Debian系の場合：

```bash
# 古いバージョンを削除
sudo apt remove docker docker-engine docker.io containerd runc

# 必要なパッケージをインストール
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl gnupg lsb-release

# Docker公式GPGキーを追加
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# リポジトリを追加
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Dockerをインストール
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io

# Dockerを起動
sudo systemctl start docker
sudo systemctl enable docker

# 動作確認
sudo docker --version
sudo docker run hello-world
```

### 1-4. Docker Composeのインストール

```bash
# 最新版をダウンロード
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# 実行権限を付与
sudo chmod +x /usr/local/bin/docker-compose

# シンボリックリンクを作成（オプション）
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

# 動作確認
docker-compose --version
```

---

## フェーズ2: プロキシ設定のデプロイ（テスト）

### 2-1. プロキシ設定ファイルを転送

**ローカルマシン（Mac）から実行：**

```bash
# プロジェクトディレクトリに移動
cd /path/to/uemura-web

# proxyディレクトリをサーバーに転送
scp -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedKeyTypes=+ssh-rsa \
    -r proxy/ root@140.227.126.196:/opt/uemuraganka-proxy/
```

### 2-2. SSL証明書の準備

**サーバー上で実行：**

```bash
cd /opt/uemuraganka-proxy

# sslディレクトリを作成
mkdir -p ssl

# 既存のApacheのSSL証明書を確認
sudo find /etc -name "*.crt" -o -name "*.pem" | grep -i uemuraganka
sudo find /etc -name "*.key" | grep -i uemuraganka

# 既存の証明書をコピー（パスは環境に応じて調整）
# 例：
# sudo cp /etc/httpd/ssl/uemuraganka.jp.crt ssl/fullchain.pem
# sudo cp /etc/httpd/ssl/uemuraganka.jp.key ssl/privkey.pem

# または、Let's Encryptで新規取得する場合：
# （Apacheを一時停止してから実行）
# sudo systemctl stop httpd
# sudo certbot certonly --standalone -d uemuraganka.jp -d www.uemuraganka.jp
# sudo cp /etc/letsencrypt/live/uemuraganka.jp/fullchain.pem ssl/
# sudo cp /etc/letsencrypt/live/uemuraganka.jp/privkey.pem ssl/
# sudo systemctl start httpd

# パーミッション設定
chmod 644 ssl/*.pem
```

### 2-3. テストポートでプロキシを起動

**docker-compose.ymlを一時的に編集：**

```bash
cd /opt/uemuraganka-proxy

# バックアップを作成
cp docker-compose.yml docker-compose.yml.backup

# エディタで編集（viまたはnano）
vi docker-compose.yml
```

**portsセクションを以下のように変更：**

```yaml
    ports:
      - "8080:80"   # テスト用ポート
      - "8443:443"  # テスト用ポート
```

**Dockerコンテナを起動：**

```bash
# コンテナを起動
docker-compose up -d

# ログを確認
docker-compose logs -f

# コンテナが起動しているか確認
docker-compose ps
```

### 2-4. テストポートで動作確認

```bash
# HTTP経由でテスト（8080ポート）
curl -I http://140.227.126.196:8080

# HTTPS経由でテスト（8443ポート）
curl -k -I https://140.227.126.196:8443

# Vercelへの転送が正しく動作しているか確認
curl -I http://140.227.126.196:8080 | grep -i "x-vercel"
```

**ブラウザでもテスト：**
- http://140.227.126.196:8080
- https://140.227.126.196:8443

### 2-5. テストが成功したら、コンテナを停止

```bash
docker-compose down
```

---

## フェーズ3: 本番環境への切り替え

### ⚠️ 注意事項

- **メンテナンス時間を設定することを推奨**
- 可能であれば、患者さんへの事前告知を検討
- バックアップは必ず取得しておく

### 3-1. Apacheのバックアップ

```bash
# Apache設定ファイルのバックアップ
sudo cp -r /etc/httpd /etc/httpd.backup.$(date +%Y%m%d)

# ドキュメントルートのバックアップ（念のため）
sudo tar -czf /root/uemuraganka-backup-$(date +%Y%m%d).tar.gz \
    /home/uemuraganka-jp/www/uemuraganka.jp/
```

### 3-2. docker-compose.ymlを本番ポートに戻す

```bash
cd /opt/uemuraganka-proxy

# バックアップから復元（元のポート設定に戻す）
cp docker-compose.yml.backup docker-compose.yml

# または、手動で編集してportsを以下に変更：
# ports:
#   - "80:80"
#   - "443:443"
```

### 3-3. Apacheを停止

```bash
# Apacheを停止
sudo systemctl stop httpd

# 自動起動を無効化
sudo systemctl disable httpd

# 停止を確認
sudo systemctl status httpd

# ポートが解放されたか確認
sudo netstat -tulpn | grep -E ':80|:443'
# 何も表示されなければOK
```

### 3-4. Nginxプロキシを本番ポートで起動

```bash
cd /opt/uemuraganka-proxy

# コンテナを起動
docker-compose up -d

# ログを確認（エラーがないか）
docker-compose logs -f

# 別のターミナルで動作確認
curl -I https://www.uemuraganka.jp
```

### 3-5. 動作確認

**ブラウザで以下をチェック：**
- https://www.uemuraganka.jp
- https://uemuraganka.jp
- すべてのページ（トップ、設備紹介、白内障、オルソケラトロジー）
- 画像の読み込み
- CSSの適用
- JavaScriptの動作

**コマンドラインで確認：**
```bash
# HTTPSで正常にアクセスできるか
curl -I https://www.uemuraganka.jp

# HTTPからHTTPSへリダイレクトされるか
curl -I http://www.uemuraganka.jp

# Vercelのヘッダーが含まれているか
curl -I https://www.uemuraganka.jp | grep -i vercel
```

---

## トラブルシューティング

### 問題1: SSL証明書エラー

```bash
# SSL証明書の内容を確認
openssl x509 -in ssl/fullchain.pem -text -noout

# 証明書の有効期限を確認
openssl x509 -in ssl/fullchain.pem -noout -dates

# 秘密鍵と証明書が一致しているか確認
openssl x509 -noout -modulus -in ssl/fullchain.pem | openssl md5
openssl rsa -noout -modulus -in ssl/privkey.pem | openssl md5
# 両方のMD5ハッシュが一致すればOK
```

### 問題2: Dockerコンテナが起動しない

```bash
# ログを詳しく確認
docker-compose logs

# Nginxの設定をテスト
docker-compose exec nginx-proxy nginx -t

# コンテナを再起動
docker-compose restart
```

### 問題3: Vercelへの接続エラー

```bash
# コンテナ内から直接テスト
docker exec -it uemuraganka-proxy sh
apk add curl
curl -I https://uemura-web-53ym.vercel.app
exit

# DNS解決を確認
nslookup uemura-web-53ym.vercel.app
```

### 問題4: 切り戻しが必要な場合

```bash
# Nginxプロキシを停止
docker-compose down

# Apacheを再起動
sudo systemctl start httpd
sudo systemctl enable httpd

# 復旧を確認
curl -I https://www.uemuraganka.jp
```

---

## 監視とメンテナンス

### 定期的に確認すべき項目

```bash
# コンテナの状態確認
docker-compose ps

# ログの確認（最新100行）
docker-compose logs --tail=100

# リソース使用状況
docker stats uemuraganka-proxy
```

### SSL証明書の更新（Let's Encrypt使用時）

```bash
# 証明書を更新
sudo certbot renew

# 更新された証明書をコピー
sudo cp /etc/letsencrypt/live/uemuraganka.jp/fullchain.pem /opt/uemuraganka-proxy/ssl/
sudo cp /etc/letsencrypt/live/uemuraganka.jp/privkey.pem /opt/uemuraganka-proxy/ssl/

# Nginxを再起動
cd /opt/uemuraganka-proxy
docker-compose restart
```

### 自動更新の設定（オプション）

```bash
# crontabに追加
sudo crontab -e

# 毎日深夜2時に実行
0 2 * * * certbot renew --quiet && cp /etc/letsencrypt/live/uemuraganka.jp/*.pem /opt/uemuraganka-proxy/ssl/ && cd /opt/uemuraganka-proxy && docker-compose restart
```

---

## チェックリスト

### デプロイ前
- [ ] Dockerがインストールされている
- [ ] Docker Composeがインストールされている
- [ ] SSL証明書が準備できている
- [ ] テストポートで動作確認済み
- [ ] Apacheのバックアップを取得済み
- [ ] ドキュメントルートのバックアップを取得済み

### デプロイ中
- [ ] Apacheを停止した
- [ ] ポート80/443が解放された
- [ ] Dockerコンテナが正常に起動した
- [ ] ログにエラーがない

### デプロイ後
- [ ] HTTPSでアクセスできる
- [ ] HTTPからHTTPSへリダイレクトされる
- [ ] すべてのページが正常に表示される
- [ ] 画像が読み込まれる
- [ ] CSSが適用されている
- [ ] JavaScriptが動作している

---

## 予想所要時間

- **フェーズ1**: Docker環境セットアップ - 20〜30分
- **フェーズ2**: テストポートでの動作確認 - 15〜20分
- **フェーズ3**: 本番切り替え - 10〜15分
- **合計**: 約1〜1.5時間

---

## サポート

問題が発生した場合は、以下の情報を収集してください：

```bash
# システム情報
uname -a
cat /etc/os-release

# Docker情報
docker --version
docker-compose --version
docker-compose ps
docker-compose logs

# ネットワーク情報
sudo netstat -tulpn | grep -E ':80|:443'
curl -I https://www.uemuraganka.jp
```

これらの情報をもとに、トラブルシューティングを行います。

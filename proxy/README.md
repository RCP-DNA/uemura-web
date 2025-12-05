# リバースプロキシ設定（140.227.126.196用）

## 概要

このディレクトリには、140.227.126.196サーバーでVercelへのリバースプロキシを設定するためのDocker構成が含まれています。

```
www.uemuraganka.jp (DNS: 140.227.126.196)
    ↓
140.227.126.196 (Nginxリバースプロキシ)
    ↓
uemura-web-53ym.vercel.app (Vercel)
```

## ファイル構成

```
proxy/
├── docker-compose.yml   # Docker Compose設定
├── nginx.conf          # Nginx設定ファイル
├── ssl/                # SSL証明書ディレクトリ（要作成）
│   ├── fullchain.pem   # SSL証明書（Let's Encryptで取得）
│   └── privkey.pem     # 秘密鍵
└── README.md           # このファイル
```

## 前提条件

- 140.227.126.196 サーバーへのSSHアクセス権限
- Dockerとdocker-composeがインストール済み
- ポート80, 443が利用可能

## セットアップ手順

### 1. サーバーにファイルを転送

```bash
# ローカル（このプロジェクトディレクトリ）から実行
scp -r proxy/ root@140.227.126.196:/opt/uemuraganka-proxy/
```

### 2. サーバーにSSHでログイン

```bash
ssh root@140.227.126.196
cd /opt/uemuraganka-proxy
```

### 3. SSL証明書の準備

#### オプションA：Let's Encryptで新規取得（推奨）

```bash
# sslディレクトリを作成
mkdir -p ssl

# certbotで証明書取得（スタンドアロンモード）
sudo certbot certonly --standalone \
  -d uemuraganka.jp \
  -d www.uemuraganka.jp \
  --email your-email@example.com \
  --agree-tos

# 証明書をコピー
sudo cp /etc/letsencrypt/live/uemuraganka.jp/fullchain.pem ssl/
sudo cp /etc/letsencrypt/live/uemuraganka.jp/privkey.pem ssl/
sudo chmod 644 ssl/*.pem
```

#### オプションB：既存の証明書を使用

既にSSL証明書がある場合は、`ssl/`ディレクトリに配置してください。

```bash
mkdir -p ssl
# 既存の証明書をコピー
cp /path/to/your/fullchain.pem ssl/
cp /path/to/your/privkey.pem ssl/
chmod 644 ssl/*.pem
```

### 4. Dockerコンテナの起動

```bash
# コンテナを起動（バックグラウンド）
docker-compose up -d

# ログを確認
docker-compose logs -f

# 正常に起動しているか確認
docker-compose ps
```

### 5. 動作確認

```bash
# ローカルマシンから確認
curl -I https://www.uemuraganka.jp

# または、ブラウザで以下にアクセス
# https://www.uemuraganka.jp
```

## トラブルシューティング

### エラー: "SSL certificate problem"

```bash
# SSL証明書のパーミッションを確認
ls -la ssl/

# 証明書の内容を確認
openssl x509 -in ssl/fullchain.pem -text -noout
```

### エラー: "Connection refused"

```bash
# Nginxのエラーログを確認
docker-compose logs nginx-proxy

# ポートが使用中か確認
sudo netstat -tulpn | grep :80
sudo netstat -tulpn | grep :443
```

### Vercelへの接続エラー

```bash
# Nginxコンテナ内から直接テスト
docker exec -it uemuraganka-proxy sh
apk add curl
curl -I https://uemura-web-53ym.vercel.app
```

## メンテナンス

### ログの確認

```bash
# リアルタイムログ
docker-compose logs -f

# 最新100行
docker-compose logs --tail=100
```

### 設定の再読み込み

```bash
# nginx.confを編集した場合
docker-compose restart
```

### コンテナの停止・削除

```bash
# 停止
docker-compose stop

# 停止＆削除
docker-compose down

# 停止＆削除（ボリュームも削除）
docker-compose down -v
```

### SSL証明書の更新

Let's Encryptの証明書は90日で期限切れになります。

```bash
# 証明書を更新
sudo certbot renew

# 更新された証明書をコピー
sudo cp /etc/letsencrypt/live/uemuraganka.jp/fullchain.pem ssl/
sudo cp /etc/letsencrypt/live/uemuraganka.jp/privkey.pem ssl/

# Nginxを再起動
docker-compose restart
```

#### 自動更新の設定

```bash
# crontabに追加（毎日深夜2時に実行）
sudo crontab -e

# 以下を追加
0 2 * * * certbot renew --quiet && cp /etc/letsencrypt/live/uemuraganka.jp/*.pem /opt/uemuraganka-proxy/ssl/ && cd /opt/uemuraganka-proxy && docker-compose restart
```

## セキュリティ上の注意

- SSL証明書と秘密鍵は厳重に管理してください
- 定期的にDocker imageを更新してください（`docker-compose pull`）
- ファイアウォールで不要なポートを閉じてください
- 定期的にログを確認してください

## Vercel URLの変更

もしVercel側のURLが変わった場合：

1. `nginx.conf` の以下の部分を更新：
   ```nginx
   proxy_pass https://新しいURL.vercel.app;
   proxy_set_header Host 新しいURL.vercel.app;
   ```

2. Nginxを再起動：
   ```bash
   docker-compose restart
   ```

## 切り戻し（ロールバック）

もし問題が発生した場合、すぐに元に戻せます：

```bash
# プロキシを停止
docker-compose down

# 元のWebサーバーを起動（既存のものがあれば）
# または、一時的にメンテナンス画面を表示
```

## 参考リンク

- [Nginx公式ドキュメント](https://nginx.org/en/docs/)
- [Let's Encrypt公式サイト](https://letsencrypt.org/)
- [Vercel公式ドキュメント](https://vercel.com/docs)

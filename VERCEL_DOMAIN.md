# Vercel ドメイン設定ガイド

## 概要

現在のVercel URL `uemura-web-53ym.vercel.app` を、より分かりやすい名前に変更する方法を説明します。

**重要**: リバースプロキシ経由でアクセスする場合、ユーザーには `www.uemuraganka.jp` が表示されるため、Vercel URLの変更は**必須ではありません**。

---

## オプション1: Vercelで新しいドメインを追加（推奨）

### メリット
- 無料
- 簡単
- すぐに反映される

### 手順

1. **Vercelダッシュボードにアクセス**
   - https://vercel.com/
   - プロジェクト `uemura-web` を選択

2. **Settings → Domains へ移動**

3. **「Add」ボタンをクリック**

4. **好きなドメイン名を入力**
   - 例1: `uemuraganka.vercel.app`
   - 例2: `uemura-ganka.vercel.app`
   - 例3: `uemura-eye-clinic.vercel.app`

5. **「Add」をクリック**
   - Vercelが自動的にDNSを設定
   - 数分で有効になる

6. **nginx.confを更新**（140.227.126.196サーバー）

```bash
# サーバーにSSH接続
ssh root@140.227.126.196

# nginx.confを編集
cd /opt/uemuraganka-proxy
vi nginx.conf
```

**以下の2箇所を新しいドメインに変更：**

```nginx
# 変更前
proxy_pass https://uemura-web-53ym.vercel.app;
proxy_set_header Host uemura-web-53ym.vercel.app;

# 変更後（例：uemuraganka.vercel.app に変更した場合）
proxy_pass https://uemuraganka.vercel.app;
proxy_set_header Host uemuraganka.vercel.app;
```

**Nginxを再起動：**

```bash
docker-compose restart
```

7. **動作確認**

```bash
curl -I https://www.uemuraganka.jp
```

---

## オプション2: カスタムドメインを設定（上級者向け）

### 前提条件
- 独自ドメインを所有している
- DNSレコードを編集できる

### メリット
- Vercelを完全に隠せる
- 独自ブランディング

### デメリット
- DNS設定が必要
- 反映に時間がかかる場合がある

### 手順（例: ドメイン `uemuraganka.com` を持っている場合）

1. **Vercelダッシュボードでカスタムドメインを追加**
   - Settings → Domains
   - 「Add」をクリック
   - `uemuraganka.com` または `vercel.uemuraganka.com` を入力

2. **Vercelが提供するDNS設定値を確認**
   - 例: `cname.vercel-dns.com`
   - または `76.76.21.21`（Aレコード）

3. **DNS管理画面で設定を追加**
   - お名前.com、ムームードメインなど
   - CNAMEレコードまたはAレコードを追加

4. **Vercelで「Verify」をクリック**
   - DNS伝播を待つ（最大48時間）

5. **nginx.confを更新**
   - 新しいカスタムドメインに変更

---

## オプション3: 何もしない（現状維持）

### メリット
- 作業不要
- リスクゼロ

### デメリット
- Vercel URLが分かりにくい

### 推奨理由
- **ユーザーには見えない**
- リバースプロキシ経由なので、常に `www.uemuraganka.jp` が表示される
- 内部的なURLなので、分かりにくくても問題ない

---

## 推奨アプローチ

### 短期的（今すぐ）
**オプション3: 何もしない**
- 現在のURL（`uemura-web-53ym.vercel.app`）のまま運用
- ユーザーには影響なし

### 中期的（時間があるとき）
**オプション1: Vercelで新しいドメインを追加**
- `uemuraganka.vercel.app` などに変更
- 管理しやすくなる

### 長期的（リニューアル時）
**オプション2: カスタムドメイン設定**
- 将来的にリバースプロキシを廃止する場合
- 完全にVercelに移行する場合

---

## FAQ

### Q: Vercel URLを変更すると、何か影響がありますか？

**A**: リバースプロキシ経由の場合、ユーザーには全く影響ありません。`nginx.conf` を更新すれば、すぐに切り替わります。

### Q: 複数のドメインを設定できますか？

**A**: はい、Vercelでは複数のドメインを同じプロジェクトに設定できます。

例：
- `uemuraganka.vercel.app`（メイン）
- `uemura-web-53ym.vercel.app`（旧URL、自動リダイレクト）

### Q: 古いURL（uemura-web-53ym.vercel.app）はどうなりますか？

**A**: 新しいドメインを追加しても、古いURLは引き続き使えます。削除しない限り、両方同時に動作します。

### Q: DNS変更後、すぐに反映されますか？

**A**: 
- Vercelドメイン（`.vercel.app`）: 数分で反映
- カスタムドメイン: 最大48時間（通常は数時間）

### Q: SSL証明書はどうなりますか？

**A**: Vercelが自動的にSSL証明書を発行・更新します。追加作業は不要です。

---

## まとめ

| オプション | 難易度 | 所要時間 | コスト | 推奨度 |
|-----------|--------|---------|--------|--------|
| オプション1: Vercelドメイン追加 | ⭐️ 簡単 | 10分 | 無料 | ⭐️⭐️⭐️⭐️ |
| オプション2: カスタムドメイン | ⭐️⭐️⭐️ 難しい | 1〜2日 | 無料 | ⭐️⭐️ |
| オプション3: 何もしない | - | 0分 | 無料 | ⭐️⭐️⭐️⭐️⭐️ |

**結論**: まずは**オプション3（何もしない）**で運用開始し、必要に応じて**オプション1**に移行することをお勧めします。

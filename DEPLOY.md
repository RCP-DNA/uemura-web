# Vercelデプロイ手順書

## 概要

このドキュメントでは、綾瀬うえむら眼科WebサイトをGitHub経由でVercelにデプロイする手順を説明します。

## 前提条件

- GitHubアカウントを持っていること
- Vercelアカウントを持っていること（無料プランで可）
- リポジトリがGitHubにプッシュされていること

## 初回デプロイ手順

### 1. GitHubにリポジトリをプッシュ

まず、このプロジェクトをGitHubにプッシュする必要があります。

```bash
# 現在のブランチを確認
git branch

# mainブランチに切り替え（必要な場合）
git checkout -b main

# GitHubにリモートリポジトリを追加（まだの場合）
git remote add origin https://github.com/YOUR_USERNAME/uemura-web.git

# プッシュ
git push -u origin main
```

### 2. Vercelアカウントにログイン

1. https://vercel.com/ にアクセス
2. 「Sign Up」または「Log In」をクリック
3. GitHubアカウントでログインすることを推奨

### 3. 新しいプロジェクトを作成

1. Vercelダッシュボードで「Add New...」→「Project」をクリック
2. 「Import Git Repository」セクションで、GitHubリポジトリを選択
3. リポジトリが表示されない場合は「Adjust GitHub App Permissions」をクリックしてアクセス許可を設定

### 4. プロジェクト設定

以下の設定を確認・入力します：

- **Project Name**: `uemura-ganka` など任意の名前
- **Framework Preset**: `Other`（静的サイトのため）
- **Root Directory**: `./`（デフォルトのまま）
- **Build Command**: 空欄（静的サイトのため不要）
- **Output Directory**: `./`（デフォルトのまま）
- **Install Command**: 空欄（不要）

### 5. 環境変数の設定（必要な場合）

このプロジェクトでは環境変数は不要ですが、将来的に必要になった場合は：

1. 「Environment Variables」セクションで追加
2. 変数名と値を入力
3. 「Production」「Preview」「Development」のどれに適用するか選択

### 6. デプロイ

「Deploy」ボタンをクリックすると、自動的にデプロイが開始されます。

デプロイが完了すると：
- 本番URL（例: `https://uemura-ganka.vercel.app`）が発行されます
- ドメイン名はVercelダッシュボードから変更可能です

## 自動デプロイの仕組み

Vercelは以下のルールで自動デプロイされます：

- **mainブランチへのプッシュ** → 本番環境（Production）に自動デプロイ
- **他のブランチへのプッシュ** → プレビュー環境（Preview）に自動デプロイ
- **プルリクエスト作成** → プレビューURLが自動生成

## 更新方法

プロジェクトを更新する場合は、以下の手順で行います：

```bash
# ファイルを編集

# 変更をステージング
git add .

# コミット
git commit -m "変更内容の説明"

# GitHubにプッシュ
git push origin main
```

GitHubにプッシュすると、Vercelが自動的に検知して再デプロイを開始します。

## カスタムドメインの設定

1. Vercelダッシュボードで該当プロジェクトを選択
2. 「Settings」→「Domains」をクリック
3. カスタムドメイン（例: `www.uemuraganka.jp`）を入力
4. 表示されるDNS設定をドメイン管理サービス（お名前.comなど）で設定
   - Aレコードまたは CNAMEレコード を追加

### DNS設定例

Vercelが提供する値を使用：

```
Type: A
Name: @
Value: 76.76.21.21

Type: CNAME
Name: www
Value: cname.vercel-dns.com
```

## トラブルシューティング

### デプロイが失敗する場合

1. Vercelダッシュボードでエラーログを確認
2. `vercel.json`の設定を見直し
3. ファイルパスが正しいか確認（大文字小文字に注意）

### ページが正しく表示されない場合

1. ブラウザのキャッシュをクリア
2. 画像やCSSのパスが相対パスになっているか確認
3. Vercelダッシュボードの「Deployment」ログで404エラーがないか確認

### カスタムドメインが反映されない場合

1. DNS設定が正しいか確認（設定反映には最大48時間かかる場合があります）
2. Vercelダッシュボードで「Verify」ボタンをクリック
3. SSL証明書の発行を待つ（自動で発行されます）

## ロールバック方法

間違った変更をデプロイしてしまった場合：

1. Vercelダッシュボードの「Deployments」タブを開く
2. 以前の正常なデプロイを選択
3. 「...」メニューから「Promote to Production」を選択

または、Gitで以前のコミットに戻す：

```bash
# 以前のコミットを確認
git log

# 特定のコミットに戻す
git revert <commit-hash>

# プッシュ
git push origin main
```

## セキュリティ設定

`vercel.json`には以下のセキュリティヘッダーが設定されています：

- `X-Content-Type-Options: nosniff`
- `X-Frame-Options: SAMEORIGIN`
- `X-XSS-Protection: 1; mode=block`

必要に応じてVercelダッシュボードから追加のセキュリティ設定が可能です。

## パフォーマンス最適化

Vercelでは以下が自動的に最適化されます：

- CDNによる配信
- 画像の最適化（Next.jsの場合）
- Gzip/Brotli圧縮
- キャッシュ制御

`vercel.json`には、画像やCSS/JSファイルに対して長期キャッシュ設定が適用されています。

## 参考リンク

- [Vercel公式ドキュメント](https://vercel.com/docs)
- [Vercel CLI](https://vercel.com/docs/cli)
- [カスタムドメイン設定ガイド](https://vercel.com/docs/concepts/projects/custom-domains)

## サポート

問題が解決しない場合は、Vercelのサポートチャンネルまたはドキュメントを参照してください。

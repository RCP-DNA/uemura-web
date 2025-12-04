# 綾瀬うえむら眼科Webサイト プロジェクト

## プロジェクト概要

綾瀬うえむら眼科の公式Webサイトプロジェクトです。  
静的なHTML/CSS/JavaScriptで構成されたWebサイトです。

- **プロジェクト名**: 綾瀬うえむら眼科Webサイト
- **プロジェクトパス**: `/Users/tkadokura/Dev/uemura-web`
- **本番URL**: http://www.uemuraganka.jp/
- **プロジェクト管理基準日**: 2025年11月30日（この日から本格的な運用・デプロイ管理を開始）

## プロジェクト構造

```
uemura-web/
├── index.html              # メインページ
├── common/                 # 共通リソース
│   ├── css/               # 共通CSS
│   └── images/            # 共通画像
├── equipment/             # 設備紹介ページ
│   ├── css/
│   ├── images/
│   └── index.html
├── hakunaisho/            # 白内障関連ページ
│   ├── css/
│   ├── images/
│   └── index.html
├── oltho/                 # オルソケラトロジー関連ページ
│   ├── css/
│   ├── images/
│   └── index.html
├── js/
│   └── slick/             # Slickスライダーライブラリ
├── images/                # メイン画像
├── pdf/                   # PDFファイル
└── sitemap.xml            # サイトマップ
```

## リモートサーバー同期設定

### サーバー情報

- **IPアドレス**: `140.227.126.196`
- **サーバープロバイダー**: Creative Japan
- **SSHユーザー**: `root`
- **ドキュメントルート**: `/home/uemu...`（詳細はサーバー設定を確認）
- **SSH接続オプション**: 
  ```bash
  -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedKeyTypes=+ssh-rsa
  ```

### 同期の経緯

1. **2025年11月30日 3:16:41 AM**: リモートサーバー同期設定が記録される
2. **2025年11月30日 6:26:22 AM**: プロジェクト管理基準日が設定される
3. **2025年11月30日 12:38:25**: Git初期コミット（"Initial commit: Sync from production server"）

本番サーバー（uemuraganka.jp）からローカル開発環境への初回同期が2025年11月30日に実施され、この日をプロジェクト管理の基準日として設定しました。

### 同期方法

本プロジェクトは、共通設定ファイル（`~/.cursor-deploy.conf`）に登録されているサーバー設定を使用します。

**利用可能なサーバー**:
- サーバー1: さくらのVPS (160.16.213.57)
- サーバー2: Creative Japan (140.227.126.196) ← 本プロジェクトで使用

## 運用方針

### 開発環境

- **開発ツール**: Cursor IDE
- **バージョン管理**: Git
- **共通設定**: `~/.cursor-deploy.conf`（全プロジェクト共通）
- **プロジェクト固有設定**: `.deploy.conf`（必要に応じて作成）

### デプロイメント

#### Vercelデプロイ（推奨）

- **方針**: GitHubのmainブランチをVercelに自動デプロイ
- **設定ファイル**: `vercel.json`
- **デプロイトリガー**: mainブランチへのプッシュで自動デプロイ
- **プレビュー環境**: プルリクエスト作成時に自動生成
- **詳細手順**: [DEPLOY.md](./DEPLOY.md) を参照

#### 従来の本番サーバー（Creative Japan）

- 本番サーバーへのデプロイは、rsyncを使用した同期方式を採用
- SSH鍵認証による接続
- デプロイ前には必ずバックアップを取得
- **注意**: 将来的にはVercelへの移行を推奨

### 注意事項

- 共通設定ファイルの自動検出とByteRover連携は、グローバルルール（`~/.cursor/rules/`）で管理されています
- プロジェクトの`.cursorrules`はチーム共有のためGit管理に含めます

## 技術スタック

- **HTML5**: 静的HTML
- **CSS3**: カスタムCSS + Slickスライダー用CSS
- **JavaScript**: 
  - jQuery 3.6.0
  - Slick Carousel 1.8.1
  - jQuery LazyLoad 1.9.1

## Git履歴

- **初期コミット**: 2025年11月30日 12:38:25
- **コミッター**: RCP-DNA <182110336+RCP-DNA@users.noreply.github.com>
- **コミットメッセージ**: "Initial commit: Sync from production server"

## 関連ドキュメント

- `.cursorrules`: Cursor IDE用のプロジェクトルール
- `sitemap.xml`: サイトマップ（最終更新: 2014-09-08）
- `vercel.json`: Vercelデプロイ設定ファイル
- `DEPLOY.md`: Vercelへのデプロイ手順書

## 更新履歴

- **2025年11月30日**: プロジェクト作成、本番サーバーからの初回同期完了
- **2025年11月30日**: Gitリポジトリ初期化、プロジェクト管理基準日設定
- **2025年12月4日**: Vercelデプロイ設定追加（vercel.json, DEPLOY.md作成）

---

**注意**: このREADME.mdは、ByteRoverのメモリに記録されている情報を基に作成されました。  
詳細な同期設定や運用手順については、共通設定ファイル（`~/.cursor-deploy.conf`）を参照してください。


# 綾瀬うえむら眼科 Webサイト

綾瀬うえむら眼科の公式Webサイトプロジェクトです。

## プロジェクト概要

静的なHTML/CSS/JavaScriptで構成されたWebサイトです。

## プロジェクト構造

```
.
├── index.html                 # メインページ
├── common/                    # 共通リソース
│   ├── css/
│   │   └── common.css        # 共通スタイル
│   └── images/               # 共通画像
├── css/
│   └── top.css               # トップページスタイル
├── equipment/                # 設備紹介ページ
│   ├── index.html
│   ├── css/
│   └── images/
├── hakunaisho/               # 白内障関連ページ
│   ├── index.html
│   ├── css/
│   └── images/
├── oltho/                    # オルソケラトロジー関連ページ
│   ├── index.html
│   ├── css/
│   └── images/
├── js/
│   └── slick/                # Slickスライダーライブラリ
├── images/                   # メインページ画像
├── pdf/                      # PDFファイル
└── sitemap.xml               # サイトマップ
```

## 主要ページ

- **トップページ** (`index.html`) - クリニックの基本情報、スライダー、お知らせなど
- **設備紹介** (`equipment/`) - 医療設備の紹介
- **白内障治療** (`hakunaisho/`) - 白内障に関する情報
- **オルソケラトロジー** (`oltho/`) - オルソケラトロジー治療に関する情報

## 使用技術

- HTML5
- CSS3
- JavaScript
- [Slick Slider](http://kenwheeler.github.io/slick/) - 画像スライダー

## ローカル環境での起動方法

このプロジェクトは静的なHTMLファイルで構成されているため、特別なビルドプロセスは不要です。

### 方法1: ブラウザで直接開く

```bash
# ブラウザでindex.htmlを開く
open index.html
```

### 方法2: ローカルサーバーを起動（推奨）

Pythonを使用する場合：

```bash
# Python 3の場合
python3 -m http.server 8000

# Python 2の場合
python -m SimpleHTTPServer 8000
```

Node.jsのhttp-serverを使用する場合：

```bash
# http-serverをグローバルインストール
npm install -g http-server

# サーバーを起動
http-server -p 8000
```

その後、ブラウザで `http://localhost:8000` にアクセスしてください。

## バックアップファイルについて

- `bk/` - バックアップディレクトリ
- `index_bk20210725.html` - 過去のバックアップファイル
- `index_bk20230122.html` - 過去のバックアップファイル
- `index_test.html` - テスト用ファイル

## Google Search Console

- `google344b263765866182.html` - Google Search Console認証ファイル
- `googleb72e84f598044b52.html` - Google Search Console認証ファイル

## ライセンス

© 綾瀬うえむら眼科. All rights reserved.

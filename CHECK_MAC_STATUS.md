# Mac側のGit状態確認方法

## 最新状態の確認コマンド

MacのCursorIDEのターミナルで、プロジェクトのルートディレクトリで以下を実行してください：

```bash
# 1. リモートの最新情報を取得
git fetch origin

# 2. 現在のブランチと状態を確認
git status

# 3. ローカルmainブランチとGitHubのorigin/mainの差分を確認
git log main..origin/main --oneline

# 4. ローカルmainブランチがGitHubより何コミット遅れているか確認
git rev-list --count main..origin/main

# 5. すべてのブランチの状態を確認
git branch -a -vv

# 6. GitHubの最新コミットを確認
git log origin/main --oneline -10
```

## 現在の状況まとめ（リモートワークスペース側）

### 最新の状態
- **GitHub (origin/main)**: `0aa9451` - 44分前（最新）
- **リモートワークスペースのローカルmain**: `1cc88e8` - 5日前（7コミット遅れ）
- **現在のブランチ**: `0aa9451` - GitHubと同期済み

### GitHubに追加された最新ファイル
- `.cursorrules` (更新)
- `.gitignore` (更新)
- `AGENTS.md` (新規)
- `DEPLOY.md` (新規)
- `README.md` (新規)
- `VERCEL_DOMAIN.md` (新規)
- `proxy/DEPLOYMENT_GUIDE.md` (新規)
- `proxy/README.md` (新規)
- `proxy/docker-compose.yml` (新規)
- `proxy/nginx.conf` (新規)
- `vercel.json` (新規)

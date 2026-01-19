# Documentation Site

このフォルダには、GitHub Pages用のドキュメントサイトの設定が含まれています。

## ローカルでプレビュー

```bash
# 依存関係をインストール
pip install -r requirements.txt

# 開発サーバーを起動
mkdocs serve
```

ブラウザで http://127.0.0.1:8000 にアクセスしてプレビューできます。

## デプロイ

mainブランチにpushすると、GitHub Actionsが自動でGitHub Pagesにデプロイします。

## 使用技術

- **MkDocs**: 静的サイトジェネレーター
- **Material for MkDocs**: 美しいテーマ
- **GitHub Actions**: 自動デプロイ
- **GitHub Pages**: ホスティング

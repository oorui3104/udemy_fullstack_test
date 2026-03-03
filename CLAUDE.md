# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 概要

Udemyフルスタックエンジニア学習用リポジトリ。PHP 8.3 + nginx + MySQL を Docker で動かす Laravel プロジェクト環境。Laravelアプリ本体は `src/` 以下に配置。

## 構成

| サービス | 内容 |
|----------|------|
| `app` | PHP 8.3-fpm + Composer 2.7 + Node.js 24 |
| `web` | nginx 1.25-alpine (ポート80) |
| `db` | MySQL 8.0 (ポート3306/外部3306) |

Viteの開発サーバーはポート5173で外部公開。テスト環境ではSQLite in-memoryを使用。

## よく使うコマンド

### Docker操作

```bash
# 起動
docker-compose up -d

# コンテナ確認
docker-compose ps

# ログ確認
docker-compose logs -f

# PHPコンテナに入る
docker-compose exec app bash

# 停止
docker-compose down

# 完全削除（DBボリュームも削除）
docker-compose down -v
```

### Laravelコマンド（コンテナ内から実行）

```bash
# 依存関係インストール
docker-compose exec app composer install

# マイグレーション
docker-compose exec app php artisan migrate

# シーダー実行
docker-compose exec app php artisan db:seed

# キャッシュクリア
docker-compose exec app php artisan cache:clear
docker-compose exec app php artisan config:clear

# ルート確認
docker-compose exec app php artisan route:list
```

### テスト実行

```bash
# 全テスト実行
docker-compose exec app php artisan test

# 単一テストファイルの実行
docker-compose exec app php artisan test tests/Feature/ExampleTest.php

# PHPUnitで直接実行
docker-compose exec app ./vendor/bin/phpunit

# 特定テストクラス・メソッドの実行
docker-compose exec app ./vendor/bin/phpunit --filter "テストメソッド名"
```

### フロントエンド（Vite）

```bash
# 開発サーバー起動（コンテナ内）
docker-compose exec app npm run dev

# ビルド
docker-compose exec app npm run build
```

## アーキテクチャ

- **ルーティング**: `src/routes/web.php`（Web）、`src/routes/console.php`（コンソール）
- **コントローラー**: `src/app/Http/Controllers/`
- **モデル**: `src/app/Models/`
- **マイグレーション**: `src/database/migrations/`
- **ビュー**: `src/resources/views/`（Bladeテンプレート）
- **フロントエンド**: `src/resources/css/app.css` + `src/resources/js/app.js`（Tailwind CSS v4 + Vite）

## 本番環境

Aurora MySQL を使用（ローカルの `db` サービスは使用しない）。

```bash
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

本番では `DB_HOST`, `DB_PORT` 等の環境変数を外部から注入。

## Tailwind CSS v4を使う場合のvite.config.js設定

DockerコンテナからViteのHMRにアクセスするには、`src/vite.config.js` の `server` セクションに以下を追加：

```js
server: {
    host: "0.0.0.0",
    port: 5173,
    strictPort: true,
    hmr: {
        host: "localhost",
        port: 5173,
    },
},
```

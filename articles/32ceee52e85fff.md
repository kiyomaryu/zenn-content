---
title: "windows10 + wsl2(ubuntu20.04) + ruby(2.7.6) +rails(6.1.0)のコンテナ開発環境構築メモ"
emoji: "🦔"
type: "tech"
topics:
  - "ruby"
published: true
published_at: "2022-05-22 23:00"
---

# 概要
記録として記載します。詳細は追記します。

## 環境
- windows10+WSL2(Ubuntu20.04)
- docker composeコマンドはv2を使用
- dockerはwindows上ではなくてwsl2上に導入済み。vscodeでwsl2にリモート接続して開発

## コンテナでrailsが起動するまでの流れ

### Dockerfile作成
・docker-compose.yml
```
version: '3'
services:
  db:
    image: mysql:8.0
    command: --default-authentication-plugin=mysql_native_password
    volumes:
      - ./src/db/mysql_data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: passW0rd
  web:
    build: .
    command: bundle exec rails s -p 3000 -b '0.0.0.0'
    volumes:
      - ./src:/app
    ports:
      - "3000:3000"
    depends_on:
      - db
```
### Gemfile作成(railsインストール用)
```
source 'https://rubygems.org'

ruby '2.7.6'

gem 'rails', '~> 6.1.0'
```

構成
```
[kyo@DESKTOP-EFHIULV rails-docker]$ tree
.
├── Dockerfile
├── docker-compose.yml
└── src
    └── Gemfile

```

### rails newコマンドにてテンプレートの生成(scaffold)
rails newコマンドでテンプレ生成
```=bash
docker compose run web rails new . --force --database=mysql
```

### Gemfileが上書きされて新しくなったのでbuild
```
# ↑のdocker compose runで起動時にマウントされたときにホスト側にDBファイルができているが不要なので削除
sudo rm -rf src/db/mysql_data/*
# build
docker compose build
```

### DBの設定
・config/database.yml
```
default: &default
  adapter: mysql2
  encoding: utf8mb4
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  username: root
  password: passW0rd←★docker-compose.ymlに記載のパスワード追加
  host: db ←★コンテナ名に変更
```

### rails db:createにてdbの作成
```
docker compose run web rails db:create
```

### 起動
```
docker compose up -d
```

## 困ったときメモ
### railsをコンテナで動かす時のポイント

- なにかソースに変更を加えたらdocker compose buildにてビルド
- rails db:createでDBのコンテナに接続できないときは古いmysqlのデータがvolumesに残っているため、一度mysqlのデータをdocker compose down --volumusにて削除してからbuildしてrails db:create打つとうまくいく。

### アクセスしてみる
http://localhost:3000/






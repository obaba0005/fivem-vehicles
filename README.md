# FiveM車両ダッシュボード

FiveMの車両を、写真と価格のカードで一覧できる共有ダッシュボード（静的サイト＋Supabase）です。

## 概要

- 形態：静的サイト（GitHub Pages）＋ Supabase
- ビルド工程なし。`index.html` に HTML / CSS / JavaScript をまとめた単一ファイル構成です
- Supabase JS は CDN（`@supabase/supabase-js@2`）から読み込みます
- 車両の項目は3つだけです
  - `name`（text、必須）— 車両名
  - `price`（bigint、任意）— 空欄可。空欄は「価格なし」と表示
  - `image`（任意）— 登録時にアップロードする
  - スポーンコード、メーカー、クラスは入れません
- 価格は `$` 表記でカンマ区切り表示します
- 既定はダークテーマです

## 機能

- 登録（画像アップロードつき）
- 編集
- 削除（削除前に確認ダイアログを表示）
- 名前検索（車両名の部分一致）
- 並び替え（新着順、価格が高い順、価格が安い順、名前順）
- 総台数の表示
- リアルタイム反映（共有モードで `postgres_changes` を購読し、変更時に一覧を再読み込み）
- 空状態の案内表示（未登録時に登録を促す文言を表示）
- 新規登録と編集はモーダルで行います

## ファイル構成

- `index.html` — ダッシュボード本体の単一ファイル
- `config.js` — Supabase 接続情報
- `README.md` — このファイル（概要と再設定手順のメモ）

## モードについて

`window.DASHBOARD_CONFIG`（`config.js`）を読み、Supabase URL と anon key の両方が入っている場合は共有モード、どちらかが空の場合はローカル確認モードになります。

- 共有モード：`vehicles` テーブルを `created_at` 降順で取得します。画像は Storage の `vehicle-images` バケットへアップロードし、公開URLを `image_url` に保存します。ファイル名は日時などを含めて一意にします
- ローカル確認モード：データを `localStorage` に保存し、画面上に「ローカル確認モード」である旨のバナーを表示します。画像は Data URL としてブラウザ内だけで完結させます

現在の `config.js` は空のプレースホルダです。Supabase プロジェクトを作成するまではローカル確認モードで表示と操作の検証ができます。

## 再設定手順（Supabase に接続するとき）

1. Supabase でプロジェクトを作成し、SQL Editor で `schema.sql`（`public.vehicles` テーブル、Row Level Security、公開ストレージバケット `vehicle-images` など）を実行し、Realtime を有効化します
2. Project Settings > API から Project URL と anon public key を取得します
3. `config.js` を次の形式で実際の値に置き換えます

```js
window.DASHBOARD_CONFIG = {
  SUPABASE_URL: "https://<PROJECT_REF>.supabase.co",
  SUPABASE_ANON_KEY: "<SUPABASE_ANON_KEY>"
};
```

4. `config.js` をリポジトリへ再アップロードし、GitHub Pages の再デプロイを待ってから公開URLで確認します
5. ローカル確認モードのバナーが出ていないこと（共有モードで動いていること）を確認します

## セキュリティについての注意

- anon key はクライアント公開前提の設計です。`config.js` に入れて公開しても、anon key 自体が秘密鍵ではありません
- アクセス制御はデータベース側のポリシー（Row Level Security および Storage ポリシー）で行う前提です
- 既定のポリシーは書き込み全公開です。URLを知る人は誰でも登録・編集・削除できます。不特定多数へ広く公開する用途では、この設定のまま使わないでください

## 注意

- 車両名は表示時にエスケープし、HTMLとして解釈させません
- Supabase Free プランは無活動が続くと自動で一時停止することがあります。長期運用する場合はキープアライブの設定を検討してください

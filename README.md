# npm-win

Windows 上で vitest を**完全オフライン**でインストールするための資源を、GitHub Actions で生成するリポジトリです。

## 何ができるか

`push`（`package.json` / `package-lock.json` の変更時）または手動の `Run workflow` で、
**`vitest-offline-windows`** という名前のアーティファクトを生成します。

アーティファクトの中身:

```
vitest-offline-win/
├── package.json          # 依存定義と同一のもの
├── package-lock.json     # ロックファイル（platform 非依存で全バイナリ込み）
└── npm-cache.tar.gz      # npm キャッシュ一式（npm config get cache の丸ごと）
```

ワークフローは次のことを**ビルド中に検証**します。

1. Windows ランナーで `npm ci` し、npm キャッシュを充填
2. そのキャッシュだけを使って `npm ci --offline` で依存を再構築（オフラインで成立することを CI 内で確認）
3. `npx vitest --version` で vitest がオフライン状態で起動することを確認

つまり「このアーティファクトを持っていけばオフラインでも vitest を入れられる」ということが、生成時点で保証されています。

## オフライン環境（Windows）での使い方

1. アーティファクト `vitest-offline-windows` をダウンロードして任意の場所に展開
2. `npm-cache.tar.gz` を展開してキャッシュフォルダを作る
3. npm にそのキャッシュを使わせる:

   ```powershell
   # 環境変数で指定（コマンドごとではなくグローバルに）
   setx NPM_CONFIG_CACHE "C:\offline\npm-cache"
   # または現在のセッションだけ
   $env:NPM_CONFIG_CACHE = "C:\offline\npm-cache"
   ```

4. 対象プロジェクトに `package.json` / `package-lock.json` をコピーして、完全オフラインでインストール:

   ```powershell
   npm ci --offline
   ```

   `npm ci` は lockfile を完全に信頼するので、ネット接続がなくてもキャッシュから確実に再現されます。

## メモ

- vitest は `^22.12.0 || ^24 || >=26` の Node を要求します。
  **Node 22 系でお使いの場合は v22.12 以上が必須**です（22.11 以前では動きません）。オフライン先では `node -v` で 22.12+ を確認してください。
- このワークフローは `node-version: 22` でキャッシュを生成します。`setup-node@v4` の `22` は Node 22 の最新マイナーを引くので常に 22.12 以上が保証され、オフライン先の Node 22 系と互換です。
- Windows ランナーで生成するため、ネイティブバイナリ（`@rolldown/binding-win32-x64-msvc`, `lightningcss-win32-x64-msvc`）が含まれます。**64bit の Windows** を対象としています。

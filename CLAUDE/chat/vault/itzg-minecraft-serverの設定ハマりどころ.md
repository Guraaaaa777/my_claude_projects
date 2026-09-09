---
created: 2026-09-09T12:31:09 +09:00
updated: 2026-09-09T12:36:00 +09:00
tags: [Minecraft,サーバー,podman,Docker,トラブルシュート]
type: ナレッジ
author: Claude Code
---

# itzg-minecraft-serverの設定ハマりどころ

[[Minecraftサーバー]] で Geyser/Floodgate を導入した際に実際に詰まった点の記録。

## 1. MEMORY に `GB` は使えない

`MEMORY: "15GB"` と書くと `-Xmx15GB` が生成され、JVM が `Invalid maximum heap size` で起動不能になる。
JVM が受け付ける単位は `K` / `M` / `G` / `T` の1文字のみ。正しくは `MEMORY: "15G"`。

`restart: unless-stopped` と組み合わさると延々と再起動ループするため、ログが同じ内容で埋まる。

## 2. Geyser は Modrinth 上で全ビルドが beta

`MODRINTH_PROJECTS` のデフォルトは release のみ許可なので、Geyser は候補が全滅して
`No candidate versions of 'Geyser' [...] matched versionType=release` になる。

`MODRINTH_ALLOWED_VERSION_TYPE: "beta"` が必要。

## 3. ボリュームのコンテナ側パスは絶対パス必須

`./patches:patches:ro` は不正。`./patches:/patches:ro` と書く。

## 4. patches はマウントしただけでは動かない

`PATCH_DEFINITIONS: "/patches"` を環境変数で指定しないと、`/patches` の中身は完全に無視される。

## 5. patch JSON の形式はディレクトリ指定かファイル指定かで違う

`PATCH_DEFINITIONS` がディレクトリの場合、その中の各 JSON は「1ファイル = 1 PatchDefinition」。
`{"patches": [...]}` でくるむとエラーになる。有効キーは `file` / `file-format` / `ops` の3つ。

```json
{
  "file": "/data/config/Geyser-NeoForge/config.yml",
  "file-format": "yaml",
  "ops": [
    { "$set": { "path": "$.java.auth-type", "value": "floodgate" } }
  ]
}
```

なお patch ツールは YAML を読み込んで書き戻すため、**元のコメントが全部消える**。

## 6. Geyser の floodgate-key-file はデフォルトが相対パス

デフォルト値 `key.pem` は Geyser 自身のフォルダ基準で解決されるが、
Floodgate が鍵を生成するのは `/data/config/floodgate/key.pem` で別ディレクトリ。
参照先が存在しない状態になるので、絶対パスを明示する。

`key.pem` が 16 バイトなのは正常（Floodgate は AES-128 の生の鍵）。

## 8. Geyser の config.yml には `java:` が2箇所ある

- トップレベルの `java:` → `auth-type` が入る
- `advanced:` 配下の `java:` → `use-haproxy-protocol` など

JSONPath はそれぞれ以下になる。

- `$.java.auth-type`
- `$.advanced.floodgate-key-file`

インデントだけ見て親を推測すると間違えるので、`cat` で全文の階層を確認してから書く。

## 7. 環境変数の変更に restart は効かない

`podman-compose restart` は既存コンテナをそのまま再起動するだけで、compose の環境変数変更は反映されない。
`down && up -d` が必要。

一方、ボリューム内やバインドマウント先のファイル変更は restart で反映される。

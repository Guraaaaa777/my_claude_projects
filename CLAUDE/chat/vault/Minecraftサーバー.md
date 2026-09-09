---
created: 2026-09-09T12:31:09 +09:00
updated: 2026-09-09T12:31:09 +09:00
tags: [Minecraft,サーバー,podman,ゲーム]
type: 環境
author: Claude Code
---

# Minecraftサーバー

## 構成

2026-09-09 時点の自宅サーバー構成。

- ホスト: `mini12s`
- ディレクトリ: `~/docker-minecraft-server-closs`
- コンテナ管理: podman-compose（コンテナ名 `docker-minecraft-server-closs_mc_1`）
- イメージ: `itzg/minecraft-server`
- サーバー種別: NeoForge 26.2.0.82 / Minecraft 26.2
- 割り当てメモリ: 15G
- 難易度: hard / MOTD: `ﾅﾁｮ‐`
- ポート: 25565（Java）, 19132/udp（Bedrock）

## Bedrock対応

Geyser + Floodgate を導入し、Bedrock版クライアントからも参加できるようにしている。

- Geyser 2.11.2-b1234
- 導入方法は `MODRINTH_PROJECTS`（geyser, floodgate）
- 設定の恒久化は `PATCH_DEFINITIONS` + `patches/geyser.json`

導入時のハマりどころは [[itzg-minecraft-serverの設定ハマりどころ]] に分離して記録。

## 出来事

- 2026-09-09: Geyser/Floodgate の導入に着手。mods に Geyser しか入らない状態から、原因を順に切り分けて Bedrock からの接続に成功
- 最後まで残った症状「Java版のアカウントとしてログインしようとしました」は、設定ではなく接続タイミング（ラグ）の問題だった

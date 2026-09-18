# Yuki System

Yuki System は、スマートデバイス、音声アシスタント、IoT 連携のためのオープンソースのモジュール型エコシステムです。
複数のデバイス（PC、モバイルデバイス、スマートスピーカー、IoT デバイス）を、統一されたプロトコルを使って中央サーバーから制御できます。

> この README は概要のみです。アーキテクチャ、プロトコルの詳細なリファレンス、セキュリティモデルについては [DOCUMENTATION.md](DOCUMENTATION.md)（[русский](DOCUMENTATION.ru.md)、[日本語](DOCUMENTATION.ja.md)）を参照してください。このファイルは [русский](readme.ru.md) と [日本語](readme.ja.md) でも利用できます。

---

## 概要

Yuki System は、それぞれ特定の役割を持つ複数のモジュールから構成されています。

```text
yuki-system（メタ / このリポジトリ）
        │
        ▼
yuki-core（サーバー / 頭脳）
        │
        ├── yuki-protocol（通信プロトコル & SDK）
        │
        ├── yuki-webui（Web インターフェース）
        │
        └── デバイス（すべてのクライアントは yuki-protocol 経由で接続）
              ├── yuki-device-pc（Windows）
              ├── yuki-device-pc-linux（Linux）
              ├── yuki-device-android
              ├── yuki-humidifier（ESP32 スマート加湿器）
              ├── yuki-device-frame（FrameOS）
              └── yuki-speaker（ESP32 スマートスピーカー）
```
## リポジトリ構成

このリポジトリは**メタリポジトリ**です。以下を提供します。

- Yuki System エコシステムのドキュメント  
- アーキテクチャ図とコンポーネントの説明  
- ロードマップと開発ガイド  
- 各リポジトリへのリンク  

> 実際のコードは、各モジュールごとに分かれた個別のリポジトリにあります。

---

## コンポーネント

| モジュール | 役割 | 説明 |
|--------|------|-------------|
| `yuki-core` | サーバー / 頭脳 | コマンドのルーティング、AI 処理、デバイス管理を担当 |
| `yuki-protocol` | プロトコル & SDK | メッセージ形式、コマンドタイプ、デバイス接続用 SDK を定義 |
| `yuki-webui` | Web インターフェース | デバイスの監視、コマンドの送信、自動化の管理が可能 |
| `yuki-speaker` | ESP32 スマートスピーカー | 音声入出力を提供し、サーバーに接続 |
| `yuki-device-pc` | Windows クライアント | サーバーからのコマンドを実行 |
| `yuki-device-pc-linux` | Linux クライアント | `yuki-device-pc` と同じ機能セット、Linux デスクトップ向け |
| `yuki-device-android` | Android クライアント | Android デバイス上でコマンドと通知を実行 |
| `yuki-humidifier` | ESP32 スマート加湿器 | サーバー経由で制御可能な Wi-Fi 加湿器デバイス |

---

## はじめに

`yuki-core` を起動し、次に `yuki-webui` を起動してから、必要なデバイスクライアントを設定してください - 正確なインストール/実行手順は各モジュール自身の README にあります。全体の手順とセキュリティモデルについては [DOCUMENTATION.md](DOCUMENTATION.md) を参照してください。

---

## ロードマップ

- [x] Yuki Core サーバーの MVP を完成  
- [x] Yuki Protocol 仕様（`yuki/1.0`）を確定  
- [x] 基本的なデバイス管理を備えた Yuki WebUI を開発  
- [x] 最初のデバイスを接続: `yuki-device-pc` と `yuki-device-android`  
- [x] デバイスエコシステムを拡張（`yuki-device-pc-linux`、`yuki-humidifier`）  
- [ ] AI ベースの音声アシスタント機能を統合  
- [ ] `yuki-device-frame`（FrameOS）  

---

---

## ライセンス

このリポジトリおよびそのドキュメントは **MIT License** の下でライセンスされています。
詳細は [LICENSE](./LICENSE) を参照してください。

---

## 補足

- Yuki System はモジュール構成です。各コンポーネントは独自のリポジトリを持ちます。  
- このメタリポジトリは、概要と一元的なドキュメントを提供することを目的としています。  

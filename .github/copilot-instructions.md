# GitHub Copilot Instructions
---

## 🔥 新規プロジェクト：モジュール編の全面改訂（2026年1月22日開始）

**目的**: モジュール編（Chapter 06-11）をわかりやすく全面改訂

### 改訂方針

**3つの必須要件**:
1. **実際のファイルを全て解説**: コード例は実際のファイルから取得、省略なし
2. **初心者が理解できる説明**: 専門用語は噛み砕いて、具体例を豊富に
3. **ストーリー仕立てのカジュアルな口調**: Chapter 01の口調を踏襲

**作業手順**:
1. 既存のChapterを読んで問題点を洗い出す
2. 実際のファイル（modules/*, main.*.tf）を完全に読む
3. `XX_タイトルnew.md`として新しいファイルを作成
4. レビュー後、既存ファイルと置き換え

### Chapter 06-11 改訂計画

#### Chapter 06: 設定テンプレート（modules/config-templating/）
- Part 1: テンプレートエンジンの仕組み
- Part 2: data.tfの解説（regionsモジュール、client_config）
- Part 3: locals.config.tfの解説（テンプレート置換ロジック）
- Part 4: outputs.tfの解説（処理結果の出力）
- Part 5: 実践：テンプレート変数の追加方法

#### Chapter 07: リソースグループ（main.resource.groups.tf）
- Part 1: リソースグループとは
- Part 2: main.resource.groups.tfの完全解説
- Part 3: for_eachの仕組みとフィルタリング
- Part 4: 実践：リソースグループの追加方法

#### Chapter 08: 管理グループとポリシー（main.management.tf）
- Part 1: 管理グループの階層構造
- Part 2: main.management.tfの完全解説
- Part 3: lib/フォルダのYAML定義
- Part 4: 実践：カスタムポリシーの追加方法

#### Chapter 09: 管理リソース（main.management.tf）
- Part 1: Log Analyticsとは
- Part 2: Automation Accountとは
- Part 3: management_resourcesモジュールの解説
- Part 4: 実践：Data Collection Rulesのカスタマイズ

#### Chapter 10: Hub-and-Spoke（main.connectivity.hub.and.spoke.virtual.network.tf）
- Part 1: Hub-and-Spokeアーキテクチャとは
- Part 2: main.connectivity.hub.and.spoke.virtual.network.tfの完全解説
- Part 3: Firewall、Bastion、VPN Gatewayの設定
- Part 4: 実践：Spoke VNetの追加方法

#### Chapter 11: Virtual WAN（main.connectivity.virtual.wan.tf）
- Part 1: Virtual WANとは（Hub-and-Spokeとの違い）
- Part 2: main.connectivity.virtual.wan.tfの完全解説
- Part 3: Virtual HubとVPN/ExpressRouteの設定
- Part 4: 実践：Virtual WANへの移行方法

### 進捗管理（モジュール編）

- ✅ Chapter 06: 設定テンプレート（完了）
- ⏳ Chapter 07: リソースグループ
- ⏳ Chapter 08: 管理グループとポリシー
- ⏳ Chapter 09: 管理リソース
- ⏳ Chapter 10: Hub-and-Spoke
- ⏳ Chapter 11: Virtual WAN

### Chapter 06校正メモ

以下の校正ルールを他のChapterでも適用すること：

**削除すべき表現：**
- 「完全版」「すべて解説」「省略なし」などの過剰なアピール表現
- コードブロックタイトルの「（完全版）」「（完全版：XX行）」
- 「ファイル別行数まとめ」のような実用性の低い表

**フォーマット統一：**
- リスト項目（`-`で始まる）の前には必ず空行を入れる
- 特に「**XXX:**」の後にリストが続く場合は必須

**書き方の方針：**
- 行数や完全版などのアピールしない
- 具体例と図解を充実させる
- 初心者が理解できる噛み砕いた説明

---


**前提サイト**: https://azure.github.io/Azure-Landing-Zones/bootstrap/


**デザイン装飾のルール**:
- ✅ 使用する要素: タブ (`=== "タイトル"`)、アドモニション (`!!! warning/info/success/tip`)、コードタイトル (```lang title="xxx")
- ❌ 使用しない: カード型レイアウト、アイコン記法 (`:material-xxx:`)


## ドキュメント作成時の絶対ルール

### 🚨 口調ルール（絶対に守ること）🚨

ストーリー仕立てでカジュアルな口調で書くこと。AIっぽくない口調で書くこと。

**Chapter 01の口調が参考。**

---


## その他のルール

- 技術的な正確さを保つ
- 複雑な概念は噛み砕いて説明
- 例え話を使って理解しやすくする
- 作業後にこのファイルの更新箇所があれば必ず更新すること

# GitHub Copilot Instructions
---
**前提サイト**: https://azure.github.io/Azure-Landing-Zones/bootstrap/

## ドキュメント作成時の絶対ルール

### 🚨 口調ルール（絶対に守ること）🚨

ストーリー仕立てでカジュアルな口調で書くこと。AIっぽくない口調で書くこと。（敬語）

---

教科書全体のフォーマットを必ず守ること。
変更する場合は、その章より前の章を必ず確認すること。


## その他のルール

- 複雑な概念は噛み砕いて説明
- 例え話を使って理解しやすくする
- 作業後にこのファイルの更新するべき約束があれば必ず更新すること
- コミットするときは、mainブランチが保護されているので、プルリクエスト、マージまでやること。

## 現在のプロジェクト

05章以降は、マニアックな人向けにazure randing zoneのterraformのコードを全て解説する教科書になっています。

今の教科書は若干わかりずらいです。

**全てのコード**（モジュールを使っているところはそのモジュールまでたどって解説する）を解説する教科書として、わかりやすいように構成して、つくりかえてみてください。

どれだけ長くなっていもいいです。よろしくお願いします。（ファイルは既存ファイルは残して新ファイルとして作成してください。）（形式は既存の教科書から逸脱しないようにしてください。全体で一つの教科書です。）適宜理解しやすいようにAzureアイコンなどを使ったdraw.ioのsvg図も入れてください。

1章ごとに出力を止め、進捗状況をこのファイルの進捗欄に必ず記載してください。

最高の教科書を作りたいです。

## 進捗

### 2026年2月2日
- ✅ **NEW_05_コード完全ガイド_概要編.md** 作成完了
  - 全体マップ図（code-complete-guide-overview.drawio.svg）作成
  - データの流れを具体例で解説
  - モジュール3層構造の説明
  - 学習ルート案内
- ✅ **NEW_06_terraform.tf_プロバイダー設定.md** 作成完了
  - required_version, required_providers解説
  - backend設定（Azure Blob Storage）
  - プロバイダーエイリアス（management, connectivity）解説
- ✅ **NEW_07_variables.tf_全変数定義.md** 作成完了
  - 型の読み方（object, optional, map等）
  - starter_locations, subscription_ids解説
  - hub_virtual_networks（1200行）の構造解説
- ✅ **NEW_08_locals.tf_ローカル変数.md** 作成完了
  - 定数定義（local.const）
  - フラグ計算（connectivity_enabled等）
  - 依存関係の構築（merge）
- 🔄 NEW_09 作成中...
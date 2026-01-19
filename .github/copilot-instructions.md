# GitHub Copilot Instructions

## 現在の状況

**プロジェクト**: GitHub Pages教科書サイトのデザイン改善

- 教科書作成: Chapter 00〜17（全18章）完成済み
- 口調統一: 全章「カジュアル敬語調」で統一完了
- サイト構築: MkDocs Material でGitHub Pages化完了
- **新タスク**: 全18章にMaterial for MkDocsのデザイン要素を追加（おしゃれなデザイン化）

**デザイン装飾の進捗**:
- ✅ Chapter 00 (00_はじめに.md): 完全装飾完了（PR #23, #24）
- ✅ Chapter 01 (01_基礎知識.md): 完全装飾完了（PR #25, #26, #27）
- ✅ Chapter 02 (02_プロジェクト構造.md): 完全装飾完了（PR #28, #29）
  - ファイル構成、データフロー、設定ファイル: タイトル化完了
  - mainファイル群、outputs、モジュール構造: タイトル化完了
  - lib/ポリシー定義、CI/CD、まとめ: タイトル化完了
- ✅ Chapter 03 (03_設定ファイル完全解説.md): 完全装飾完了（PR #30, #31）
  - Part 1-7: テンプレート変数、リージョン設定、リソース名、有効/無効設定
  - Part 8-9: 管理グループ、ポリシー、ネットワーク設定、練習問題
- ✅ Chapter 04 (04_変数定義.md): 完全装飾完了（PR #32）
  - 型定義、バリデーション、デフォルト値の例
  - 全変数定義（starter_locations、subscription_ids等）
  - 全型の例（基本型、コレクション型、構造型）
  - 全バリデーションパターン、実践例
- ✅ Chapter 05 (05_ローカル変数.md): 完全装飾完了（PR #33）
  - locals定義、variablesとの違い、具体例
  - 定数定義、接続タイプ判定、依存関係
  - 全5パターン、実践例3つ、デバッグ技
- ✅ Chapter 06 (06_設定テンプレート.md): 完全装飾完了（PR #34）
  - config-templatingモジュール構造、入力変数、データソース
  - テンプレート処理4段階（組込み変数、カスタム名、リソースID、最終統合）
  - JSON変換トリック、実践例3つ、デバッグ技術
- ✅ Chapter 07 (07_リソースグループ.md): 完全装飾完了（PR #36）
  - リソースグループ基本構造、削除ルール
  - tfvars設定（テンプレート、タグ、enabled）
  - 変数定義（map(object)、optional）
  - モジュール呼び出し（for_each、try、providers）
  - 実践例3つ、デバッグ技術、エラー対処、設計パターン
- ✅ Chapter 08 (08_管理グループとポリシー.md): 完全装飾完了（PR #38）
  - 管理グループ階層構造（完全版・簡易版）
  - アーキテクチャ定義、アーキタイプ定義
  - tfvars設定（architecture、policy_default_values、subscription_placement）
  - モジュール呼び出し、ポリシー種類4つ
  - ポリシー継承、実践例2つ、デバッグ技術、設計パターン4種類
- ✅ Chapter 09 (09_管理リソース.md): 完全装飾完了（PR #40）
  - 管理リソースの役割、3つのシナリオ
  - tfvars設定（enabled、location、workspace_name、DCR、Managed Identity）
  - モジュール呼び出し（count、providers、coalesce）
  - Log Analytics詳細（KQLクエリ、アラート、設定項目）
  - DCR詳細3種類（change_tracking、defender_sql、vm_insights）
  - Managed Identity（System/User-assigned、使用フロー、ポリシー連携）
  - 実践例3つ、デバッグ技術、エラー対処、コスト最適化
- ✅ Chapter 10 (10_Hub-and-Spoke.md): 完全装飾完了（PR #42）
  - Hub-and-Spokeアーキテクチャ、Virtual WANとの比較
  - tfvars設定（connectivity_type、DDoS Protection、hub_virtual_networks）
  - Hub VNet構成要素6つ（VNet、Firewall、VPN GW、ER GW、Bastion、DNS）
  - マルチリージョン構成（primary/secondary）
  - ルーティング設定（UDR）、コスト削減版、デバッグ技術、エラー対処4つ
- ✅ Chapter 11 (11_Virtual_WAN.md): 完全装飾完了（PR #44）
  - Virtual WANのグローバルアーキテクチャ、Hub-and-Spokeとの比較
  - tfvars設定（connectivity_type、virtual_wan_settings、virtual_hubs）
  - Virtual Hub設定（address_prefix、routing_preference、スケールユニット）
  - Firewall（Virtual WAN版）、VPN Gateway、VNet接続、ルーティング
  - Routing Intent、Sidecar VNet、コスト削減版、デバッグ技術、エラー対処4つ
- ✅ Chapter 12 (12_GitHub_Actions.md): 完全装飾完了（PR #46）
  - GitHub ActionsによるCI/CDパイプライン
  - ワークフロー定義（ci.yaml、cd.yaml）、トリガー設定
  - 再利用可能ワークフロー（ci-template、cd-template）
  - ステップ詳細（Checkout、Setup、Azure Login、Init、Validate、Plan、Comment）
  - OIDC設定（Azure ADアプリ、Service Principal、Federated Credential）
  - 実践手順、デバッグ技術、エラー対処4つ、ベストプラクティス4つ
- ✅ Chapter 13 (13_デプロイ手順.md): 完全装飾完了（PR #48）
  - ゼロから環境を作るデプロイ手順（全6フェーズ）
  - Phase 1-2: Azure準備、GitHub準備（Service Principal、Storage、Secrets）
  - Phase 3-4: ローカルデプロイ、GitHub Actionsデプロイ
  - Phase 5-6: 本番デプロイ、動作確認（Management Group、Policy、Firewall等）
  - トラブルシューティング4つ、デプロイ後チェックリスト
- ✅ Chapter 14 (14_トラブルシューティング.md): 完全装飾完了（PR #50）
  - 認証エラー（OIDC、Azure login、Service Principal）
  - Terraform State問題（Lock、permissions、drift）
  - リソース作成エラー（Zones、naming、Firewall timeout）
  - GitHub Actionsエラー（permissions、secrets、concurrency）
  - モジュールエラー（version、input）
  - デバッグテクニック5つ（TF_LOG、graph、show、console、Azure CLI）
  - FAQ 5項目（version upgrade、state management、import、cost reduction）
- ✅ Chapter 15 (15_カスタマイズ.md): 完全装飾完了（PR #52）
  - lib構造とファイルの役割（architecture_definitions、archetype_definitions）
  - Management Group追加・削除・階層変更
  - ポリシーカスタマイズ（追加・削除・パラメータ変更）
  - 命名規則カスタマイズ（環境prefix、リージョン略語）
  - ネットワークカスタマイズ（サブネット、Firewallルール、Spoke VNet）
  - サブスクリプション配置、タグ、ロギング設定
  - 実践例：3環境構成（本番・ステージング・開発）
- ✅ Chapter 16 (16_ベストプラクティス.md): 完全装飾完了（PR #54）
  - セキュリティ（最小権限、MFA、Private Endpoint、アクセスレビュー）
  - コスト管理（予算、タグ、自動停止、Reserved Instances、アラート）
  - Infrastructure as Code（全コード化、State管理、モジュール化、検証、Output）
  - チーム協業（ブランチ戦略、PRルール、コードレビュー、ペアプログラミング）
  - ドキュメント（README、ADR、ランブック、トラブルシューティングガイド）
  - 監視とアラート（Log Analytics、アラートルール、Action Group）
  - バックアップとDR（Stateバックアップ、マルチリージョン、DR計画）
  - 変更管理（変更ウィンドウ、承認プロセス、ロールバック計画）
  - 継続的改善（定期レビュー、バージョンアップ、技術負債管理）
- ✅ Chapter 17 (17_FAQ.md): 完全装飾完了（PR #56）
  - 基本編（ALZとは、必要性、デプロイ時間、コスト、ツール選択）
  - デプロイ編（既存環境、複数環境管理、失敗時対処、ローカル実行）
  - ネットワーク編（Hub vs vWAN、移行、MG設計、リージョン数、DRサイト、Spoke数、Firewall vs NSG）
  - 運用編（バージョンアップ、手動変更、State破損、チーム拡大）
  - カスタマイズ編（命名規則、独自ポリシー、MG構造変更）
  - トラブル編（apply遅延、GitHub Actions、zones エラー）
  - その他（マルチクラウド、サポート、ドキュメント、学習リソース、本番チェックリスト）

## 🎊 完了：全18章の装飾が完成しました！ 🎊

**プロジェクト完了日**: 2026年1月19日

**完成した教科書サイト**: https://shuhei0720org01.github.io/alz-mgmt/

**成果**:
- 全18章（Chapter 00-17）にMaterial for MkDocsデザイン要素を追加
- 総コードタイトル数: 約340個以上（bash、hcl、yaml、json、kql、cron等）
- 総PR数: 36個（18章×2PR = 章装飾PR + instruction更新PR）
- 文章は一言一句変更なし（マークアップのみ追加）
- 全ビルド成功

**追加作業 - Markdown書式修正**:
- 日付: 2026年1月19日
- PR #58: markdown formatting修正（リストと表の空行追加）
- 修正箇所: 6箇所（Chapter 03×3、06×1、08×1、16×1）
- 問題: ヘッダー（`**xxx**:`）の直後にリスト/表を書くとレンダリング不具合
- 解決: ヘッダーとリスト/表の間に空行を追加
- 文章変更なし（空行追加のみ）
- ビルドテスト成功

**教科書の内容**:
- 基礎編（00-02）: Azure Landing Zonesの全体像、Terraform基礎、プロジェクト構造
- 設定編（03-06）: tfvars完全解説、変数定義、ローカル変数、テンプレート処理
- リソース作成編（07-11）: RG、管理グループ、ポリシー、管理リソース、Hub-and-Spoke、Virtual WAN
- 運用編（12-14）: GitHub Actions、デプロイ手順、トラブルシューティング
- 応用編（15-17）: カスタマイズ、ベストプラクティス、FAQ

**デザイン装飾のルール**:
- ✅ 使用する要素: タブ (`=== "タイトル"`)、アドモニション (`!!! warning/info/success/tip`)、コードタイトル (```lang title="xxx")
- ❌ 使用しない: カード型レイアウト、アイコン記法 (`:material-xxx:`)

### 🚨🚨🚨 装飾時の絶対厳守ルール（違反したら即終了） 🚨🚨🚨

**以下のルールを1つでも破ったら作業中止。絶対に守ること。**

1. **文章は一言一句変更しない**
   - 既存の文章を書き換えない
   - 文章を追加しない
   - 文章を削除しない
   - 文章を移動しない
   - 文章の順序を変えない

2. **追加するのはマークアップ記号だけ**
   - コードブロックに `title="説明"` を追加するだけ
   - 既存のコードブロックを ` ``` ` から ` ```language title="xxx"` に変更するだけ
   - 既存のコード例を `===` でタブ化する際も、コード内容は絶対変更しない
   - 既存のリストや段落を `!!!` でアドモニション化する際も、文章内容は絶対変更しない

3. **やってはいけないこと（厳禁）**
   - ❌ 文章をアドモニションの中に移動する
   - ❌ アドモニションのタイトルに新しい文章を書く
   - ❌ 既存の文章を「わかりやすく」書き直す
   - ❌ 説明を追加する
   - ❌ 表のセルの中身を変更する
   - ❌ リストの項目を変更する
   - ❌ コード例の中身を変更する

4. **正しい装飾の例**
   ```markdown
   <!-- 元の文章 -->
   これは例です：
   ```hcl
   code = "example"
   ```
   
   <!-- 正しい装飾（コードタイトルだけ追加） -->
   これは例です：
   ```hcl title="設定例"
   code = "example"
   ```
   ```

5. **間違った装飾の例（絶対NG）**
   ```markdown
   <!-- 元の文章 -->
   これは例です：
   ```hcl
   code = "example"
   ```
   
   <!-- ❌ NG：文章を変更 -->
   !!! info "例の説明"
       これは設定の例です：
       ```hcl title="設定例"
       code = "example"
       ```
   ```

**装飾前に必ず確認すること**:
- [ ] 元のファイルの文章を一言一句そのまま残しているか
- [ ] 追加したのはマークアップ記号（` ``` title=`、`===`、`!!!`）だけか
- [ ] git diff で文章の変更がないことを確認したか

**作業方針**:
- 1章ずつ完全に装飾してからコミット
- 装飾完了後、必ずinstructionファイルを更新
- ビルド確認してからPR作成・マージ

---

## ドキュメント作成時の絶対ルール

### 🚨 口調ルール（絶対に守ること）🚨

**このプロジェクトのドキュメントは「カジュアル敬語調」で書く。Chapter 01の口調が正解。**

「です・ます」を使うけど、友達に話しかけるような親しみやすさもある口調。
AIっぽい硬い説明文ではなく、教えるのが好きな先輩が後輩に説明する感じ。

#### ✅ 使うべき表現（Chapter 01スタイル）
- 「〜です」「〜ます」「〜ました」（基本の敬語）
- 「〜してください」「〜しましょう」（柔らかい依頼・提案）
- 「〜ですよね」「〜ですか？」（親しみのある問いかけ）
- 「見てみましょう」「試してみてください」（一緒に進む感じ）
- 「わかりますか？」「理解できましたか？」（確認）
- 「簡単ですよね」「便利ですよね」（共感）

#### ❌ 避けるべき表現
- 「〜になります」（コンビニ敬語）
- 「〜いたします」「〜させていただきます」（過剰敬語）
- 「ご確認ください」「ご覧ください」（硬すぎる）
- 「〜であろう」「〜である」（論文調）
- 「〜だね」「〜だよ」「〜じゃん」（タメ口すぎる）
- 「〜しちゃう」「〜してる」（カジュアルすぎる）

### 具体例

#### ✅ 良い例（Chapter 01スタイル）
```markdown
変数を定義する際は、以下のように記述します。
このコードは、文字列型の変数を定義しています。
型を指定することで、不正な値の代入を防ぐことができます。

わかりますか？では次に進みましょう。
```

#### ❌ ダメな例1（硬すぎる）
```markdown
変数を定義する際は、以下のように記述することになります。
当該コードにおきましては、文字列型の変数を定義しております。
型を指定することで、不正な値の代入を防止することが可能となります。
```

#### ❌ ダメな例2（カジュアルすぎる）
```markdown
変数ってこう書くんだ。
これ、文字列型の変数だね。
型を指定しとけば、変な値が入るのを防げる。

わかる？じゃあ次いこう。
```

### よく使うフレーズ
- 「見てみましょう」「確認してみましょう」
- 「〜ですよね」「〜ですね」
- 「わかりますか？」「理解できましたか？」
- 「試してみてください」「実行してみましょう」
- 「簡単ですよね」「便利ですよね」
- 「次に進みましょう」「詳しく見ていきます」

### 参考ファイル（この口調が正解）
- `docs_new/01_基礎知識.md` ← **これが正解の口調！**
- `docs_new/02_プロジェクト構造.md`

**Chapter 01の口調を真似て書く。**

---

## チェックリスト

ドキュメント作成時は必ず確認：

- [ ] 「です・ます」を適切に使っている
- [ ] 「〜になります」を使っていない（コンビニ敬語NG）
- [ ] 硬すぎる敬語（ご確認くださいetc.）になっていない
- [ ] カジュアルすぎる（〜だね、〜だよ）になっていない
- [ ] Chapter 01のような親しみやすい敬語調になっている
- [ ] 読者に語りかける口調（〜ですよね、わかりますか？）を使っている

---

## その他のルール

- 技術的な正確さを保つ
- コード例は実際に動くものを記載
- 複雑な概念は噛み砕いて説明
- 例え話を使って理解しやすくする
- 「わかる？」「見て」など読者に語りかける
- 作業後にこのファイルの更新箇所があれば必ず更新すること

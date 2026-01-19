# 17. FAQ - よくある質問

## このChapterでやること

Azure Landing Zonesに関するよくある質問をまとめたよ。

**困ったときはここを見てください！**

---

## 基本編

### Q1: Azure Landing Zonesって何？

**A**: Azureを企業で使うための「土台」です。

```
個人で使う：

- 適当にリソース作ってOK

企業で使う：

- セキュリティ
- ガバナンス
- コスト管理
- 複数チーム
→ ちゃんとした構造が必要
  ↓
それがAzure Landing Zones
```

**構成要素**：

- Management Group（組織階層）
- Policy（ルール）
- Network（Hub-and-Spoke）
- Logging（監視）

### Q2: 必ずAzure Landing Zonesを使わないとダメ？

**A**: いや、絶対じゃない。

**使った方がいい場合**：
```
✓ 複数のチーム・プロジェクト
✓ セキュリティ要件が厳しい
✓ ガバナンスが必要
✓ 長期運用
✓ エンタープライズ
```

**使わなくてもいい場合**：
```
✓ 個人プロジェクト
✓ 小規模（VM数台）
✓ 短期間の検証
✓ PoC（概念実証）
```

**判断基準**：
```
Subscription 1個、VM 10台以下
  ↓
普通にAzureリソース作るだけでOK

Subscription 3個以上、複数部署
  ↓
Azure Landing Zones使おう
```

### Q3: デプロイにどのくらい時間かかる？

**A**: 初回は2〜3時間。

**内訳**：
```
準備（Azure、GitHub設定）: 1時間
初回デプロイ: 1時間
動作確認: 30分
```

**2回目以降**：
```
変更内容による
- リソースグループ追加: 5分
- Firewall作成: 30〜60分
- Management Group追加: 10分
```

### Q4: いくらかかる？

**A**: 構成による。

**最小構成（開発環境）**：
```
VNet: 無料
Log Analytics: 5GB/月まで無料
Management Group: 無料
Policy: 無料
----------
合計: ほぼ0円/月
```

**標準構成（本番環境）**：
```
Firewall Standard: 17万円/月
VPN Gateway: 4万円/月
Bastion: 2.7万円/月
Log Analytics: 1万円/月
----------
合計: 約25万円/月
```

**フル構成（大規模）**：
```
Firewall Premium: 25万円/月
DDoS Protection: 40万円/月
ExpressRoute: 10万円/月
VPN Gateway: 4万円/月
Bastion: 2.7万円/月
----------
合計: 約80万円/月
```

**コスト削減のコツ**：

- 開発環境はFirewall無効化
- DDoS Protectionは本番のみ
- VM自動停止
- Reserved Instances活用

### Q5: Terraformじゃないとダメ？

**A**: いや、他の方法もある。

**選択肢**：
1. **Terraform**（このプロジェクト）
   - メリット：柔軟、マルチクラウド、コミュニティ大きい
   - デメリット：学習コスト

2. **Bicep**（Azure公式）
   - メリット：Azure専用、シンプル
   - デメリット：Azure限定

3. **Azure Portal**（GUI）
   - メリット：簡単
   - デメリット：再現性ない、自動化できない

4. **Azure CLI/PowerShell**
   - メリット：スクリプト化できる
   - デメリット：複雑になる

**推奨**：
```
小規模 → Azure Portal（楽）
中規模 → Bicep（バランス良い）
大規模 → Terraform（柔軟）
```

---

## デプロイ編

### Q6: 既存のAzure環境にデプロイできる？

**A**: できるけど注意が必要。

**問題点**：
```
既存のManagement Groupがある
  ↓
Terraformの管理下に入れる必要
  ↓
terraform import
```

**手順**：
```bash title="既存Management Groupのimport"
# 既存Management Groupをimport
terraform import 'module.management_groups[0].azurerm_management_group.level_1["existing-mg"]' /providers/Microsoft.Management/managementGroups/existing-mg

# Plan確認
terraform plan

# 差分があれば調整
```

**推奨**：
```
新しいAzure環境で始める
  ↓
既存環境は移行計画を立てる
```

### Q7: 複数環境（本番・開発）をどう管理する？

**A**: 方法は3つ。

#### 方法1: 同じリポジトリ、別State（推奨）

```
alz-mgmt/
├── environments/
│   ├── production/
│   │   └── terraform.tfvars.json
│   └── development/
│       └── terraform.tfvars.json
├── main.tf
└── ...
```

**デプロイ**：
```bash title="環境別のデプロイ"
# 本番
cd environments/production
terraform init -backend-config="key=prod.tfstate"
terraform apply -var-file=terraform.tfvars.json

# 開発
cd environments/development
terraform init -backend-config="key=dev.tfstate"
terraform apply -var-file=terraform.tfvars.json
```

#### 方法2: Terraform Workspace

```bash title="Workspaceでの環境分離"
# Workspace作成
terraform workspace new production
terraform workspace new development

# 切り替え
terraform workspace select production
terraform apply

terraform workspace select development
terraform apply
```

#### 方法3: 完全に別リポジトリ

```
alz-mgmt-production/
alz-mgmt-development/
```

**推奨**：方法1（別State）

### Q8: デプロイ中に失敗したらどうする？

**A**: まず落ち着いて確認。

**手順**：
```
1. エラーメッセージ確認
2. Chapter 14（トラブルシューティング）を見る
3. Azureの状態確認
4. terraform state確認
5. 再度terraform apply
```

**よくある失敗**：
```
- Firewall作成タイムアウト
  → もう一度terraform apply

- State Lock
  → terraform force-unlock

- 権限エラー
  → Service Principalの権限確認
```

**ロールバック**：
```bash title="変更のロールバック"
# 最後のCommitに戻す
git revert HEAD
git push

# またはDestroyして再作成
terraform destroy
terraform apply
```

### Q9: GitHub Actionsなしでデプロイできる？

**A**: できる。ローカルで実行すればOK。

**ローカル実行**：
```bash title="ローカルでのTerraform実行"
# 環境変数設定
export ARM_STORAGE_ACCOUNT_NAME="..."
export ARM_CONTAINER_NAME="tfstate"
export ARM_KEY="alz-mgmt.tfstate"
export ARM_RESOURCE_GROUP_NAME="rg-terraform-state"

# Azureログイン
az login

# デプロイ
terraform init
terraform plan
terraform apply
```

**メリット**：
```
- シンプル
- 設定が少ない
```

**デメリット**：
```
- 自動化されない
- チーム協業しにくい
- 履歴が残りにくい
```

**推奨**：
```
最初はローカル
  ↓
慣れたらGitHub Actions
```

---

## ネットワーク編

### Q10: Hub-and-SpokeとVirtual WAN、どっち選ぶ？

**A**: 規模と要件で決める。

**Hub-and-Spoke**：
```
✓ 小〜中規模（VNet 10個以下）
✓ 単一〜数リージョン
✓ コスト重視
✓ 細かい制御が必要
✓ ネットワークの仕組みを理解したい

例：

- 国内のみ展開
- Spoke VNet 5個
- VPN接続 1〜2本
- コスト: 約¥50万/月
```

**Virtual WAN**：
```
✓ 大規模（VNet 10個以上）
✓ 多数のリージョン（5+）
✓ 多数のVPN接続（10+）
✓ グローバル展開
✓ 管理の簡素化重視
✓ Microsoftに任せたい

例：

- 世界10リージョン
- Spoke VNet 50個
- 拠点VPN接続 100本
- コスト: 約¥150万/月
```

**判断基準**：
```
VNet 10個以下 → Hub-and-Spoke
VNet 10個以上 → Virtual WAN

リージョン 3個以下 → Hub-and-Spoke
リージョン 4個以上 → Virtual WAN

VPN拠点 10個以下 → Hub-and-Spoke
VPN拠点 10個以上 → Virtual WAN

初めてのLanding Zones → Hub-and-Spoke（学習しやすい）
```

**詳細**：[Chapter 10](./10_Hub-and-Spoke.md)と[Chapter 11](./11_Virtual_WAN.md)で詳しく解説しています。

### Q10.5: Hub-and-SpokeからVirtual WANに移行できる？

**A**: できるけど、結構大変。

**移行手順（概要）**：
```
1. Virtual WAN作成
2. Virtual Hub作成（既存Hubと別リージョン推奨）
3. Spoke VNetを新Hubに接続
4. DNSとルーティングの切り替え
5. 既存Hubの停止
6. 動作確認
7. 既存Hub削除
```

**ダウンタイム**：
```
計画的に切り替えれば最小限（数分〜数時間）
```

**コスト影響**：
```
移行期間中：両方のコストがかかる
移行後：Virtual WANの方が高い（約2〜3倍）
```

**推奨**：
```
最初からVirtual WANにするか、Hub-and-Spokeで十分か慎重に検討。
移行は可能だけど、最初の選択が重要。
```

### Q10.6: 管理グループの階層はどう設計する？

**A**: 組織構造に合わせる。

**基本パターン（このプロジェクト）**：
```
Root
└── Azure Landing Zones
    ├── Platform（基盤チーム管理）
    │   ├── Management（監視）
    │   ├── Connectivity（ネットワーク）
    │   └── Identity（認証）
    ├── Landing Zones（アプリチーム管理）
    │   ├── Corp（社内向け）
    │   └── Online（インターネット公開）
    ├── Sandbox（実験用）
    └── Decommissioned（廃止予定）
```

**大企業パターン（部門別）**：
```
Root
└── Azure Landing Zones
    ├── Platform
    ├── Division A（営業部門）
    │   ├── Production
    │   └── Development
    ├── Division B（開発部門）
    │   ├── Production
    │   └── Development
    └── Shared Services（共有サービス）
```

**グローバル企業パターン（地域別）**：
```
Root
└── Azure Landing Zones
    ├── Platform
    ├── Asia Pacific
    │   ├── Japan
    │   └── Singapore
    ├── Europe
    │   ├── UK
    │   └── Germany
    └── Americas
        ├── US East
        └── US West
```

**設計のポイント**：
1. **階層は3〜4層まで**（深すぎると管理が複雑）
2. **責任範囲で分ける**（誰が管理するか）
3. **ポリシーの継承を考える**（親の設定が子に適用される）
4. **将来の拡張を考慮**（後から追加しやすく）

**詳細**：[Chapter 08](./08_管理グループとポリシー.md)で詳しく解説しています。

### Q10.7: リージョンは何個必要？

**A**: 要件による。

**最小構成（1リージョン）**：
```
✓ 災害対策不要（割り切り）
✓ コスト最重視
✓ 検証環境

例：

- Japan East のみ
- コスト: 約¥50万/月
```

**推奨構成（2リージョン）**：
```
✓ 災害対策あり（基本）
✓ バランス重視
✓ 本番環境

例：

- Japan East（プライマリ）
- Japan West（セカンダリ）
- コスト: 約¥100万/月
```

**グローバル構成（3+リージョン）**：
```
✓ グローバル展開
✓ 低遅延（ユーザーに近い）
✓ 規制対応（データを国外に出せない）

例：

- Japan East
- Southeast Asia
- East US
- West Europe
- コスト: 約¥300万/月
```

**判断基準**：
```
RPO/RTO要件なし → 1リージョン
RPO/RTO要件あり → 2リージョン
海外ユーザーあり → 3+リージョン
```

**注意点**：

- リージョン間のデータ転送は課金される
- 管理が複雑になる（リージョン数 × 設定数）
- このプロジェクトは2リージョン構成

### Q10.8: DRサイト（災害対策サイト）は必須？

**A**: ビジネス要件による。

**DR不要なケース**：
```
- 停止しても大きな影響なし
- 復旧に数日かけてOK
- コスト最優先
- 検証環境

→ 1リージョンで十分
```

**DR必要なケース**：
```
- 止まったら困る（ECサイト、基幹システム）
- SLA（可用性保証）が必要
- 規制要件
- 本番環境

→ 2リージョン構成
```

**DRの種類**：

| タイプ | RPO | RTO | コスト | 説明 |
|--------|-----|-----|--------|------|
| **Cold Standby** | 数時間 | 数時間〜1日 | 低 | 必要時にリソース作成 |
| Warm Standby | 数分〜数時間 | 数時間 | 中 | 最小限のリソースを常時起動 |
| **Hot Standby** | 数秒〜数分 | 数分 | 高 | 全リソースを常時起動（Active-Passive） |
| **Active-Active** | ほぼ0 | ほぼ0 | 最高 | 両リージョンで常時稼働 |

**このプロジェクトの構成**：

- Warm Standby相当
- ネットワーク基盤は両リージョンに構築
- アプリは必要に応じて追加

**詳細**：[Chapter 00](./00_はじめに.md)で2拠点構成について説明しています。

---

### Q11: Spoke VNetはいくつ作れる？

**A**: 制限はあるけど、実用上は問題ない。

**制限**：
```
Hub-and-Spoke：

- VNetピアリング: 最大500個/VNet
  ↓
Spoke VNet 500個まで

Virtual WAN：

- VNet接続: 制限なし（実質無制限）
```

**実用上**：
```
ほとんどの組織は10〜50個で十分
```

### Q12: オンプレミスとの接続は必須？

**A**: いや、必須じゃない。

**構成例**：

#### オンプレ接続あり
```
オンプレデータセンター
  ↓ VPN / ExpressRoute
Azure Hub VNet
  ↓
Spoke VNet（アプリ）
```

#### オンプレ接続なし（クラウドネイティブ）
```
Azure Hub VNet
  ↓
Spoke VNet（アプリ）
  ↓
インターネット経由でユーザーアクセス
```

**オンプレ接続が必要な場合**：
```
✓ 既存システムとの連携
✓ Active Directory連携
✓ データ移行
✓ ハイブリッドクラウド
```

**不要な場合**：
```
✓ 新規クラウドネイティブアプリ
✓ SaaS的な利用
✓ 完全クラウド移行
```

### Q13: FirewallとNSGの違いは？

**A**: 役割が違う。

**NSG（Network Security Group）**：
```
レベル: サブネット・NIC
機能: L4（IPアドレス、ポート）
用途: 基本的なフィルタリング
コスト: 無料
```

**Azure Firewall**：
```
レベル: VNet全体
機能: L7（FQDN、URL、脅威検知）
用途: 高度なフィルタリング
コスト: 約17万円/月〜
```

**使い分け**：
```
NSG:
- サブネット間の通信制御
- VM間の通信制御
- 基本的なセキュリティ

Firewall:
- インターネット向けトラフィック制御
- アプリケーションレベルのフィルタリング
- 脅威検知
- 一元管理
```

**両方使う**：
```
Firewall: Hub VNetに配置（全体の制御）
NSG: 各サブネットに配置（詳細制御）
  ↓
多層防御
```

---

## 運用編

### Q14: Terraformのバージョンアップはどうする？

**A**: 段階的にアップグレード。

**手順**：
```bash title="Terraformバージョンアップグレード手順"
1. リリースノート確認
   https://github.com/hashicorp/terraform/releases

2. 破壊的変更（Breaking Changes）確認

3. ローカルでテスト
   terraform init -upgrade
   terraform plan

4. 問題なければterraform.tf更新
   terraform {
     required_version = "~> 1.13"  # 更新
   }

5. Commit & Deploy

6. GitHub Actions設定も更新
   terraform_cli_version: '1.13.0'
```

**注意**：
```
メジャーバージョンアップ（1.x → 2.x）は慎重に
  ↓
破壊的変更が多い
  ↓
十分なテストが必要
```

### Q15: リソースを手動で変更しちゃったらどうする？

**A**: Terraformに取り込むか、戻す。

#### 方法1: Terraformに取り込む（推奨）

```bash title="手動変更をTerraformに反映"
# 現在の状態を取り込む
terraform apply -refresh-only

# Plan確認
terraform plan
# "No changes" と表示される

# .tfファイルも更新
# （手動変更内容をコードに反映）
```

#### 方法2: 手動変更を戻す

```bash title="Terraform定義通りに戻す"
# Terraformの定義通りに戻す
terraform apply
```

**予防策**：
```
Azure Portalでの手動変更を禁止
  ↓
Policy設定
  ↓
特定タグがないリソースは作成不可
```

### Q16: Stateファイルが壊れたらどうする？

**A**: バックアップから復元。

**対処法**：

#### バックアップがある場合
```bash title="Stateファイルをバックアップから復元"
# Stateファイルをバックアップから復元
az storage blob upload \
  --account-name $ARM_STORAGE_ACCOUNT_NAME \
  --container-name $ARM_CONTAINER_NAME \
  --name alz-mgmt.tfstate \
  --file ./backups/terraform-state-20260115.json \
  --overwrite
```

#### バックアップがない場合
```bash title="Azureの現状からState再構築"
# Azureの現状からStateを再構築
terraform import <リソースタイプ>.<名前> <Azure Resource ID>

# 全リソースをimport（大変）
terraform import azurerm_resource_group.management /subscriptions/.../resourceGroups/rg-management
terraform import azurerm_virtual_network.hub /subscriptions/.../virtualNetworks/vnet-hub
...
```

**予防策**：
```
定期的にStateをバックアップ
  ↓
Chapter 16のbackup-tfstate.shを使う
```

### Q17: チームメンバーが増えたら？

**A**: 権限とプロセスを整備。

**権限設定**：
```
1. Azure AD Groupに追加
2. GitHub Teamに追加
3. 必要な権限を付与
   - Reader（閲覧）
   - Contributor（作業）
   - Owner（管理）
```

**プロセス整備**：
```
1. ドキュメント整備
   - README更新
   - 運用手順書
   - アーキテクチャ図

2. オンボーディング
   - ツールインストール
   - 環境セットアップ
   - ペアプログラミング

3. コードレビュー
   - PRテンプレート
   - レビュー観点
   - 承認ルール
```

---

## カスタマイズ編

### Q18: 命名規則を変えたい

**A**: locals.tfを修正。

**デフォルト**：
```
rg-myorg-management-001
```

**変更例**：
```
prd-rg-myorg-management-001  # 環境prefix追加
```

**実装**：
```hcl title="環境prefix付き命名規則"
# locals.tf
locals {
  environment_prefix = var.environment  # "prd", "dev"
  
  resource_group_name = format(
    "%s-rg-%s-%s-%s",
    local.environment_prefix,
    var.root_id,
    var.purpose,
    var.default_postfix
  )
}
```

Chapter 15に詳しい例があるよ。

### Q19: 独自のポリシーを追加したい

**A**: lib/archetype_definitionsに追加。

**手順**：
```yaml
# lib/archetype_definitions/corp_custom.alz_archetype_override.yaml
policy_definitions_to_add:
  - name: Custom-Policy-Example
    display_name: "My Custom Policy"
    mode: "Indexed"
    policy_rule:
      if:
        field: "tags['CostCenter']"
        exists: false
      then:
        effect: "deny"

policy_assignments_to_add:
  - Custom-Policy-Example
```

Chapter 15でカスタムポリシーの作り方を解説しています。

### Q20: Management Group構造を変えたい

**A**: lib/architecture_definitionsを修正。

**手順**：
```yaml
# lib/architecture_definitions/alz_custom.alz_architecture_definition.yaml
management_groups:
  - id: myorg
    display_name: My Organization
    parent_id: null
  
  - id: production  # 追加
    display_name: Production
    parent_id: myorg
  
  - id: development  # 追加
    display_name: Development
    parent_id: myorg
```

Chapter 15に詳しく書いてある。

---

## トラブル編

### Q21: terraform applyがめっちゃ遅い

**A**: Firewallが原因かも。

**原因**：
```
Firewall作成: 30〜60分
  ↓
どうしようもない
  ↓
Azureの仕様
```

**対処法**：
```
コーヒー飲んで待つ☕
```

**時短のコツ**：
```
開発環境はFirewall無効化
  ↓
enabled_resources = {
  firewall = false
}
```

### Q22: GitHub Actionsで謎のエラー

**A**: ローカルで再現するか確認。

**手順**：
```
1. ローカルで同じコマンド実行
   terraform init
   terraform plan

2. 再現する
   → コードの問題
   → 修正

3. 再現しない
   → GitHub Actions固有の問題
   → Secrets確認
   → 権限確認
```

Chapter 14にトラブルシューティングガイドがある。

### Q23: "zones are not supported"エラー

**A**: Japan regionの問題。

**原因**：
```
Japan East/WestはAvailability Zones非対応
  ↓
でもzones指定してる
  ↓
エラー
```

**対処法**：
```hcl
# NG
zones = ["1", "2", "3"]

# OK
zones = []  # 空リスト
```

Chapter 3で詳しく解説しています。

---

## その他

### Q24: 他のクラウド（AWS、GCP）でも使える？

**A**: Terraformなら理論的には可能。

**実際**：
```
このプロジェクト：

- Azure専用
- azurermプロバイダー
- Azure固有の機能

マルチクラウド化するには：

- AWS providerを追加
- AWSリソースを定義
- モジュール分離
→ かなり大変
```

**推奨**：
```
Azure: このプロジェクト
AWS: AWS Landing Zone使う
GCP: GCP Foundation使う
  ↓
各クラウド専用のものを使う方が楽
```

### Q25: サポートはある？

**A**: いくつかの選択肢がある。

**コミュニティサポート**：
```
GitHub Issues:
- このプロジェクトのIssue
- モジュールのIssue

Stack Overflow:
- terraform タグ
- azure タグ
```

**有償サポート**：
```
Microsoft:
- Azure サポートプラン
- Professional Services

HashiCorp:
- Terraform Enterprise
- コンサルティング

パートナー:
- SIer
- クラウドインテグレーター
```

### Q26: ドキュメントはどこ？

**A**: このドキュメント群と公式ドキュメント。

**このプロジェクトのドキュメント**：
```
docs_new/
├── 00_はじめに.md
├── README.md
├── 01_基礎知識.md
├── 02_プロジェクト構造.md
├── 03_設定ファイル完全解説.md
...
└── 17_FAQ.md（これ）
```

**公式ドキュメント**：
```
Terraform:
https://www.terraform.io/docs

Azure Provider:
https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs

Azure Landing Zones（Microsoft公式）:
https://docs.microsoft.com/azure/cloud-adoption-framework/ready/landing-zone/
```

### Q27: 学習リソースは？

**A**: たくさんある。

**無料**：
```
Microsoft Learn:
https://docs.microsoft.com/learn/azure/

Terraform Tutorial:
https://learn.hashicorp.com/terraform

YouTube:
- Azure Friday
- HashiCorp
```

**有料**：
```
Udemy:
- Azure コース
- Terraform コース

Pluralsight:
- Azure Learning Path
- Terraform Learning Path
```

**実践**：
```
一番の学習は実際に触ること
  ↓
このプロジェクトをデプロイしてみる
  ↓
カスタマイズしてみる
  ↓
壊して直す
```

### Q28: 本番運用の前にチェックすることは？

**A**: チェックリストを使おう。

**セキュリティ**：
```
□ MFA有効化
□ 最小権限の原則
□ Private Endpoint使用
□ Secrets管理（Key Vault）
□ NSG・Firewall設定確認
```

**ネットワーク**：
```
□ アドレス空間重複なし
□ ルーティング確認
□ VPNまたはExpressRoute接続テスト
□ DNS設定確認
```

**監視**：
```
□ Log Analytics設定
□ アラートルール設定
□ コストアラート設定
□ Dashboard作成
```

**バックアップ**：
```
□ Terraform Stateバックアップ
□ DRプラン作成
□ ドキュメント整備
```

**テスト**：
```
□ 開発環境でフルテスト
□ ロールバック手順確認
□ 緊急連絡先確認
```

Chapter 16のベストプラクティスを見てください！

### Q29: 困ったときは？

**A**: この順番で確認。

**ステップ1**: このドキュメント
```
Chapter 14: トラブルシューティング
Chapter 17: FAQ（これ）
```

**ステップ2**: エラーメッセージをググる
```
"Error: zones are not supported" terraform azure
  ↓
Stack Overflow、GitHub Issuesに答えがある
```

**ステップ3**: GitHub Issueを見る
```
https://github.com/Azure/terraform-azurerm-avm-ptn-alz/issues
  ↓
似たような問題が報告されてるかも
```

**ステップ4**: 質問する
```
Stack Overflow:
- terraform タグ付けて質問
- エラーメッセージ、コード、環境情報を添える

GitHub Issue:
- 再現手順を明確に
- terraform version、provider version記載
```

**ステップ5**: サポートに連絡
```
Azureサポート契約がある場合
  ↓
サポートチケット作成
```

---

## 最後に

### Q30: 結局、Azure Landing Zonesって必要？

**A**: プロジェクト次第。

**必要な場合**：
```
✓ エンタープライズ
✓ 複数チーム
✓ 長期運用
✓ セキュリティ・ガバナンス重視
✓ 拡張性が必要

→ Azure Landing Zones使おう
```

**オーバースペックな場合**：
```
✓ 小規模プロジェクト
✓ 個人利用
✓ 短期間のPoC
✓ シンプルな構成

→ 普通にAzureリソース作ればOK
```

**判断基準**：
```
「後で拡張する可能性がある？」
  ↓
Yes → Azure Landing Zones
No → シンプルにいこう
```

**このドキュメントの目的**：
```
Azure Landing Zonesを理解して
  ↓
自分で構築・カスタマイズできるようになる
  ↓
最高の教科書
```

**達成できた？**
```
Chapter 00: はじめに
Chapter 01: 基礎知識
Chapter 02: プロジェクト構造
Chapter 03: 設定ファイル完全解説
Chapter 04: 変数定義
Chapter 05: ローカル変数
Chapter 06: 設定テンプレート
Chapter 07: リソースグループ
Chapter 08: 管理グループとポリシー
Chapter 09: 管理リソース
Chapter 10: Hub-and-Spoke
Chapter 11: Virtual WAN
Chapter 12: GitHub Actions
Chapter 13: デプロイ手順
Chapter 14: トラブルシューティング
Chapter 15: カスタマイズ
Chapter 16: ベストプラクティス
Chapter 17: FAQ

全部読めば完璧！
```

**最後のメッセージ**：
```
完璧を目指さなくていい
  ↓
まず動かしてみる
  ↓
失敗から学ぶ
  ↓
少しずつ改善
  ↓
それがベストプラクティス
```

**がんばって！🚀**

---

## まとめ

**この教科書で学んだこと**：

### 基礎編（Chapter 00-02）
- Azure Landing Zonesの全体像と目的
- Terraform・Azure・IaCの基礎知識
- プロジェクトファイル構造の理解

### 設定編（Chapter 03-06）
- `platform-landing-zone.auto.tfvars`の完全解説
- `variables.tf`の型定義とバリデーション
- `locals.tf`のローカル変数活用
- `config-templating`モジュールのテンプレート処理

### リソース作成編（Chapter 07-11）
- リソースグループの作成
- 管理グループとポリシーの構成
- Log Analytics・DCRなど管理リソース
- Hub-and-Spoke / Virtual WANネットワーク

### 運用編（Chapter 12-14）
- GitHub Actionsによる自動デプロイ
- 実際のデプロイ手順
- トラブルシューティング

### 応用編（Chapter 15-17）
- カスタマイズ方法
- ベストプラクティス
- よくある質問（FAQ）

**次のステップ**：
1. 開発環境で実際にデプロイしてみる
2. 自社の要件に合わせてカスタマイズする
3. 段階的に本番環境へ適用する
4. チームでナレッジを共有する

お疲れ様でした！

---

**所要時間**: 45分  
**難易度**: ★★☆☆☆  
**前**: [16_ベストプラクティス.md](./16_ベストプラクティス.md)  
**次**: [README.md](./README.md)（教科書の目次に戻る）

---

## 索引

各Chapterへのリンク：

- [Chapter 00: はじめに](./00_はじめに.md)
- [Chapter 01: 基礎知識](./01_基礎知識.md)
- [Chapter 02: プロジェクト構造](./02_プロジェクト構造.md)
- [Chapter 03: 設定ファイル完全解説](./03_設定ファイル完全解説.md)
- [Chapter 04: 変数定義](./04_変数定義.md)
- [Chapter 05: ローカル変数](./05_ローカル変数.md)
- [Chapter 06: 設定テンプレート](./06_設定テンプレート.md)
- [Chapter 07: リソースグループ](./07_リソースグループ.md)
- [Chapter 08: 管理グループとポリシー](./08_管理グループとポリシー.md)
- [Chapter 09: 管理リソース](./09_管理リソース.md)
- [Chapter 10: Hub-and-Spoke](./10_Hub-and-Spoke.md)
- [Chapter 11: Virtual WAN](./11_Virtual_WAN.md)
- [Chapter 12: GitHub Actions](./12_GitHub_Actions.md)
- [Chapter 13: デプロイ手順](./13_デプロイ手順.md)
- [Chapter 14: トラブルシューティング](./14_トラブルシューティング.md)
- [Chapter 15: カスタマイズ](./15_カスタマイズ.md)
- [Chapter 16: ベストプラクティス](./16_ベストプラクティス.md)
- [Chapter 17: FAQ](./17_FAQ.md)（これ）

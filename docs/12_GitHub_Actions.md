# 12. GitHub Actions - 自動デプロイの仕組み

## このChapterでやること

GitHub Actionsで自動デプロイを理解しよう。

**GitHub Actionsって何？**
GitHub上で動くCI/CD（継続的インテグレーション/デリバリー）サービス。

**CI/CDって何？**
```
CI（Continuous Integration）：
コードをプッシュ
  ↓
自動でテスト・検証
  ↓
問題があれば教えてくれる

CD（Continuous Delivery）：
mainブランチにマージ
  ↓
自動でデプロイ
  ↓
本番環境に反映
```

**例えるなら**：

- **手動デプロイ**：料理を全部自分で作る
- **GitHub Actions**：オートメーションキッチン（自動で調理）

**📊 CI/CDパイプライン全体図**

```
【開発の流れ】

1. コード変更
   ↓
   開発者が feature ブランチで作業
   variables.tf を編集
   ↓

2. Pull Request作成
   ↓
   feature → main への PR
   ↓

3. 自動検証（terraform plan）
   ━━━━━━━━━━━━━━━━━━━━━━━
   │ GitHub Actions 起動       │
   │   ↓                      │
   │ terraform init            │
   │   ↓                      │
   │ terraform fmt -check      │
   │   ↓                      │
   │ terraform validate        │
   │   ↓                      │
   │ terraform plan           │
   │   ↓                      │
   │ 変更内容をPRにコメント    │
   ━━━━━━━━━━━━━━━━━━━━━━━
   ↓

4. レビュー・承認
   ↓
   チームメンバーが確認
   「OK、承認！」
   ↓

5. マージ
   ↓
   main ブランチにマージ
   ↓

6. 自動デプロイ（terraform apply）
   ━━━━━━━━━━━━━━━━━━━━━━━
   │ GitHub Actions 起動       │
   │   ↓                      │
   │ terraform init            │
   │   ↓                      │
   │ terraform apply -auto-approve │
   │   ↓                      │
   │ Azure にリソース作成      │
   │   ↓                      │
   │ 結果を Slack に通知      │
   ━━━━━━━━━━━━━━━━━━━━━━━
   ↓

7. 本番環境に反映 ✅
```

**🔄 詳細なワークフロー図**

```
┌─────────────┐
│  開発者PC   │
│             │
│ git push    │
└──────┬──────┘
       │
       ↓
┌──────────────────────────────────┐
│        GitHub Repository         │
│  ┌────────────────────────────┐  │
│  │ Pull Request               │  │
│  │   feature → main           │  │
│  └────────┬───────────────────┘  │
│           ↓                      │
│  ┌────────────────────────────┐  │
│  │  GitHub Actions            │  │
│  │  (Workflow実行)            │  │
│  │                            │  │
│  │  Step 1: Checkout          │  │
│  │  Step 2: Setup Terraform   │  │
│  │  Step 3: terraform plan    │  │
│  └────────┬───────────────────┘  │
└───────────┼──────────────────────┘
            │
            ↓
    ┌───────────────┐
    │    Azure      │
    │  (認証・接続) │
    │               │
    │  OIDC認証     │
    │  ↓            │
    │  State取得    │
    │  ↓            │
    │  Plan実行     │
    └───────────────┘
            ↓
    結果をPRにコメント
```

---

## なぜGitHub Actionsを使う？

### 1. ヒューマンエラー防止

```
手動デプロイ：
terraform plan → 確認 → terraform apply
  ↓
コマンド打ち間違い
環境変数設定し忘れ
承認なしでデプロイ
  ↓
事故る
```

```
GitHub Actions：
PRを作る → 自動でplan → レビュー → マージで自動apply
  ↓
手順が自動化
  ↓
間違いが減る
```

### 2. 履歴が残る

```
手動デプロイ：
誰がいつデプロイしたかわからない

GitHub Actions：
全部GitHubに記録
- いつ
- 誰が
- 何を
- どうなったか
  ↓
完全にトレース可能
```

### 3. 承認フロー

```
GitHub Actions：
PRでplan結果を確認
  ↓
レビュー・承認
  ↓
マージ後にapply
  ↓
勝手にデプロイされない
```

### 4. 並行実行の防止

```
手動デプロイ：
2人が同時にapply実行
  ↓
ステートファイル破損
  ↓
やばい

GitHub Actions：
同時実行を制御
  ↓
安全
```

---

## Part 1: ワークフローの構成

このプロジェクトには2つのワークフローがある：

### 1. CI（Continuous Integration）

**ファイル**：`.github/workflows/ci.yaml`

```yaml
name: 01 Azure Landing Zones Continuous Integration
on:
  pull_request:
    branches:
      - main
  workflow_dispatch:
    inputs:
      terraform_cli_version:
        description: 'Terraform CLI Version'
        required: true
        default: 'latest'
        type: string

jobs:
  validate_and_plan:
    uses: shuhei0720org01/alz-mgmt-templates/.github/workflows/ci-template.yaml@main
    name: 'CI'
    permissions:
      id-token: write
      contents: read
      pull-requests: write
    with:
      root_module_folder_relative_path: '.'
      terraform_cli_version: ${{ inputs.terraform_cli_version }}
```

**何してる？**：
```
1. PRが作られる
2. 自動でterraform plan実行
3. PR上にplan結果を表示
4. レビュアーが確認
```

### 2. CD（Continuous Delivery）

**ファイル**：`.github/workflows/cd.yaml`

```yaml
name: 02 Azure Landing Zones Continuous Delivery
on:
  push:
    branches:
      - main
  workflow_dispatch:
    inputs:
      terraform_action:
        description: 'Terraform Action to perform'
        required: true
        default: 'apply'
        type: choice
        options:
          - 'apply'
          - 'destroy'
      terraform_cli_version:
        description: 'Terraform CLI Version'
        required: true
        default: 'latest'
        type: string

jobs:
  plan_and_apply:
    uses: shuhei0720org01/alz-mgmt-templates/.github/workflows/cd-template.yaml@main
    name: 'CD'
    permissions:
      id-token: write
      contents: read
    with:
      terraform_action: ${{ inputs.terraform_action }}
      root_module_folder_relative_path: '.'
      terraform_cli_version: ${{ inputs.terraform_cli_version }}
```

**何してる？**：
```
1. mainブランチにpush（マージ）される
2. 自動でterraform apply実行
3. Azureに反映
```

---

## Part 2: ワークフローの詳細解説

### on（トリガー）

#### CI: pull_request

```yaml
on:
  pull_request:
    branches:
      - main
```

**何？**：PRが作られたら実行

```
feature/add-vnet ブランチ
  ↓ PR作成
main ブランチ
  ↓ トリガー
CI実行
```

#### CD: push

```yaml
on:
  push:
    branches:
      - main
```

**何？**：mainブランチにpushされたら実行

```
PR承認
  ↓ マージ
main ブランチ
  ↓ トリガー
CD実行
```

#### workflow_dispatch

```yaml
on:
  workflow_dispatch:
    inputs:
      terraform_action:
        description: 'Terraform Action to perform'
        required: true
        default: 'apply'
        type: choice
        options:
          - 'apply'
          - 'destroy'
```

**何？**：手動実行

**使い道**：
```
GitHub UI → Actions → ワークフロー選択 → Run workflow
  ↓
terraform_actionを選択
  - apply：リソース作成
  - destroy：リソース削除
  ↓
手動で実行
```

**便利**：
```
緊急時：

- destroy実行（全削除）
- 特定バージョンのTerraformでapply
```

### permissions

```yaml
permissions:
  id-token: write
  contents: read
  pull-requests: write
```

**何？**：GitHubトークンの権限

#### id-token: write

```
OIDC（OpenID Connect）でAzureにログイン
  ↓
パスワード不要
  ↓
セキュア
```

後で詳しく解説するね。

#### contents: read

```
リポジトリのコードを読む
  ↓
Terraformファイルを取得
```

#### pull-requests: write

```
PRにコメント投稿
  ↓
plan結果を表示
```

### jobs

```yaml
jobs:
  validate_and_plan:
    uses: shuhei0720org01/alz-mgmt-templates/.github/workflows/ci-template.yaml@main
    with:
      root_module_folder_relative_path: '.'
      terraform_cli_version: ${{ inputs.terraform_cli_version }}
```

**何してる？**：再利用可能なワークフローを呼び出し

**構造**：
```
alz-mgmt（このリポジトリ）
  ├── .github/workflows/ci.yaml（トリガー定義だけ）
  └── 実際の処理は別リポジトリ
        ↓
alz-mgmt-templates
  └── .github/workflows/ci-template.yaml（実際の処理）
```

**メリット**：
```
複数のプロジェクトで同じワークフロー使える
  ↓
1箇所修正すれば全プロジェクトに反映
  ↓
DRY（Don't Repeat Yourself）
```

---

## Part 3: Reusable Workflow（再利用可能ワークフロー）

### ci-template.yaml（想定される内容）

実際のファイルは`alz-mgmt-templates`リポジトリにありますが、典型的な内容は以下の通りです：

```yaml
name: CI Template
on:
  workflow_call:
    inputs:
      root_module_folder_relative_path:
        required: true
        type: string
      terraform_cli_version:
        required: true
        type: string

jobs:
  terraform_plan:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ inputs.terraform_cli_version }}
      
      - name: Azure Login (OIDC)
        uses: azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
      
      - name: Terraform Init
        run: terraform init
        working-directory: ${{ inputs.root_module_folder_relative_path }}
      
      - name: Terraform Validate
        run: terraform validate
        working-directory: ${{ inputs.root_module_folder_relative_path }}
      
      - name: Terraform Plan
        run: terraform plan -out=tfplan
        working-directory: ${{ inputs.root_module_folder_relative_path }}
      
      - name: Comment PR
        uses: actions/github-script@v7
        with:
          script: |
            // plan結果をPRにコメント
```

### ステップ解説

#### 1. Checkout

```yaml
- name: Checkout
  uses: actions/checkout@v4
```

**何？**：リポジトリのコードをクローン

```
GitHub Actions Runner（実行環境）
  ↓
git clone
  ↓
コードを取得
```

#### 2. Setup Terraform

```yaml
- name: Setup Terraform
  uses: hashicorp/setup-terraform@v3
  with:
    terraform_version: ${{ inputs.terraform_cli_version }}
```

**何？**：Terraformをインストール

```
指定されたバージョンのTerraformをダウンロード
  ↓
PATHに追加
  ↓
terraform コマンドが使える
```

#### 3. Azure Login (OIDC)

```yaml
- name: Azure Login (OIDC)
  uses: azure/login@v2
  with:
    client-id: ${{ secrets.AZURE_CLIENT_ID }}
    tenant-id: ${{ secrets.AZURE_TENANT_ID }}
    subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
```

**何？**：Azureにログイン（OIDC）

**OIDC（OpenID Connect）**：
```
従来：

- Service Principalのパスワードをシークレットに保存
- パスワードが漏洩するリスク
- 定期的なローテーション必要

OIDC：

- パスワード不要
- 一時的なトークンで認証
- より安全
```

**仕組み**：
```
1. GitHub Actions → Azure ADに「私はGitHubです」と証明
2. Azure AD → GitHubを信頼（事前設定）
3. Azure AD → 一時トークン発行
4. GitHub Actions → トークンでAzureにアクセス
```

#### 4. Terraform Init

```yaml
- name: Terraform Init
  run: terraform init
  working-directory: ${{ inputs.root_module_folder_relative_path }}
```

**何？**：Terraformの初期化

```
terraform init
  ↓
- Providerダウンロード
- Backend設定読み込み
- Stateファイル取得
```

**Backend設定**：
```hcl
# terraform.tf
backend "azurerm" {}
```

**環境変数で設定**：
```bash
export ARM_STORAGE_ACCOUNT_NAME="stterraform12345"
export ARM_CONTAINER_NAME="tfstate"
export ARM_KEY="alz-mgmt.tfstate"
export ARM_RESOURCE_GROUP_NAME="rg-terraform-state"
```

GitHub ActionsのSecretsに保存しておく。

#### 5. Terraform Validate

```yaml
- name: Terraform Validate
  run: terraform validate
```

**何？**：構文チェック

```
terraform validate
  ↓
- .tfファイルの構文確認
- 変数の参照確認
- モジュールの整合性チェック
```

**エラー例**：
```
Error: Missing required argument
  on main.tf line 10:
  10: resource "azurerm_resource_group" "example" {
```

#### 6. Terraform Plan

```yaml
- name: Terraform Plan
  run: terraform plan -out=tfplan
```

**何？**：変更内容の確認

```
terraform plan
  ↓
- 現在のState
- 設定ファイル
- 実際のAzure
を比較
  ↓
何が変わるか表示
```

**-out=tfplan**：
```
plan結果をファイルに保存
  ↓
apply時に使う
  ↓
planとapplyで差異がない
```

#### 7. Comment PR

```yaml
- name: Comment PR
  uses: actions/github-script@v7
  with:
    script: |
      // plan結果をPRにコメント
```

**何？**：plan結果をPRに投稿

**例**：
```
## Terraform Plan
```
Terraform will perform the following actions:

# azurerm_resource_group.example will be created
+ resource "azurerm_resource_group" "example" {
    + name     = "rg-example"
    + location = "japaneast"
  }

Plan: 1 to add, 0 to change, 0 to destroy.
```
```

**便利**：
レビュアーがPR画面で変更内容を確認できる。

---

## Part 4: CD Workflow（Apply）

### cd-template.yaml（想定される内容）

```yaml
name: CD Template
on:
  workflow_call:
    inputs:
      terraform_action:
        required: true
        type: string
      root_module_folder_relative_path:
        required: true
        type: string
      terraform_cli_version:
        required: true
        type: string

jobs:
  terraform_apply:
    runs-on: ubuntu-latest
    environment: production  # ←環境設定
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ inputs.terraform_cli_version }}
      
      - name: Azure Login (OIDC)
        uses: azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
      
      - name: Terraform Init
        run: terraform init
        working-directory: ${{ inputs.root_module_folder_relative_path }}
      
      - name: Terraform Apply
        if: ${{ inputs.terraform_action == 'apply' }}
        run: terraform apply -auto-approve
        working-directory: ${{ inputs.root_module_folder_relative_path }}
      
      - name: Terraform Destroy
        if: ${{ inputs.terraform_action == 'destroy' }}
        run: terraform destroy -auto-approve
        working-directory: ${{ inputs.root_module_folder_relative_path }}
```

### CI との違い

#### 1. environment: production

```yaml
environment: production
```

**何？**：環境設定

**使い道**：
```
GitHub設定 → Environments → production
  ↓
- 承認者を設定（required reviewers）
- タイムアウト設定
- Environment Secrets
```

**効果**：
```
CDワークフロー実行
  ↓
承認待ち
  ↓
承認者が承認
  ↓
apply実行
```

**安全**：
```
mainにマージされても即座にapplyされない
  ↓
一度止まる
  ↓
承認後にapply
```

#### 2. -auto-approve

```yaml
run: terraform apply -auto-approve
```

**何？**：確認スキップ

```
通常：
terraform apply
  ↓
Do you want to perform these actions? (yes/no): ←手動入力

CI/CD：
terraform apply -auto-approve
  ↓
確認スキップで自動実行
```

**安全性**：
```
planで確認済み
  ↓
PR承認済み
  ↓
Environment承認済み
  ↓
-auto-approveでOK
```

#### 3. terraform_action分岐

```yaml
- name: Terraform Apply
  if: ${{ inputs.terraform_action == 'apply' }}
  run: terraform apply -auto-approve

- name: Terraform Destroy
  if: ${{ inputs.terraform_action == 'destroy' }}
  run: terraform destroy -auto-approve
```

**何？**：applyかdestroyを選べる

```
workflow_dispatch（手動実行）
  ↓
terraform_action選択
  - apply → リソース作成/更新
  - destroy → リソース削除
```

---

## Part 5: Secrets設定

### 必要なSecrets

```
AZURE_CLIENT_ID：Azure ADアプリのClient ID
AZURE_TENANT_ID：Azure ADのTenant ID
AZURE_SUBSCRIPTION_ID：Subscription ID
ARM_STORAGE_ACCOUNT_NAME：StateファイルのStorage Account
ARM_CONTAINER_NAME：Stateファイルのコンテナ
ARM_KEY：Stateファイル名
ARM_RESOURCE_GROUP_NAME：StateファイルのResource Group
```

### 設定場所

```
GitHub → Settings → Secrets and variables → Actions
  ↓
- Repository secrets（リポジトリ全体）
- Environment secrets（環境ごと）
```

**使い分け**：
```
Repository secrets：

- Tenant ID
- Storage Account情報
→ 環境問わず共通

Environment secrets：

- Client ID（本番用、開発用）
- Subscription ID（本番用、開発用）
→ 環境ごとに違う
```

---

## Part 6: OIDC設定（Azure側）

### 1. Azure ADアプリ作成

```bash
az ad app create --display-name "GitHub Actions OIDC"
```

### 2. Service Principal作成

```bash
APP_ID="..."  # ↑で取得したApp ID

az ad sp create --id $APP_ID
```

### 3. Federated Credential設定

```bash
az ad app federated-credential create \
  --id $APP_ID \
  --parameters @federated-credential.json
```

**federated-credential.json**：
```json
{
  "name": "github-actions-oidc",
  "issuer": "https://token.actions.githubusercontent.com",
  "subject": "repo:shuhei0720org01/alz-mgmt:ref:refs/heads/main",
  "audiences": [
    "api://AzureADTokenExchange"
  ]
}
```

**subject**：
```
repo:<owner>/<repo>:ref:refs/heads/<branch>
  ↓
このリポジトリのこのブランチからのアクセスだけ許可
```

### 4. 権限付与

```bash
SUBSCRIPTION_ID="..."

az role assignment create \
  --assignee $APP_ID \
  --role "Owner" \
  --scope "/subscriptions/$SUBSCRIPTION_ID"
```

**Owner**：強力

**本番では**：
```
- Contributorにする
- カスタムロールで最小権限
```

---

## 実践：ワークフローを動かしてみよう

### 1. ブランチ作成

```bash
git checkout -b feature/add-resource-group
```

### 2. コード変更

```hcl
# main.tf
resource "azurerm_resource_group" "test" {
  name     = "rg-test"
  location = "japaneast"
}
```

### 3. Commit & Push

```bash
git add main.tf
git commit -m "Add test resource group"
git push origin feature/add-resource-group
```

### 4. PR作成

```
GitHub → Pull requests → New pull request
  ↓
base: main ← compare: feature/add-resource-group
  ↓
Create pull request
```

### 5. CI実行確認

```
PR画面 → Checks タブ
  ↓
01 Azure Landing Zones Continuous Integration
  ↓
実行中...
  ↓
完了
```

**結果**：
```
✓ Checkout
✓ Setup Terraform
✓ Azure Login
✓ Terraform Init
✓ Terraform Validate
✓ Terraform Plan
✓ Comment PR
```

**PR画面にコメント**：
```
## Terraform Plan
...
Plan: 1 to add, 0 to change, 0 to destroy.
```

### 6. レビュー & 承認

```
PR画面 → Files changed
  ↓
変更内容確認
  ↓
Review changes → Approve
```

### 7. マージ

```
PR画面 → Merge pull request
  ↓
Confirm merge
```

### 8. CD実行確認

```
Actions タブ
  ↓
02 Azure Landing Zones Continuous Delivery
  ↓
実行中...
  ↓
Environment承認待ち（設定している場合）
  ↓
承認
  ↓
完了
```

### 9. Azure確認

```bash
az group show --name rg-test
```

**作られてる！**

---

## デバッグ技

### ワークフロー実行ログ

```
Actions タブ → ワークフロー選択 → 実行選択
  ↓
各ステップのログが見える
```

**エラー時**：
```
✓ Checkout
✓ Setup Terraform
✓ Azure Login
✗ Terraform Init  ←ここで失敗
```

**ログ展開**：
```
Error: Failed to get existing workspaces: storage account not found
```

**原因**：Storage Accountの設定間違い

### Terraform Debug

```yaml
- name: Terraform Plan
  run: terraform plan -out=tfplan
  env:
    TF_LOG: DEBUG  # ←デバッグログ有効化
```

**詳細ログ**：
```
全APIリクエスト・レスポンスが見える
  ↓
問題の特定が楽
```

### Re-run Jobs

```
Actions → 失敗したワークフロー → Re-run jobs
  ↓
もう一度実行
```

**一時的なエラー**（ネットワーク等）は再実行で直ることある。

---

## よくあるエラー

### エラー1: OIDC認証失敗

```
Error: Unable to get ACTIONS_ID_TOKEN_REQUEST_URL env variable
```

**原因**：
```yaml
permissions:
  id-token: write  # ←これがない
```

**対処法**：permissions追加

### エラー2: Backend設定エラー

```
Error: Failed to get existing workspaces
```

**原因**：環境変数が設定されてない

**対処法**：
```
GitHub Secrets確認：

- ARM_STORAGE_ACCOUNT_NAME
- ARM_CONTAINER_NAME
- ARM_KEY
- ARM_RESOURCE_GROUP_NAME
```

### エラー3: Terraform Lock

```
Error: Error acquiring the state lock
```

**原因**：
```
前回のワークフローが失敗
  ↓
Lockが残ってる
  ↓
次の実行がブロックされる
```

**対処法**：
```bash
# Azure Portal → Storage Account → tfstate コンテナ → .terraform.lock.info
# 削除

# または
terraform force-unlock <LOCK_ID>
```

### エラー4: 権限不足

```
Error: insufficient privileges to complete the operation
```

**原因**：Service Principalの権限が足りない

**対処法**：
```bash
# 権限確認
az role assignment list --assignee <CLIENT_ID>

# 権限追加
az role assignment create \
  --assignee <CLIENT_ID> \
  --role "Contributor" \
  --scope "/subscriptions/<SUBSCRIPTION_ID>"
```

---

## ベストプラクティス

### 1. Branch Protection

```
GitHub → Settings → Branches → Branch protection rules
  ↓
main ブランチに設定：

- Require pull request reviews before merging
- Require status checks to pass before merging
  - CI（terraform plan）
- Require branches to be up to date before merging
```

**効果**：
```
mainに直接pushできない
  ↓
必ずPR経由
  ↓
CI通過しないとマージできない
```

### 2. Environment Protection

```
GitHub → Settings → Environments → production
  ↓
- Required reviewers（承認者）
- Wait timer（待機時間）
```

**効果**：
```
mainにマージ
  ↓
自動で即applyされない
  ↓
承認者が承認
  ↓
apply実行
```

### 3. Concurrency制御

```yaml
concurrency:
  group: terraform-${{ github.ref }}
  cancel-in-progress: false
```

**何？**：同時実行の制御

```
2つのPRが同時にマージ
  ↓
2つのCDが同時実行
  ↓
State Lock競合
  ↓
エラー

concurrency設定：
1つ目のCDが実行中
  ↓
2つ目のCDは待機
  ↓
1つ目が完了
  ↓
2つ目が実行
```

### 4. Terraform Version固定

```yaml
terraform_cli_version: '1.12.0'  # ←バージョン固定
```

**なぜ？**
```
'latest'だと：

- 突然新バージョンが使われる
- 互換性問題でエラー

固定すると：

- 安定
- アップグレードは計画的に
```

---

## まとめ

**GitHub Actions の流れ**：
```
1. ブランチ作成・コード変更
2. PR作成
3. CI実行（terraform plan）
4. PR上で確認
5. レビュー・承認
6. マージ
7. CD実行（terraform apply）
8. Environment承認（設定している場合）
9. Azureに反映
```

**メリット**：

- 自動化（ヒューマンエラー削減）
- 履歴が残る
- 承認フロー
- 並行実行防止

**重要な設定**：

- **OIDC**：パスワードレス認証
- **Secrets**：機密情報の管理
- **Environment**：承認フロー
- **Branch Protection**：mainブランチ保護

次のChapterでは、実際のデプロイ手順を見ていきます。
ゼロから環境を作る時のステップバイステップガイド。

---

**所要時間**: 60分  
**難易度**: ★★★★☆  
**前**: [11_Virtual_WAN.md](./11_Virtual_WAN.md)  
**次**: [13_デプロイ手順.md](./13_デプロイ手順.md)

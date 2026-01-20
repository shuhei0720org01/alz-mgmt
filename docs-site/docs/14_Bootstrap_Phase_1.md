# 14. Bootstrap Phase 1 - 準備とBootstrap環境のセットアップ

!!! info "この章で学ぶこと"
    Azure公式Bootstrap手順のPhase 1を実践します：

    1. 前提条件の準備
    2. Starter Moduleの選択
    3. Bootstrap環境のセットアップ
    4. Phase 1デプロイ実行

    この章を読めば、Landing Zonesをデプロイする準備が整います。

---

## Part 1: 前提条件の準備

### 必要なツール

まず、必要なツールをインストールします。

=== "PowerShell 7.4+"

    **Windows**:
    
    ```powershell title="PowerShellでインストール"
    winget install Microsoft.PowerShell
    ```
    
    **macOS**:
    
    ```bash title="Homebrewでインストール"
    brew install powershell
    ```
    
    **Linux**:
    
    ```bash title="aptでインストール（Ubuntu/Debian）"
    sudo apt-get update
    sudo apt-get install -y powershell
    ```
    
    **確認**:
    
    ```powershell
    $PSVersionTable.PSVersion
    # 7.4.0以上であればOK
    ```

=== "Azure CLI 2.55+"

    **Windows**:
    
    ```powershell title="MSIインストーラー"
    # https://aka.ms/installazurecliwindows
    # からダウンロードしてインストール
    ```
    
    **macOS**:
    
    ```bash title="Homebrewでインストール"
    brew update && brew install azure-cli
    ```
    
    **Linux**:
    
    ```bash title="スクリプトでインストール"
    curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
    ```
    
    **確認**:
    
    ```bash
    az version
    # "azure-cli": "2.55.0" 以上であればOK
    ```

=== "Git"

    **Windows**:
    
    ```powershell title="Git for Windows"
    winget install Git.Git
    ```
    
    **macOS**:
    
    ```bash title="Homebrewでインストール"
    brew install git
    ```
    
    **Linux**:
    
    ```bash title="aptでインストール"
    sudo apt-get install git
    ```
    
    **確認**:
    
    ```bash
    git --version
    # git version 2.x.x であればOK
    ```

!!! tip "すべてインストールできたか確認"
    ```powershell
    # PowerShell 7.4+
    $PSVersionTable.PSVersion
    
    # Azure CLI 2.55+
    az version
    
    # Git
    git --version
    ```

### Azure Subscriptions

4つのSubscriptionを準備します。

=== "必要なSubscription"

    Landing Zonesには4つのSubscriptionが必要です：
    
    1. **Management**: 管理リソース（Log Analytics等）
    2. **Connectivity**: ネットワークリソース（Hub VNet等）
    3. **Identity**: ID管理リソース（AD等）
    4. **Security**: セキュリティリソース（Sentinel等）

=== "Subscriptionの作成"

    **Enterprise Agreement (EA)の場合**:
    
    ```bash title="EA Subscriptionの作成"
    az account create \
      --offer-type MS-AZR-0017P \
      --display-name "ALZ-Management" \
      --enrollment-account-name <enrollment-account-name>
    ```
    
    **Microsoft Customer Agreement (MCA)の場合**:
    
    Azureポータルから作成：
    
    1. Azureポータルにログイン
    2. "Subscriptions" を検索
    3. "+ Add" をクリック
    4. Subscription名を入力（例：ALZ-Management）
    5. Billing accountを選択
    6. "Create" をクリック

=== "Subscription IDの確認"

    作成したSubscriptionのIDをメモします：
    
    ```bash title="Subscription一覧を表示"
    az account list --output table
    ```
    
    ```text title="出力例"
    Name             SubscriptionId                        TenantId
    ---------------  ------------------------------------  ------------------------------------
    ALZ-Management   11111111-1111-1111-1111-111111111111  aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa
    ALZ-Connectivity 22222222-2222-2222-2222-222222222222  aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa
    ALZ-Identity     33333333-3333-3333-3333-333333333333  aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa
    ALZ-Security     44444444-4444-4444-4444-444444444444  aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa
    ```
    
    これらのIDを控えておきます。

!!! warning "命名規則"
    Subscription名は組織の命名規則に従ってください。この教科書では「ALZ-」プレフィックスを使用します。

### Azure権限設定

適切な権限を持つアカウントでログインします。

=== "必要な権限"

    **すべてのIaCオプション（Bicep/Terraform）で必要**:
    
    - 親Management Groupに対する `Owner` 権限
    - 4つのSubscriptionに対する `Owner` 権限
    
    **Bicep (AVM)のみ追加で必要**:
    
    - Root `/` レベルに対する `User Access Administrator` 権限

=== "権限の確認"

    ```bash title="現在の権限を確認"
    # Management Groupの権限確認
    az role assignment list \
      --scope "/providers/Microsoft.Management/managementGroups/<mg-id>" \
      --assignee <your-object-id> \
      --output table
    
    # Subscriptionの権限確認
    az role assignment list \
      --scope "/subscriptions/<subscription-id>" \
      --assignee <your-object-id> \
      --output table
    ```

=== "Ownerロールの付与"

    権限がない場合、管理者に依頼して付与してもらいます：
    
    ```bash title="Ownerロールの付与"
    # Management Groupへの付与
    az role assignment create \
      --assignee <user-email-or-object-id> \
      --role "Owner" \
      --scope "/providers/Microsoft.Management/managementGroups/<mg-id>"
    
    # Subscriptionへの付与
    az role assignment create \
      --assignee <user-email-or-object-id> \
      --role "Owner" \
      --scope "/subscriptions/<subscription-id>"
    ```

!!! danger "Rootアクセスについて"
    Bicep (AVM)を使用する場合、Root `/` レベルの `User Access Administrator` が必要です。詳細は[Root Accessドキュメント](https://azure.github.io/Azure-Landing-Zones/accelerator/1_prerequisites/root-access/)を参照してください。

### Version Control Systemの選択

GitHub、Azure DevOps、またはLocal File Systemを選択します。

=== "GitHub（推奨）"

    **必要なもの**:
    
    - GitHubアカウント
    - 組織（Organization）またはPersonalアカウント
    - Personal Access Token (PAT)
    
    **PATの作成**:
    
    1. GitHub → Settings → Developer settings
    2. Personal access tokens → Tokens (classic)
    3. Generate new token (classic)
    4. Scopes:
        - `repo` (full control)
        - `workflow`
        - `admin:org` (Organization使用時)
    5. Generate token → トークンをコピー

=== "Azure DevOps"

    **必要なもの**:
    
    - Azure DevOps組織
    - Personal Access Token (PAT)
    
    **PATの作成**:
    
    1. Azure DevOps → User settings → Personal access tokens
    2. + New Token
    3. Scopes:
        - Code: Read & write
        - Build: Read & execute
        - Service Connections: Read, query, & manage
    4. Create → トークンをコピー

=== "Local File System"

    **特徴**:
    
    - VCSなし（手動管理）
    - テスト・学習用途
    
    **必要なもの**:
    
    - ローカルフォルダのみ

!!! tip "この教科書ではGitHubを使用"
    以降の手順はGitHubを前提に説明します。Azure DevOpsも同様の手順で実行できます。

### インターネットアクセス

プロキシ環境の場合の対処方法です。

=== "プロキシなし"

    通常のインターネット接続があればOKです。

=== "プロキシあり"

    **PowerShell設定**:
    
    ```powershell title="プロキシ設定"
    $env:HTTP_PROXY = "http://proxy.example.com:8080"
    $env:HTTPS_PROXY = "http://proxy.example.com:8080"
    ```
    
    **Azure CLI設定**:
    
    ```bash title="Azure CLIプロキシ設定"
    export HTTP_PROXY=http://proxy.example.com:8080
    export HTTPS_PROXY=http://proxy.example.com:8080
    
    az config set proxy.http_proxy=http://proxy.example.com:8080
    az config set proxy.https_proxy=http://proxy.example.com:8080
    ```

=== "プロキシ回避策"

    !!! tip "Azure VMを使う"
        企業プロキシ環境の場合、一時的にAzure VMを立ててそこから実行するのも手です：
        
        1. Azure Portal → Virtual machines → Create
        2. Ubuntu 22.04 LTSを選択
        3. Standard B2s以上
        4. PowerShell・Azure CLI・Gitをインストール
        5. Bootstrapを実行
        6. 完了後、VMを削除

---

## Part 2: Starter Moduleの選択

### Starter Moduleとは

Landing Zonesの初期構成テンプレートです。

```text title="Starter Moduleの役割"
Starter Module
  ↓
  初期設定（tfvars/parameters）が含まれたテンプレート
  ↓
  組織の要件に合わせてカスタマイズ
  ↓
  デプロイ
```

!!! info "Starter Moduleに含まれるもの"
    - 管理グループ構造
    - ポリシー定義・割り当て
    - ネットワーク構成（Hub-and-Spoke or Virtual WAN）
    - 管理リソース（Log Analytics等）

### 利用可能なStarter Module一覧

Terraformでは5つのStarter Moduleから選べます。

=== "basic"

    **特徴**: 最小構成
    
    - 管理グループのみ
    - ポリシーなし
    - ネットワークなし
    
    **用途**: テスト・学習用

=== "hubnetworking"

    **特徴**: Hub-and-Spoke + 管理リソース
    
    - 管理グループ
    - ポリシー
    - Hub VNet
    - Log Analytics
    
    **用途**: 標準的な本番環境（推奨）

=== "complete"

    **特徴**: フル機能
    
    - 管理グループ
    - ポリシー
    - Hub VNet
    - Log Analytics
    - すべてのオプション機能
    
    **用途**: エンタープライズ本番環境

=== "multi-region"

    **特徴**: 複数リージョン対応
    
    - 複数リージョンのHub VNet
    - リージョン間接続
    
    **用途**: グローバル展開

=== "sovereignty"

    **特徴**: 主権クラウド対応
    
    - Government Cloud対応
    - コンプライアンス重視
    
    **用途**: 政府機関・規制業界

!!! success "推奨: hubnetworking"
    初めてのデプロイでは `hubnetworking` がおすすめです。必要な機能がバランスよく含まれています。

### Hub-and-Spoke vs Virtual WAN

ネットワークアーキテクチャの選択です。

=== "Hub-and-Spoke"

    ```text title="Hub-and-Spoke構成"
    Hub VNet
      ├── Firewall
      ├── VPN Gateway
      └── ExpressRoute Gateway
    
    Spoke VNets
      ├── Spoke1 (App)
      ├── Spoke2 (DB)
      └── Spoke3 (Web)
    
    ピアリング接続でHub-Spokeを接続
    ```
    
    **メリット**:
    
    - シンプル
    - コスト効率が良い
    - 小〜中規模に最適
    
    **デメリット**:
    
    - 手動でピアリング設定
    - 大規模だと管理が煩雑

=== "Virtual WAN"

    ```text title="Virtual WAN構成"
    Virtual WAN
      └── Virtual Hub
          ├── VPN Gateway
          ├── ExpressRoute Gateway
          └── Firewall
    
    Spoke VNets
      ├── Spoke1 → 自動接続
      ├── Spoke2 → 自動接続
      └── Spoke3 → 自動接続
    
    Virtual WAN が自動でルーティング
    ```
    
    **メリット**:
    
    - 自動ルーティング
    - 大規模に対応
    - グローバル接続
    
    **デメリット**:
    
    - 高コスト
    - 複雑

!!! tip "選び方"
    - **小〜中規模（〜50 VNets）**: Hub-and-Spoke
    - **大規模（50+ VNets）**: Virtual WAN
    - **グローバル展開**: Virtual WAN
    - **コスト重視**: Hub-and-Spoke

### 要件に応じた選択方法

自社の要件から選択します。

```mermaid
graph TD
    A[要件を確認] --> B{本番環境?}
    B -->|No| C[basic]
    B -->|Yes| D{ネットワーク必要?}
    D -->|No| E[basic]
    D -->|Yes| F{規模は?}
    F -->|小〜中| G[hubnetworking]
    F -->|大| H{複数リージョン?}
    H -->|No| I[complete]
    H -->|Yes| J[multi-region]
```

**質問に答えて選択**:

1. **本番環境ですか？**
   - No → `basic`
   - Yes → 次へ

2. **ネットワークは必要ですか？**
   - No → `basic`
   - Yes → 次へ

3. **何個のVNetを使いますか？**
   - 〜50個 → `hubnetworking`（Hub-and-Spoke）
   - 50個以上 → `complete` or `multi-region`（Virtual WAN）

4. **複数リージョンに展開しますか？**
   - No → `complete`
   - Yes → `multi-region`

!!! example "選択例"
    **ケース1**: 学習用、ネットワーク不要
    → `basic`
    
    **ケース2**: 本番環境、日本のみ、20個のVNet
    → `hubnetworking`（Hub-and-Spoke）
    
    **ケース3**: 本番環境、グローバル展開、100個のVNet
    → `multi-region`（Virtual WAN）

---

## Part 3: Bootstrap環境のセットアップ

### ALZ PowerShell Moduleのインストール

Bootstrapを実行するPowerShellモジュールをインストールします。

```powershell title="ALZ Moduleのインストール"
# PowerShell 7.4+ で実行
Install-PSResource -Name ALZ -Repository PSGallery -TrustRepository
```

**確認**:

```powershell
Get-InstalledPSResource -Name ALZ
```

```text title="出力例"
Name Version Prerelease Repository
---- ------- ---------- ----------
ALZ  0.1.0              PSGallery
```

!!! tip "アップデート方法"
    ```powershell
    # 既にインストール済みの場合
    Update-PSResource -Name ALZ
    ```

### Interactive Modeの実行

対話形式でBootstrap設定を行います。

```powershell title="ALZ Bootstrapの起動"
# PowerShell 7.4+ で実行
Deploy-Accelerator
```

**起動すると質問が始まります**：

```text title="Interactive Mode"
Welcome to the Azure Landing Zone Accelerator!

? Which version control system would you like to use?
  > GitHub
    Azure DevOps
    Local File System

? What is your GitHub organization or user name?
  > shuhei0720org01

? What is your GitHub Personal Access Token?
  > ************************************

? Which IaC tool would you like to use?
  > Terraform
    Bicep

? Which starter module would you like to use?
    basic
  > hubnetworking
    complete
    multi-region
    sovereignty

...（続く）
```

### 各種パラメータの入力

Interactive Modeで入力する主要パラメータです。

=== "VCS設定"

    ```text
    ? Which version control system would you like to use?
    > GitHub
    
    ? What is your GitHub organization or user name?
    > shuhei0720org01
    
    ? What is your GitHub Personal Access Token?
    > ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
    
    ? What would you like to name your repository?
    > alz-mgmt
    
    ? Do you want to create a private repository?
    > Yes
    ```

=== "IaC設定"

    ```text
    ? Which IaC tool would you like to use?
    > Terraform
    
    ? Which starter module would you like to use?
    > hubnetworking
    
    ? What is your preferred Azure region for the management resources?
    > japaneast
    
    ? What is your preferred Azure region for the connectivity resources?
    > japaneast
    ```

=== "Subscription設定"

    ```text
    ? What is your Management subscription ID?
    > 11111111-1111-1111-1111-111111111111
    
    ? What is your Connectivity subscription ID?
    > 22222222-2222-2222-2222-222222222222
    
    ? What is your Identity subscription ID?
    > 33333333-3333-3333-3333-333333333333
    ```

=== "Management Group設定"

    ```text
    ? What is your root management group ID?
    > alz
    
    ? What is your parent management group ID?
    > Tenant Root Group
    
    ? Do you want to create the management group hierarchy?
    > Yes
    ```

=== "承認者設定"

    ```text
    ? Do you want to configure apply approvers?
    > Yes
    
    ? Enter the GitHub usernames of the apply approvers (comma separated):
    > user1,user2
    
    ? Do you want to configure plan reviewers?
    > No
    ```

!!! tip "設定のコツ"
    - **Repository名**: 組織の命名規則に従う
    - **Region**: 日本なら `japaneast` が一般的
    - **Approvers**: 必ず信頼できる人を指定

### Bootstrap実行

すべての質問に答えると、Bootstrapが実行されます。

```text title="Bootstrap実行中"
Starting Bootstrap...

✓ Creating GitHub repository: alz-mgmt
✓ Creating Azure resource group: alz-bootstrap-rg
✓ Creating Storage Account for Terraform state: stoalzmgmt001
✓ Creating Managed Identity: alz-plan-identity
✓ Creating Managed Identity: alz-apply-identity
✓ Creating Federated Credentials
✓ Assigning permissions to Managed Identities
✓ Creating GitHub repository: alz-mgmt-templates
✓ Uploading workflow templates
✓ Creating GitHub environments: alz-mgmt-plan, alz-mgmt-apply
✓ Setting environment variables
✓ Setting branch protection rules

Bootstrap completed successfully!

Repository: https://github.com/shuhei0720org01/alz-mgmt
Templates: https://github.com/shuhei0720org01/alz-mgmt-templates

Next steps:
1. Review the generated code in your repository
2. Make any necessary customizations
3. Trigger the CI/CD pipeline
```

わかりますか？Bootstrapが自動的にすべての設定を行ってくれます。

---

## Part 4: Phase 1デプロイ実行

### State Storage作成

Terraform Stateを保存するAzure Storageが作成されます。

=== "作成されるリソース"

    ```text
    Resource Group: alz-bootstrap-rg
      └── Storage Account: stoalzmgmt001
          └── Container: tfstate
              └── Blob: terraform.tfstate
    ```

=== "確認方法"

    ```bash title="Storage Accountの確認"
    az storage account list \
      --resource-group alz-bootstrap-rg \
      --output table
    ```
    
    ```bash title="Containerの確認"
    az storage container list \
      --account-name stoalzmgmt001 \
      --output table
    ```

!!! info "State Storageの役割"
    - Terraform Stateを安全に保存
    - チームで共有可能
    - ロック機能でコンフリクト防止

### Managed Identityの作成

OIDCで使用するManaged Identityが作成されます。

=== "作成されるIdentity"

    ```text
    Resource Group: alz-identity-rg
      ├── Managed Identity: alz-plan-identity
      └── Managed Identity: alz-apply-identity
    ```
    
    **用途**:
    
    - `alz-plan-identity`: Terraform Plan用（読み取り権限）
    - `alz-apply-identity`: Terraform Apply用（書き込み権限）

=== "確認方法"

    ```bash title="Managed Identityの確認"
    az identity list \
      --resource-group alz-identity-rg \
      --output table
    ```
    
    ```bash title="Client IDの取得"
    az identity show \
      --name alz-plan-identity \
      --resource-group alz-identity-rg \
      --query clientId -o tsv
    ```

### Federated Credentialの設定

GitHub ActionsからAzureへの認証設定が作成されます。

=== "Plan用Federated Credential"

    ```bash title="設定内容"
    Name: github-actions-plan
    Identity: alz-plan-identity
    Issuer: https://token.actions.githubusercontent.com
    Subject: repo:shuhei0720org01/alz-mgmt:environment:alz-mgmt-plan
    Audiences: api://AzureADTokenExchange
    ```

=== "Apply用Federated Credential"

    ```bash title="設定内容"
    Name: github-actions-apply
    Identity: alz-apply-identity
    Issuer: https://token.actions.githubusercontent.com
    Subject: repo:shuhei0720org01/alz-mgmt:environment:alz-mgmt-apply
    Audiences: api://AzureADTokenExchange
    ```

=== "確認方法"

    ```bash title="Federated Credentialの確認"
    az identity federated-credential list \
      --identity-name alz-plan-identity \
      --resource-group alz-identity-rg \
      --output table
    ```

!!! success "OIDCのメリット（再確認）"
    - パスワード不要
    - 自動トークン発行
    - 短期間で期限切れ（安全）

### GitHub Repository/Actions作成

2つのリポジトリが作成されます。

=== "alz-mgmt（メイン）"

    **内容**:
    
    ```text
    alz-mgmt/
    ├── .github/
    │   └── workflows/
    │       ├── ci.yaml
    │       └── cd.yaml
    ├── main.tf
    ├── variables.tf
    ├── terraform.tfvars
    └── lib/
        ├── archetype_definitions/
        └── architecture_definitions/
    ```
    
    - Terraformコード
    - tfvars設定ファイル
    - CI/CD workflow呼び出し

=== "alz-mgmt-templates（テンプレート）"

    **内容**:
    
    ```text
    alz-mgmt-templates/
    └── .github/
        └── workflows/
            ├── ci-template.yaml
            └── cd-template.yaml
    ```
    
    - 再利用可能ワークフロー
    - CI/CDロジック

=== "GitHub Environments"

    **alz-mgmt-plan**:
    
    - Protection rules: なし（自動実行）
    - Variables:
        - `AZURE_CLIENT_ID`: Plan Identity
        - `AZURE_TENANT_ID`: Tenant ID
        - `AZURE_SUBSCRIPTION_ID`: Management Subscription
        - Backend設定
    
    **alz-mgmt-apply**:
    
    - Protection rules: Required reviewers（承認必須）
    - Variables:
        - `AZURE_CLIENT_ID`: Apply Identity
        - `AZURE_TENANT_ID`: Tenant ID
        - `AZURE_SUBSCRIPTION_ID`: Management Subscription
        - Backend設定

### 検証手順

Bootstrap完了後、正しく設定されているか確認します。

=== "リポジトリの確認"

    ```bash title="ブラウザで確認"
    # メインリポジトリ
    https://github.com/<org>/alz-mgmt
    
    # テンプレートリポジトリ
    https://github.com/<org>/alz-mgmt-templates
    ```
    
    **確認項目**:
    
    - ✅ Terraformコードがある
    - ✅ .github/workflows/ci.yaml がある
    - ✅ .github/workflows/cd.yaml がある

=== "Managed Identityの確認"

    ```bash title="権限の確認"
    # Plan Identityの権限（Reader）
    az role assignment list \
      --assignee <plan-identity-client-id> \
      --all
    
    # Apply Identityの権限（Contributor）
    az role assignment list \
      --assignee <apply-identity-client-id> \
      --all
    ```

=== "GitHub Environments確認"

    ```text
    GitHub → Settings → Environments
    
    ✓ alz-mgmt-plan
      - Protection rules: なし
      - Variables: 6個
    
    ✓ alz-mgmt-apply
      - Protection rules: Required reviewers
      - Variables: 6個
    ```

=== "Workflow確認"

    ```bash title="ローカルにclone"
    git clone https://github.com/<org>/alz-mgmt.git
    cd alz-mgmt
    
    # ファイル構成確認
    tree -L 2
    ```
    
    ```text title="期待される構成"
    alz-mgmt/
    ├── .github/
    │   └── workflows/
    ├── main.tf
    ├── variables.tf
    ├── terraform.tfvars
    └── lib/
    ```

!!! success "Phase 1完了チェックリスト"
    - ✅ State Storage作成済み
    - ✅ Managed Identity作成済み
    - ✅ Federated Credential設定済み
    - ✅ GitHub Repository作成済み
    - ✅ GitHub Environments設定済み
    - ✅ Workflow作成済み

---

## まとめ

この章で学んだこと：

### ✅ Part 1: 前提条件の準備

- PowerShell 7.4+、Azure CLI 2.55+、Gitのインストール
- 4つのAzure Subscriptionの準備
- Owner権限の確認
- Version Control Systemの選択（GitHub推奨）

### ✅ Part 2: Starter Moduleの選択

- Starter Moduleとは
- 5種類のStarter Module（basic、hubnetworking、complete、multi-region、sovereignty）
- Hub-and-Spoke vs Virtual WAN
- 要件に応じた選択方法

### ✅ Part 3: Bootstrap環境のセットアップ

- ALZ PowerShell Moduleのインストール
- Interactive Modeの実行
- 各種パラメータの入力
- Bootstrap実行

### ✅ Part 4: Phase 1デプロイ実行

- State Storage作成
- Managed Identity作成
- Federated Credential設定
- GitHub Repository/Actions作成
- 検証手順

次の章では、実際にLanding Zonesをデプロイします（Phase 2）。

## 練習問題

理解度チェックです。休憩中に考えてみましょう。

### 問題1
Bootstrap Phase 1で作成するリソースは何ですか？（3つ答えてください）

### 問題2
Terraform StateをAzure Storageに保存する理由は何ですか？

### 問題3
Federated Credentialで設定する`subject`の値は何を表していますか？

---

## 練習問題の答え

### 答え1
1. **Storage Account**（Terraform State保存用）
2. **User Assigned Managed Identity**（GitHub Actions用の認証ID）
3. **Federated Credential**（OIDC認証用の信頼関係）

これらがBootstrap環境の基盤になります。

### 答え2
**複数人でTerraformを実行してもState情報を共有できるから**です。

ローカルState（❌）:
```
開発者A: terraform apply → ローカルにstate保存
開発者B: terraform apply → 開発者Aの変更が見えない！
→ リソースが重複作成される
```

リモートState（✅）:
```
開発者A: terraform apply → Azure Storageにstate保存
開発者B: terraform apply → Azure Storageからstate取得
→ 開発者Aの変更が見える
```

また、**State Lockingにより同時実行を防止**できます。

### 答え3
**どのGitHubリポジトリ・ブランチからの認証を許可するか**を表しています。

```hcl
subject = "repo:myorg/alz-mgmt:ref:refs/heads/main"
```

構造:
```
repo:{organization}/{repository}:ref:refs/heads/{branch}
```

例:
- `repo:myorg/alz-mgmt:ref:refs/heads/main` → mainブランチからのみ許可
- `repo:myorg/alz-mgmt:ref:refs/heads/develop` → developブランチからのみ許可
- `repo:myorg/alz-mgmt:pull_request` → Pull Requestからも許可

これにより、**特定のリポジトリ・ブランチからのみAzure認証を許可**できます。

!!! tip "次の章へ"
    [Chapter 15: Bootstrap Phase 2](15_Bootstrap_Phase_2.md)で、Landing Zonesのデプロイと検証を学びます。

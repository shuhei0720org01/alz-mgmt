# 10. Hub-and-Spoke - ネットワークの中核

## 8つの設計領域との対応

このChapterは以下の設計領域を実装します：

- ✅ **4. Network Topology and Connectivity**（ネットワーク構成）
- ✅ **5. Security**（セキュリティ - Firewallによる通信制御）

→ Chapter 01で学んだ8つの設計領域を復習したい方は[01_基礎知識.md](./01_基礎知識.md)へ

---

## ネットワークアーキテクチャの選択

このプロジェクトでは**Hub-and-Spoke**を採用していますが、もう1つの選択肢として**Virtual WAN**もあります。

### 2つのアーキテクチャ比較

| 観点 | Hub-and-Spoke | Virtual WAN |
|------|--------------|-------------|
| **推奨規模** | 中小企業（〜1000人） | 大企業（1000人以上） |
| **リージョン数** | 1〜3リージョン | 4リージョン以上 |
| **初期コスト** | 低（¥50万/月〜） | 高（¥100万/月〜） |
| **管理の柔軟性** | 高（全て自分で設定） | 低（Microsoftが自動管理） |
| **学習難易度** | 易（VNetの知識があればOK） | 難（Virtual WAN固有の概念） |
| **グローバル接続** | 手動設定が必要 | 自動で最適化 |
| **実装時間** | 2〜4時間 | 1〜2時間 |
| **トラブルシューティング** | 自分で全部確認 | Microsoftが大部分を管理 |

### どっちを選ぶべき？

**Hub-and-Spokeを選ぶべきケース：**
- ✅ 最初のLanding Zones構築（学習しやすい）
- ✅ 日本国内のみ、またはアジア圏のみ
- ✅ 細かくカスタマイズしたい
- ✅ コストを抑えたい
- ✅ ネットワークの仕組みを理解したい

**Virtual WANを選ぶべきケース：**
- ✅ グローバル展開（アメリカ、ヨーロッパ、アジア全て）
- ✅ 複数のオンプレミス拠点がある
- ✅ SD-WANを使っている
- ✅ Microsoftに任せて運用を楽にしたい
- ✅ 既にVirtual WANの経験がある

**このプロジェクトの選択：**  
このプロジェクトはHub-and-Spokeを採用しています。理由は：
1. 初心者にも理解しやすい
2. カスタマイズの自由度が高い
3. コストが予測しやすい
4. 日本の中小企業に最適

Virtual WANについては[Chapter 11](./11_Virtual_WAN.md)で詳しく解説します。

---

## このChapterでやること

Hub-and-Spokeネットワークの仕組みを理解しよう。

**Hub-and-Spokeって何？**
中央に1つのHub（ハブ）VNet、周りに複数のSpoke（スポーク）VNetを配置するネットワーク構成。

**例えるなら**：

- **Hub**：空港のハブ（成田、羽田）
- **Spoke**：地方空港
- 地方から海外行くには、一度ハブを経由する

**ネットワークも同じ**：

- **Hub**：中央のVNet（Firewall、VPN Gatewayがある）
- **Spoke**：アプリケーション用VNet
- Spokeからインターネットやオンプレに行くには、Hubを経由

**📊 Hub-and-Spoke構成図（このプロジェクト）**

```
                    インターネット
                         ↕
                   Azure Firewall
                         |
         ┌───────────────┼───────────────┐
         │         Hub VNet              │
         │    (10.0.0.0/16)             │
         │                               │
         │  ┌─────────┐  ┌──────────┐   │
         │  │Firewall │  │  Bastion │   │
         │  └─────────┘  └──────────┘   │
         │                               │
         │  ┌──────────┐  ┌───────────┐ │
         │  │VPN GW    │  │ER Gateway │ │
         │  └──────────┘  └───────────┘ │
         │       ↓             ↓         │
         └───────┼─────────────┼─────────┘
                 ↓             ↓
         オンプレミス      ExpressRoute
           (VPN)           (専用線)
                 
      VNet Peering ↓      ↓      ↓
         
    ┌──────────┐  ┌──────────┐  ┌──────────┐
    │ Spoke 1  │  │ Spoke 2  │  │ Spoke 3  │
    │ (Corp)   │  │ (Online) │  │  (Dev)   │
    │10.1.0.0  │  │10.2.0.0  │  │10.3.0.0  │
    └──────────┘  └──────────┘  └──────────┘
        Web          API         Database
       Server      Server        Server
```

**🎯 通信の流れ（例）**

```
Spoke 1 (Webサーバー) → インターネット へアクセスしたい
  ↓
1. User Defined Route (UDR) で Hub へ
  ↓
2. Hub の Firewall でセキュリティチェック
  ↓
3. 許可されたらインターネットへ
  ↓
4. 戻りも Firewall 経由
  ↓
5. Spoke 1 へ到着
```

**🔐 このアーキテクチャの特徴**

| 要素 | 配置場所 | 役割 | 共有/個別 |
|------|----------|------|-----------|
| **Firewall** | Hub | 全トラフィックの検査 | 全Spokeで共有 |
| **Bastion** | Hub | VM への安全なログイン | 全Spokeで共有 |
| **VPN Gateway** | Hub | オンプレミス接続 | 全Spokeで共有 |
| **ExpressRoute GW** | Hub | 専用線接続 | 全Spokeで共有 |
| **アプリケーション** | Spoke | 実際のワークロード | 個別 |

---

## Hub-and-Spokeのメリット

### 1. セキュリティ集約

```
全トラフィックがHubを経由
  ↓
Firewallで一元管理
  ↓
「Spokeごとにファイアウォール買う」不要
```

### 2. コスト削減

```
VPN Gateway：1台約4万円/月
  ↓
Hubに1台だけ置く
  ↓
全Spokeで共有
  ↓
Spokeごとに買わなくていい
```

### 3. 管理が楽

```
DNSサーバー：Hubに1台
Firewallルール：Hubで一元管理
VPN接続：Hubだけ設定
```

### 4. 柔軟性

```
新しいアプリ追加
  ↓
新しいSpoke VNet作る
  ↓
Hubにピアリング
  ↓
すぐ使える
```

---

## Part 1: 設定ファイル（tfvars）

### connectivity_type

```hcl
connectivity_type = "hub_and_spoke_vnet"
```

**何？**：ネットワークの種類

- `hub_and_spoke_vnet`：Hub-and-Spoke構成
- `virtual_wan`：Virtual WAN構成（Chapter 11で解説）
- `none`：ネットワーク作らない

### hub_and_spoke_networks_settings

```hcl
hub_and_spoke_networks_settings = {
  enabled_resources = {
    ddos_protection_plan = true
  }
  ddos_protection_plan = {
    name                = "ddos-alz"
    resource_group_name = "rg-ddos"
    location            = "japaneast"
  }
}
```

#### ddos_protection_plan

**DDoS Protectionって何？**
DDoS攻撃（大量アクセスでサーバーダウンさせる攻撃）から守る仕組み。

**料金**：
```
Standard：約40万円/月（高い！）
Basic：無料（自動で有効）
```

**注意**：
```hcl
# 開発環境ではfalseにしとこう
enabled_resources = {
  ddos_protection_plan = false  # ←コスト削減
}
```

Chapter 3で見た設定ですね。

### hub_virtual_networks

```hcl
hub_virtual_networks = {
  primary = {
    location          = "japaneast"
    default_parent_id = "/subscriptions/.../resourceGroups/rg-jpe-connectivity"
    
    enabled_resources = {
      firewall                              = true
      bastion                               = true
      virtual_network_gateway_express_route = false
      virtual_network_gateway_vpn           = true
      private_dns_zones                     = true
      private_dns_resolver                  = false
    }
    
    hub_virtual_network = {
      name          = "vnet-jpe-hub"
      address_space = ["10.0.0.0/16"]
    }
    
    firewall = {
      subnet_address_prefix = "10.0.0.0/26"
      name                  = "fw-jpe-hub"
    }
    
    ...
  }
}
```

**primary**ってキーは何でもいい。複数のHubを作る時に識別するため。

---

## Part 2: Hub VNetの構成要素

### 1. Hub Virtual Network（VNet）

```hcl
hub_virtual_network = {
  name          = "vnet-jpe-hub"
  address_space = ["10.0.0.0/16"]
}
```

**何？**：中央のVNet

**address_space**：
```
10.0.0.0/16
  ↓
10.0.0.0 〜 10.0.255.255
  ↓
約65,000個のIPアドレス
```

**サブネット分割**：
```
10.0.0.0/26   → Firewall（64個）
10.0.0.64/26  → Firewall Management（64個）
10.0.1.0/26   → Gateway（64個）
10.0.2.0/27   → Bastion（32個）
10.0.3.0/24   → アプリ用（256個）
...
```

### 2. Azure Firewall

```hcl
firewall = {
  subnet_address_prefix            = "10.0.0.0/26"
  management_subnet_address_prefix = "10.0.0.64/26"
  name                             = "fw-jpe-hub"
  
  default_ip_configuration = {
    public_ip_config = {
      name = "pip-fw-jpe-hub"
    }
  }
  
  management_ip_enabled = true
  management_ip_configuration = {
    public_ip_config = {
      name = "pip-fw-jpe-hub-mgmt"
    }
  }
}
```

#### Azure Firewallって何？

**次世代ファイアウォール**：

- ネットワークレベル（L3/L4）のフィルタリング
- アプリケーションレベル（L7）のフィルタリング
- 脅威インテリジェンス
- DNS Proxy

**料金**：
```
Basic：約8万円/月（最小構成）
Standard：約17万円/月
Premium：約25万円/月
```

**めっちゃ高い！**

#### 2つのサブネット

##### AzureFirewallSubnet（データプレーン）
```hcl
subnet_address_prefix = "10.0.0.0/26"
```

**何をしているのでしょうか？**：実際のトラフィック処理

```
Spoke VNet → Hub Firewall → インターネット
オンプレ → Hub Firewall → Spoke VNet
```

##### AzureFirewallManagementSubnet（管理プレーン）
```hcl
management_subnet_address_prefix = "10.0.0.64/26"
```

**何をしているのでしょうか？**：Firewallの管理・設定

**なんで分ける？**
- データプレーンがビジー状態でも管理できる
- セキュリティ分離（管理操作とトラフィック処理を分離）

#### management_ip_enabled

```hcl
management_ip_enabled = true
```

**何？**：管理用Public IPを有効化

**true**：
```
Public IP 2個必要
- データプレーン用
- 管理プレーン用
→ コスト高い
```

**false**：
```
Public IP 1個だけ
→ コスト削減
→ でも管理機能が制限される
```

**おすすめ**：
```hcl
# 本番環境
management_ip_enabled = true  # ←安定性重視

# 開発環境
management_ip_enabled = false  # ←コスト重視
```

#### firewall_policy

```hcl
firewall_policy = {
  name = "fwp-jpe-hub"
}
```

**何？**：Firewallのルール集

**ポリシーの内容**：

- ネットワークルール（IPアドレス、ポート）
- アプリケーションルール（FQDN、URL）
- NATルール（ポート転送）

**例**：
```
ネットワークルール：

- Spoke VNetから80/443ポートを許可

アプリケーションルール：

- *.microsoft.comへのアクセスを許可
- *.google.comへのアクセスを拒否

NATルール：

- Public IP:8080 → VM:80にポート転送
```

### 3. VPN Gateway

```hcl
virtual_network_gateways = {
  subnet_address_prefix = "10.0.1.0/26"
  
  vpn = {
    name = "vpngw-jpe-hub"
    
    ip_configurations = {
      default = {
        public_ip = {
          name = "pip-vpngw-jpe-hub"
        }
      }
      second = {
        public_ip = {
          name = "pip-vpngw-jpe-hub-second"
        }
      }
    }
  }
}
```

#### VPN Gatewayって何？

**オンプレとAzureを繋ぐゲートウェイ**：
```
オンプレのオフィス
  ↓ VPN接続（暗号化）
Azure VPN Gateway
  ↓
Hub VNet
  ↓ ピアリング
Spoke VNet
```

#### Active-Active構成

```hcl
ip_configurations = {
  default = { ... }  # ←1個目のPublic IP
  second = { ... }   # ←2個目のPublic IP
}
```

**何？**：冗長化構成

```
2つのPublic IP
  ↓
2つのVPNトンネル
  ↓
片方が落ちても大丈夫
```

**料金**：
```
VpnGw1：約4万円/月
VpnGw2：約9万円/月
VpnGw3：約20万円/月
```

**スループット**：
```
VpnGw1：650 Mbps
VpnGw2：1 Gbps
VpnGw3：1.25 Gbps
```

#### GatewaySubnet

```hcl
subnet_address_prefix = "10.0.1.0/26"
```

**サブネット名は固定**：`GatewaySubnet`

変えちゃダメ。Azureが自動で認識する。

### 4. ExpressRoute Gateway

```hcl
express_route = {
  name = "ergw-jpe-hub"
  ip_configurations = {
    default = {
      public_ip = {
        name = "pip-ergw-jpe-hub"
      }
    }
  }
}
```

#### ExpressRouteって何？

**専用線接続**：
```
オンプレ
  ↓ 物理専用線（通信キャリア経由）
Azure ExpressRoute Gateway
  ↓
Hub VNet
```

**VPNとの違い**：
```
VPN：

- インターネット経由
- 暗号化あり
- 安い
- 速度遅い（最大1.25Gbps）

ExpressRoute：

- 専用線
- より高セキュリティ
- 超高い（数十万円/月〜）
- 速い（最大100Gbps）
```

**普通はVPNでOK**。

### 5. Azure Bastion

```hcl
bastion = {
  subnet_address_prefix = "10.0.2.0/27"
  name                  = "bas-jpe-hub"
  
  public_ip = {
    name  = "pip-bas-jpe-hub"
    zones = []  # ←Japan regionは空リスト
  }
}
```

#### Azure Bastionって何？

**安全なRDP/SSH接続**：
```
従来：
PC → Public IP → VM
  ↓
問題：

- VMにPublic IP必要
- セキュリティリスク
- 攻撃されやすい

Bastion：
PC → Azureポータル → Bastion → VM（Private IPだけ）
  ↓
メリット：

- VMにPublic IP不要
- セキュア
- NSGで保護
```

#### zones = []

Chapter 3で見た重要な設定：

```hcl
zones = []  # ←Japan regionはAvailability Zones非対応
```

**Japan East/Westの場合は必須**：
```hcl
# OK
zones = []

# NG（エラーになる）
zones = ["1", "2", "3"]
```

**料金**：
```
Basic：約2.7万円/月
Standard：約21万円/月
```

#### AzureBastionSubnet

```hcl
subnet_address_prefix = "10.0.2.0/27"
```

**サブネット名は固定**：`AzureBastionSubnet`

これも変えちゃダメ。

### 6. Private DNS Zones

```hcl
enabled_resources = {
  private_dns_zones = true
}
```

**何？**：プライベートDNS

**使い道**：
```
Azure PaaS（Storage、SQL）をプライベート接続
  ↓
Private Endpoint作成
  ↓
Private DNS Zoneで名前解決
  ↓
storage.blob.core.windows.net → 10.0.3.4（Private IP）
```

**自動作成されるゾーン**：
```
privatelink.blob.core.windows.net
privatelink.database.windows.net
privatelink.azurewebsites.net
...
```

約50個のゾーンが自動作成される。

---

## Part 3: マルチリージョン構成

### primary / secondary

```hcl
hub_virtual_networks = {
  primary = {
    location = "japaneast"
    ...
  }
  secondary = {
    location = "japanwest"
    ...
  }
}
```

**何をしているのでしょうか？**：2つのHubを作ります

**構成**：
```
Japan East Hub
  ├── Firewall
  ├── VPN Gateway
  └── Bastion

Japan West Hub
  ├── Firewall
  ├── VPN Gateway
  └── Bastion

2つのHubをVNetピアリングで接続
```

**メリット**：

- 冗長化（片方が落ちても大丈夫）
- 地理的分散（災害対策）
- レイテンシ削減（近いHubを使う）

**デメリット**：

- コスト2倍
- 管理が複雑

**おすすめ**：
```hcl
# 開発環境：primaryだけ
hub_virtual_networks = {
  primary = { ... }
  # secondaryはコメントアウト
}

# 本番環境：両方
hub_virtual_networks = {
  primary = { ... }
  secondary = { ... }
}
```

---

## Part 4: ルーティング

### User Defined Route（UDR）

```hcl
hub_virtual_network = {
  route_table_name_firewall     = "rt-fw-jpe"
  route_table_name_user_subnets = "rt-user-jpe"
}
```

**何？**：カスタムルート

#### route_table_name_firewall

```
Firewallサブネット用のルートテーブル
```

**ルール例**：
```
送信先：0.0.0.0/0（インターネット）
次ホップ：Internet
  ↓
Firewallから直接インターネットに出る
```

#### route_table_name_user_subnets

```
アプリ用サブネット（Spoke）用のルートテーブル
```

**ルール例**：
```
送信先：0.0.0.0/0（インターネット）
次ホップ：10.0.0.4（Firewallの内部IP）
  ↓
全トラフィックをFirewall経由にする
```

**効果**：
```
Spoke VM → インターネット
  ↓
1. VMから出る（送信先：8.8.8.8）
2. UDRでFirewallに転送
3. Firewallで検査
4. Firewallから出る
5. インターネットに到達
```

---

## 実践：コスト削減版を作ってみよう

### 最小構成

```hcl
hub_virtual_networks = {
  primary = {
    location = "japaneast"
    
    enabled_resources = {
      firewall                              = false  # ←無効化
      bastion                               = false  # ←無効化
      virtual_network_gateway_express_route = false
      virtual_network_gateway_vpn           = false  # ←無効化
      private_dns_zones                     = true
      private_dns_resolver                  = false
    }
    
    hub_virtual_network = {
      name          = "vnet-jpe-hub"
      address_space = ["10.0.0.0/16"]
    }
  }
}

hub_and_spoke_networks_settings = {
  enabled_resources = {
    ddos_protection_plan = false  # ←無効化
  }
}
```

**コスト**：
```
通常構成：
Firewall：17万円/月
VPN Gateway：4万円/月
Bastion：2.7万円/月
DDoS：40万円/月
-----------------
合計：約64万円/月

最小構成：
VNet：無料
Private DNS：無料
-----------------
合計：約0円/月（ほぼ無料）
```

**デメリット**：

- インターネット接続なし
- オンプレ接続なし
- RDP/SSH接続不便

**使い道**：

- 開発環境の初期構築
- ネットワーク設計の検証
- コスト削減が最優先

---

## デバッグ技

### VNetの確認

```bash
# VNet一覧
az network vnet list --output table

# 特定のVNetの詳細
az network vnet show \
  --resource-group rg-jpe-connectivity \
  --name vnet-jpe-hub
```

### Firewallの確認

```bash
# Firewall一覧
az network firewall list --output table

# Firewallのログ確認
az monitor log-analytics query \
  --workspace <workspace-id> \
  --analytics-query "AzureDiagnostics | where Category == 'AzureFirewallApplicationRule' | take 10"
```

### ルートテーブルの確認

```bash
# ルートテーブル一覧
az network route-table list --output table

# 特定のルートテーブルの詳細
az network route-table show \
  --resource-group rg-jpe-connectivity \
  --name rt-user-jpe
```

### Bastionの接続

Azureポータル：
```
VM → Connect → Bastion
  ↓
ユーザー名とパスワード入力
  ↓
ブラウザでRDP/SSH接続
```

---

## よくあるエラー

### エラー1: サブネット名が違う

```
Error: subnet name must be 'AzureFirewallSubnet'
```

**原因**：Firewallのサブネット名が間違ってる

**対処法**：
サブネット名は自動生成されるから、手動で作らない。

### エラー2: zonesエラー（Japan region）

```
Error: availability zones are not supported in this region
```

**原因**：Japan East/Westで`zones`を指定してる

**対処法**：
```hcl
zones = []  # ←空リスト
```

Chapter 3で詳しく解説したやつ。

### エラー3: アドレス空間の重複

```
Error: address space overlaps with existing VNet
```

**原因**：VNetのアドレス空間が他と重複

**対処法**：
```hcl
# NG
primary:   10.0.0.0/16
secondary: 10.0.0.0/16  # ←重複

# OK
primary:   10.0.0.0/16
secondary: 10.1.0.0/16  # ←分ける
```

### エラー4: Firewallがタイムアウト

```
Error: timeout waiting for firewall to be ready
```

**原因**：Firewallの作成に時間がかかる（30分〜1時間）

**対処法**：
```bash
# もう一度apply
terraform apply

# または待つ
```

---

## まとめ

**Hub-and-Spokeの構成要素**：
1. **Hub VNet**：中央のVNet
2. **Firewall**：トラフィック制御
3. **VPN Gateway**：オンプレ接続
4. **ExpressRoute Gateway**：専用線接続
5. **Bastion**：安全なRDP/SSH
6. **Private DNS**：プライベート名前解決

**覚えておくこと**：

- Hub経由で全トラフィック制御
- Firewallでセキュリティ集約
- VPN Gatewayでオンプレ接続
- Bastionで安全なVM接続
- Japan regionは`zones = []`必須
- コスト削減：不要なリソースは無効化

次のChapterでは、Virtual WAN構成を見ていきます。
Hub-and-Spokeのマネージド版で、より大規模な環境向けです。

---

**所要時間**: 50分  
**難易度**: ★★★★★  
**前**: [09_管理リソース.md](./09_管理リソース.md)  
**次**: [11_Virtual_WAN.md](./11_Virtual_WAN.md)

# 11. Virtual WAN - 次世代のグローバルネットワーク

## 8つの設計領域との対応

このChapterは以下の設計領域を実装します：

- ✅ **4. Network Topology and Connectivity**（ネットワーク構成 - 高度な実装）
- ✅ **8. Platform Automation and DevOps**（プラットフォーム自動化）

→ Chapter 01で学んだ8つの設計領域を復習したい方は[01_基礎知識.md](./01_基礎知識.md)へ

---

## Hub-and-Spokeから来た方へ

[Chapter 10](./10_Hub-and-Spoke.md)でHub-and-Spokeネットワークを学びましたよね。  
Virtual WANはその「マネージド版」です。比較の詳細はChapter 10で解説しているので、ここではVirtual WANの実装に集中します。

**このChapterを読むべき人：**
- ✅ グローバル展開を考えている
- ✅ Virtual WANに興味がある
- ✅ Hub-and-Spokeとの違いを知りたい
- ✅ 将来の移行を検討している

**スキップしてもOKな人：**
- 日本国内のみの展開
- 当面Hub-and-Spokeで十分
- すぐにデプロイしたい → [Chapter 13](./13_デプロイ手順.md)へ

---

## このChapterでやること

Virtual WANを理解しよう。

**Virtual WANって何？**
Microsoftがマネージドで提供するグローバルネットワークサービス。

**Hub-and-Spokeとの違い**：
```text
Hub-and-Spoke：

- 自分でVNet作る
- 自分でFirewall設定
- 自分でルーティング設定
- 小〜中規模向け

Virtual WAN：

- Microsoftが管理
- 設定がシンプル
- グローバル規模
- 大規模向け
```text

**例えるなら**：

- **Hub-and-Spoke**：自家用車（自分で運転）
- **Virtual WAN**：タクシー（プロに任せる）

---

## Virtual WANのメリット

### 1. マネージド

```text
Hub-and-Spoke：
VNet作成 → サブネット設計 → Firewall設定 → ルーティング設定
  ↓
全部自分でやる

Virtual WAN：
Virtual Hub作成 → VNet接続
  ↓
ルーティングは自動
```text

### 2. グローバルトランジット

**📊 Virtual WANのグローバル接続図**

```text
                  インターネット
                       ↕
            ┌─────────────────────┐
            │  Microsoft Backbone  │
            │   (自動で最適化)      │
            └─────────────────────┘
                       ↕
    ┌──────────────┬─────────────┬──────────────┐
    │              │             │              │
┌───┴────┐    ┌───┴────┐   ┌───┴────┐    ┌───┴────┐
│Tokyo   │    │Singapore│   │London  │    │ New York│
│vHub    │────│vHub     │───│vHub    │────│vHub    │
│(東日本) │    │(東南ア) │   │(欧州)  │    │(米国)  │
└───┬────┘    └───┬────┘   └───┬────┘    └───┬────┘
    │              │             │              │
    ↓              ↓             ↓              ↓
┌───────┐      ┌───────┐     ┌───────┐     ┌───────┐
│Spoke  │      │Spoke  │     │Spoke  │     │Spoke  │
│VNet   │      │VNet   │     │VNet   │     │VNet   │
│(日本  │      │(ASEAN │     │(欧州  │     │(北米  │
│ アプリ)│      │ アプリ)│     │ アプリ)│     │ アプリ)│
└───────┘      └───────┘     └───────┘     └───────┘

【特徴】
✅ 全vHub間が自動でメッシュ接続
✅ Microsoft Backboneで最適化された通信
✅ ルーティングは自動設定
✅ どのリージョンからでも相互通信可能
```text

**🔄 通信例：東京のアプリ → ロンドンのDB**

```text
Tokyo Spoke VNet (アプリ)
  ↓
Tokyo vHub
  ↓
Microsoft Backbone (最適経路を自動選択)
  ↓
London vHub
  ↓
London Spoke VNet (DB)

【所要時間】約150ms（自動最適化）
```text

**Hub-and-Spokeだと**：
```text
VNetピアリングを手動で設定
  ↓
めちゃくちゃ複雑
```text

### 3. スケーラビリティ

```text
Hub-and-Spoke：
VPN Gateway：最大30接続

Virtual WAN：
VPN Gateway：最大10,000接続
  ↓
超大規模対応
```text

### 4. 統合された管理

```text
Virtual WAN Hub内で全部管理：

- VPN接続
- ExpressRoute接続
- Firewall
- ルーティング
- SD-WAN統合
```text

---

## デメリット

### 1. コストが高い

```text
Virtual Hub：約5.5万円/月
+ データ転送料金
+ 接続料金

Hub-and-Spokeより高い
```text

### 2. カスタマイズに制限

```text
Hub-and-Spoke：

- 細かい設定変更可能
- 自由度高い

Virtual WAN：

- マネージドなので制限あり
- Microsoftの設計に従う
```text

### 3. 学習コスト

```text
新しい概念：

- Virtual Hub
- Virtual Network Connection
- Routing Intent
- Route Table（Hub用）

慣れるまで大変
```text

---

## Part 1: 設定ファイル（tfvars）

### connectivity_type

```hcl
connectivity_type = "virtual_wan"
```text

**Chapter 10との違い**：
```hcl
# Hub-and-Spoke
connectivity_type = "hub_and_spoke_vnet"

# Virtual WAN
connectivity_type = "virtual_wan"
```text

どっちか片方だけ使う。両方は使えない。

### virtual_wan_settings

```hcl
virtual_wan_settings = {
  enabled_resources = {
    resource_group = true
  }
  resource_group = {
    name     = "rg-vwan"
    location = "japaneast"
  }
  virtual_wan = {
    name = "vwan-alz"
  }
}
```text

#### resource_group

**何？**：Virtual WAN用のリソースグループ

**独立している理由**：
```text
Virtual WANはグローバルリソース
  ↓
特定リージョンに縛られない
  ↓
専用のリソースグループに入れる
```text

#### virtual_wan

```hcl
virtual_wan = {
  name = "vwan-alz"
}
```text

**Virtual WANリソース**：最上位のコンテナ

**階層構造**：
```text
Virtual WAN（最上位）
  └── Virtual Hub（リージョンごと）
       ├── VPN Gateway
       ├── ExpressRoute Gateway
       ├── Firewall
       └── VNet接続
```text

---

## Part 2: Virtual Hub

### virtual_hubs

```hcl
virtual_hubs = {
  primary = {
    location = "japaneast"
    
    enabled_resources = {
      firewall                              = true
      bastion                               = false
      virtual_network_gateway_express_route = false
      virtual_network_gateway_vpn           = true
      private_dns_zones                     = true
      private_dns_resolver                  = false
      sidecar_virtual_network               = false
    }
    
    default_hub_address_space = "10.0.0.0/16"
    default_parent_id         = "/subscriptions/.../resourceGroups/rg-vwan"
    
    hub = {
      name                                   = "vhub-jpe"
      address_prefix                         = "10.0.0.0/23"
      hub_routing_preference                 = "ExpressRoute"
      virtual_router_auto_scale_min_capacity = 2
    }
    
    ...
  }
}
```text

**primaryってキー**：何でもいい。複数のHubを識別するため。

### hub.address_prefix

```hcl
address_prefix = "10.0.0.0/23"
```text

**何？**：Virtual Hub自体のアドレス空間

**サイズ**：
```text
/23 → 512個のIPアドレス
/24 → 256個
/22 → 1024個
```text

**推奨**：`/23`（512個）

**使い道**：
```text
Hub内部の仮想ルーター用
  ↓
ユーザーは使わない
  ↓
Microsoftが管理
```text

### hub_routing_preference

```hcl
hub_routing_preference = "ExpressRoute"
```text

**何？**：ルーティング優先順位

**選択肢**：
```text
ExpressRoute：

- ExpressRoute経由を優先
- VPNはバックアップ

VpnGateway：

- VPN経由を優先
- ExpressRouteはバックアップ

ASPath：

- BGP AS Pathで判断
- 細かい制御
```text

**普通は**：`ExpressRoute`

### virtual_router_auto_scale_min_capacity

```hcl
virtual_router_auto_scale_min_capacity = 2
```text

**何？**：Virtual Hubのスケールユニット

**スケールユニット**：
```text
1ユニット = 約500Mbps
2ユニット = 約1Gbps
3ユニット = 約1.5Gbps
...
```text

**料金**：
```text
1ユニット：約2.7万円/月
2ユニット：約5.5万円/月
3ユニット：約8.2万円/月
```text

**最小値**：2（推奨）

---

## Part 3: Firewall（Virtual WAN版）

### firewall

```hcl
firewall = {
  name                         = "fw-vhub-jpe"
  threat_intelligence_mode     = "Alert"
  virtual_hub_key              = "primary"
  dns_servers                  = []
  public_ip_count              = 1
  sku_tier                     = "Standard"
  threat_intelligence_allowlist = {}
}
```text

#### Hub-and-SpokeのFirewallとの違い

```text
Hub-and-Spoke：

- 自分でサブネット作る
- 自分でPublic IP管理
- VNetにデプロイ

Virtual WAN：

- サブネット不要
- Public IP自動作成
- Virtual Hubにデプロイ
- 設定がシンプル
```text

#### threat_intelligence_mode

```hcl
threat_intelligence_mode = "Alert"
```text

**何？**：脅威インテリジェンス

**選択肢**：
```text
Off：

- 無効
- 脅威検知しない

Alert：

- 警告だけ
- ブロックしない
- ログに記録

Deny：

- 警告 + ブロック
- 悪意あるトラフィックを遮断
```text

**おすすめ**：
```hcl
# 開発環境
threat_intelligence_mode = "Alert"  # ←影響確認

# 本番環境
threat_intelligence_mode = "Deny"   # ←本気でブロック
```text

#### sku_tier

```hcl
sku_tier = "Standard"
```text

**選択肢**：
```text
Basic：

- 約8万円/月
- 基本機能のみ

Standard：

- 約17万円/月
- 脅威インテリジェンス
- アプリケーションルール

Premium：

- 約25万円/月
- TLS検査
- IDPS（侵入検知/防止）
- Webカテゴリ
```text

Chapter 10と同じやつ。

### firewall_policy

```hcl
firewall_policy = {
  name     = "fwp-vhub-jpe"
  base_policy = {
    parent_firewall_policy_id = null
  }
}
```text

**Virtual WAN独自**：`base_policy`

**何？**：親ポリシーの継承

```text
親ポリシー（本社の共通ルール）
  ↓ 継承
子ポリシー（拠点ごとのルール）
  ↓
拠点ごとにカスタマイズしつつ、共通ルールも適用
```text

**使い方**：
```hcl
# 親ポリシー
base_policy = {
  parent_firewall_policy_id = null  # ←親なし
}

# 子ポリシー
base_policy = {
  parent_firewall_policy_id = "/subscriptions/.../firewallPolicies/fwp-parent"
}
```text

---

## Part 4: VPN Gateway（Virtual WAN版）

### virtual_network_gateway_vpn

```hcl
virtual_network_gateway_vpn = {
  name       = "vpngw-vhub-jpe"
  scale_unit = 1
  
  vpn_sites = {
    site1 = {
      name          = "vpnsite-onprem-tokyo"
      address_cidrs = ["192.168.0.0/16"]
      device_vendor = "Cisco"
      
      link = {
        name       = "link1"
        ip_address = "203.0.113.10"
        bgp = {
          asn             = 65001
          peering_address = "192.168.1.1"
        }
      }
    }
  }
}
```text

#### Hub-and-SpokeのVPN Gatewayとの違い

```text
Hub-and-Spoke：

- VPN Gatewayだけ作る
- 接続は別途設定（Local Network Gateway）

Virtual WAN：

- VPN Gateway + VPN Site（接続先情報）をまとめて設定
- 設定がシンプル
```text

#### scale_unit

```hcl
scale_unit = 1
```text

**何？**：スケールユニット（性能）

**スケール**：
```text
1ユニット：500Mbps、約2.7万円/月
2ユニット：1Gbps、約5.5万円/月
3ユニット：1.5Gbps、約8.2万円/月
...
20ユニット：10Gbps、約54万円/月
```text

**Hub-and-Spokeとの違い**：
```text
Hub-and-Spoke：

- VpnGw1、VpnGw2、VpnGw3（固定SKU）

Virtual WAN：

- 1〜20ユニット（柔軟にスケール）
```text

#### vpn_sites

```hcl
vpn_sites = {
  site1 = {
    name          = "vpnsite-onprem-tokyo"
    address_cidrs = ["192.168.0.0/16"]
    device_vendor = "Cisco"
    
    link = {
      name       = "link1"
      ip_address = "203.0.113.10"  # ←オンプレのPublic IP
      bgp = {
        asn             = 65001      # ←オンプレのAS番号
        peering_address = "192.168.1.1"
      }
    }
  }
}
```text

**何？**：VPN接続先の情報

**構成**：
```text
オンプレ（Tokyo）
  Public IP: 203.0.113.10
  Private Net: 192.168.0.0/16
  BGP ASN: 65001
  ↓ VPN接続
Azure Virtual WAN Hub
```text

**device_vendor**：ログに表示される（任意）

---

## Part 5: VNet接続

### virtual_network_connections

```hcl
virtual_network_connections = {
  spoke1 = {
    name                      = "conn-spoke1"
    remote_virtual_network_id = "/subscriptions/.../virtualNetworks/vnet-spoke1"
    internet_security_enabled = true
    
    routing = {
      associated_route_table_key = "defaultRouteTable"
      propagated_route_table = {
        labels = ["default"]
      }
    }
  }
}
```text

#### Hub-and-SpokeのVNetピアリングとの違い

```text
Hub-and-Spoke：

- VNet Peering作成
- 双方向で設定
- ルーティング手動設定

Virtual WAN：

- Virtual Network Connection作成
- 片方向でOK
- ルーティング自動設定
```text

#### internet_security_enabled

```hcl
internet_security_enabled = true
```text

**何？**：インターネット向けトラフィックをFirewall経由にする

**true**：
```text
Spoke VM → インターネット
  ↓
Virtual Hub Firewall経由
  ↓
検査してから出る
```text

**false**：
```text
Spoke VM → インターネット
  ↓
直接出る（Firewall経由しない）
```text

**普通は**：`true`（セキュリティ重視）

#### routing

```hcl
routing = {
  associated_route_table_key = "defaultRouteTable"
  propagated_route_table = {
    labels = ["default"]
  }
}
```text

**何？**：Virtual WAN独自のルーティング機構

**associated_route_table**：
```text
このVNetが使うルートテーブル
```text

**propagated_route_table**：
```text
このVNetのルートを他のルートテーブルに伝播
```text

**よくわからん？**
次のセクションで詳しく解説するね。

---

## Part 6: Virtual WANのルーティング

### ルートテーブルの概念

**Hub-and-Spokeのルートテーブル**：
```text
サブネットに紐付く
  ↓
送信先：0.0.0.0/0
次ホップ：10.0.0.4（Firewall）
```text

**Virtual WANのルートテーブル**：
```text
Virtual Hub内の論理的なテーブル
  ↓
VNet接続に紐付く
  ↓
自動でルート学習・伝播
```text

### defaultRouteTable

```text
Virtual Hubを作ると自動で作られる
  ↓
全VNet接続のデフォルト
```text

**動き**：
```text
1. Spoke1を接続
   → defaultRouteTableに紐付く
   → Spoke1のルート（10.1.0.0/16）を学習

2. Spoke2を接続
   → defaultRouteTableに紐付く
   → Spoke2のルート（10.2.0.0/16）を学習

3. ルート伝播
   → Spoke1からSpoke2へ通信可能
   → Spoke2からSpoke1へ通信可能
```text

### カスタムルートテーブル

```hcl
route_tables = {
  production = {
    name   = "rt-production"
    labels = ["prod"]
  }
  development = {
    name   = "rt-development"
    labels = ["dev"]
  }
}
```text

**何をしているのでしょうか？**：環境分離

```text
本番VNet
  ↓
rt-productionに紐付け
  ↓
本番同士のみ通信可能

開発VNet
  ↓
rt-developmentに紐付け
  ↓
開発同士のみ通信可能

本番 ↔︎ 開発は通信できない
```text

### Routing Intent

```hcl
routing_intents = {
  default = {
    name = "routing-intent-default"
    routing_policies = [
      {
        name                  = "InternetTraffic"
        destinations          = ["Internet"]
        next_hop_firewall_key = "primary"
      },
      {
        name                  = "PrivateTraffic"
        destinations          = ["PrivateTraffic"]
        next_hop_firewall_key = "primary"
      }
    ]
  }
}
```text

**何？**：全トラフィックをFirewall経由にする設定

**InternetTraffic**：
```text
インターネット向けトラフィック
  ↓
Firewall経由
```text

**PrivateTraffic**：
```text
VNet間、オンプレ向けトラフィック
  ↓
Firewall経由
```text

**効果**：
```text
internet_security_enabled = true と似てる
  ↓
でもこっちの方が強力
  ↓
全トラフィック（プライベート含む）をFirewall経由
```text

---

## Part 7: Sidecar VNet

```hcl
enabled_resources = {
  sidecar_virtual_network = true
}

sidecar_virtual_network = {
  name          = "vnet-sidecar-jpe"
  address_space = ["10.100.0.0/16"]
  subnets = {
    bastion = {
      name             = "AzureBastionSubnet"
      address_prefixes = ["10.100.0.0/26"]
    }
  }
}

bastion = {
  name = "bas-vhub-jpe"
  public_ip = {
    name  = "pip-bas-vhub-jpe"
    zones = []
  }
}
```text

### Sidecar VNetって何？

**問題**：
```text
Virtual Hub
  ↓
Bastionがデプロイできない
  ↓
Hubは仮想ルーター（サブネット作れない）
```text

**解決策**：
```text
Virtual Hubの横に普通のVNetを作る
  ↓
そこにBastionをデプロイ
  ↓
Hubに接続
  ↓
Spoke VMにアクセス可能
```text

**Sidecar（サイドカー）**：バイクの横に付ける椅子

**構成**：
```text
Virtual Hub（メイン）
  ↓ 横に
Sidecar VNet
  └── Bastion
```text

**zones = []**：Chapter 3で見たやつ。Japan regionは必須。

---

## 実践：Virtual WANを作ってみよう

### 最小構成

```hcl
connectivity_type = "virtual_wan"

virtual_wan_settings = {
  enabled_resources = {
    resource_group = true
  }
  resource_group = {
    name     = "rg-vwan"
    location = "japaneast"
  }
  virtual_wan = {
    name = "vwan-alz"
  }
}

virtual_hubs = {
  primary = {
    location                  = "japaneast"
    default_hub_address_space = "10.0.0.0/16"
    
    enabled_resources = {
      firewall                              = false  # ←無効化
      bastion                               = false
      virtual_network_gateway_express_route = false
      virtual_network_gateway_vpn           = false  # ←無効化
      private_dns_zones                     = false
      private_dns_resolver                  = false
      sidecar_virtual_network               = false
    }
    
    hub = {
      name                                   = "vhub-jpe"
      address_prefix                         = "10.0.0.0/23"
      virtual_router_auto_scale_min_capacity = 2
    }
  }
}
```text

**コスト**：
```text
Virtual Hub：約5.5万円/月（2ユニット）
  ↓
これだけ
```text

**Firewall無効化でコスト削減**。

---

## Hub-and-Spoke vs Virtual WAN：どっち使う？

### Hub-and-Spokeがおすすめ

```text
✓ 小〜中規模（VNet 10個以下）
✓ 単一リージョン、または2〜3リージョン
✓ 細かい制御が必要
✓ コスト重視
✓ シンプルな構成
```text

**例**：
```text
日本国内の業務システム
- Japan East/West（2リージョン）
- Spoke VNet 5個
- VPN接続 1本
```text

### Virtual WANがおすすめ

```text
✓ 大規模（VNet 10個以上）
✓ 多数のリージョン（5リージョン以上）
✓ 多数のVPN接続（10本以上）
✓ グローバル展開
✓ 管理の簡素化重視
```text

**例**：
```text
グローバル企業のネットワーク
- 世界10リージョン
- Spoke VNet 50個
- 拠点VPN接続 100本
- SD-WAN統合
```text

---

## デバッグ技

### Virtual WANの確認

```bash
# Virtual WAN一覧
az network vwan list --output table

# 詳細
az network vwan show \
  --resource-group rg-vwan \
  --name vwan-alz
```text

### Virtual Hubの確認

```bash
# Virtual Hub一覧
az network vhub list --output table

# 詳細
az network vhub show \
  --resource-group rg-vwan \
  --name vhub-jpe

# ルーティング確認
az network vhub route-table list \
  --resource-group rg-vwan \
  --vhub-name vhub-jpe
```text

### VNet接続の確認

```bash
# 接続一覧
az network vhub connection list \
  --resource-group rg-vwan \
  --vhub-name vhub-jpe \
  --output table

# Effective Routes（実効ルート）
az network vhub get-effective-routes \
  --resource-group rg-vwan \
  --name vhub-jpe \
  --resource-type VirtualNetworkConnection \
  --resource-id /subscriptions/.../virtualNetworks/vnet-spoke1
```text

**Effective Routes**：超重要

```text
設定したルートじゃなく、実際に使われるルート
  ↓
トラブルシューティングの鍵
```text

---

## よくあるエラー

### エラー1: アドレス空間の重複

```text
Error: address space overlaps
```text

**原因**：Virtual HubとVNetのアドレス空間が重複

**対処法**：
```hcl
# NG
hub.address_prefix: 10.0.0.0/23
spoke VNet: 10.0.0.0/24  # ←重複

# OK
hub.address_prefix: 10.0.0.0/23
spoke VNet: 10.1.0.0/16  # ←分ける
```text

### エラー2: スケールユニット不足

```text
Error: insufficient scale units
```text

**原因**：VNet接続数に対してスケールユニットが少ない

**対処法**：
```hcl
# 目安
1〜5 VNet：2ユニット
6〜10 VNet：3ユニット
11〜20 VNet：4ユニット
...
```text

### エラー3: Routing Intent競合

```text
Error: routing intent already exists
```text

**原因**：Routing Intentは1つのHubに1つだけ

**対処法**：
```hcl
# NG（2個作ろうとしています）
routing_intents = {
  intent1 = { ... }
  intent2 = { ... }
}

# OK（1個だけ）
routing_intents = {
  default = { ... }
}
```text

### エラー4: Sidecar VNet作成失敗

```text
Error: bastion requires sidecar virtual network
```text

**原因**：Bastionを有効化したけどSidecar VNetがない

**対処法**：
```hcl
enabled_resources = {
  bastion                 = true
  sidecar_virtual_network = true  # ←これも有効化
}

sidecar_virtual_network = { ... }
```text

---

## まとめ

**Virtual WANの特徴**：
1. **マネージド**：Microsoftが管理
2. **グローバル**：複数リージョンを自動メッシュ
3. **スケーラブル**：大規模対応
4. **シンプル**：設定が簡単
5. **高コスト**：Hub-and-Spokeより高い

**使い分け**：
```text
小〜中規模 → Hub-and-Spoke
大規模 → Virtual WAN
```text

**重要な設定**：

- `virtual_router_auto_scale_min_capacity`：性能とコスト
- `internet_security_enabled`：Firewall経由
- `routing_intents`：全トラフィック制御
- `sidecar_virtual_network`：Bastion用

次のChapterでは、GitHub Actionsを使った自動デプロイを見ていきます。
Terraform実行を自動化するやつ。

---

**所要時間**: 55分  
**難易度**: ★★★★★  
**前**: [10_Hub-and-Spoke.md](./10_Hub-and-Spoke.md)  
**次**: [12_GitHub_Actions.md](./12_GitHub_Actions.md)

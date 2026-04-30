# AWS 高可用性・コスト最適化ハイブリッドアーキテクチャの構築

本プロジェクトは、「高可用性」「セキュリティ」「コスト最適化」の3要素を軸とした、AWS上でのエンタープライズ向けインフラ構築の実践をまとめたものです。

## 1. アーキテクチャ構成図

> [!TIP]
> 以下の図は、本プロジェクトのネットワーク、コンピューティング、およびサーバーレス拡張の統合を示しています。

<img width="1753" height="1969" alt="mermaid-diagram-2026-04-30-175128" src="https://github.com/user-attachments/assets/ac8bf2a2-f183-4ac4-baa6-9afad03423e8" />

```
graph TB

%% --- スタイル定義 ---
classDef vpc fill:#ffffff,stroke:#000000,stroke-width:2px;
classDef subnet fill:#ffffff,stroke:#666666,stroke-width:1px,stroke-dasharray: 5 5;
classDef highlight fill:#f8f8f8,stroke:#333333,font-weight:bold;
classDef darkNode fill:#333333,stroke:#000000,color:#ffffff;
classDef stepNode fill:#ffffff,stroke:#333333,stroke-width:2px,font-style:italic;

%% --- 外部ユーザー ---
User([ユーザー / ブラウザ])

subgraph AWS_Cloud ["AWS Cloud (Region)"]
    
    %% Phase 4: Serverless
    subgraph Serverless ["Phase 4: Serverless 拡張 (API + Lambda)"]
        S3Static[S3: 静的サイト]
        APIGW[API Gateway]
        Lambda{Lambda: 最小権限原則}
        DynamoDB[(DynamoDB: UserInfo)]
    end

    %% Phase 1: Network
    subgraph VPC_Network ["Phase 1: VPC 基盤 (10.0.0.0/16)"]
        
        subgraph PublicSubnet ["パブリックサブネット (ALB配置)"]
            ALB[ALB: 負荷分散]
        end

        %% Phase 2: Compute
        subgraph PrivateSubnet ["Phase 2: 高可用バックエンド (EC2 x 2)"]
            EC2A[EC2 Instance A]
            EC2B[EC2 Instance B]
        end

        %% Phase 3: Cost Optimization
        VPCE{{Phase 3: S3 Gateway Endpoint}}
    end

    S3_Data[(Amazon S3: データ保存)]
end

%% --- 接続ライン ---
User -->|HTTP/80| ALB
ALB --> EC2A
ALB --> EC2B

EC2A -.->|IAM Role 賦権| VPCE
EC2B -.->|IAM Role 賦権| VPCE
VPCE -.->|コスト最適化通信| S3_Data

User -.-> S3Static
S3Static -.->|CORS解決| APIGW
APIGW --> Lambda
Lambda -->|Read/Write| DynamoDB

%% --- クラス適用 ---
class VPC_Network vpc;
class PublicSubnet,PrivateSubnet subnet;
class ALB,EC2A,EC2B,Lambda,DynamoDB,S3Static,APIGW highlight;
class User darkNode;
class VPCE stepNode;
```

## 2. プロジェクト概要
既存のインフラ構成における「NAT Gatewayによる通信コストの増大」および「公衆網を経由するセキュリティリスク」という課題を解決するため、VPC Endpointを活用したセキュアかつ低コストな構成への移行と、Serverless技術による機能拡張を実証しました。

## 3. 実装の詳細（4つのフェーズ）

### Phase 1: ネットワーク基盤の構築 (Network Foundation)
*   **VPC設計**: `10.0.0.0/16` のCIDRブロックを採用し、将来的なスケーラビリティを確保。
*   **サブネット分離**: 外部露出を最小限に抑えるため、Public/Privateサブネットを厳密に分離。
*   **IGW設定**: パブリックサブネットのみにInternet Gatewayを紐付け、ルートテーブルによりトラフィックを制御。

### Phase 2: 高可用バックエンドの構築 (HA Compute)
*   **Multi-AZ構成**: 2つのアベイラビリティゾーンにEC2インスタンスを分散配置し、データセンター障害への耐性を確保。
*   **ALB (Application Load Balancer)**: ヘルスチェック機能を活用し、障害発生時の自動フェイルオーバーを実現。
*   **セキュリティグループ**: インバウンドルールを「ALBからのHTTP通信のみ」に制限し、EC2の直接露出を遮断。

### Phase 3: 内網セキュリティとコスト最適化 (Cost Optimization)
*   **S3 Gateway Endpoint**: 従来のNAT Gatewayを廃止し、VPCエンドポイントを導入。
*   **メリット**:
    *   **コスト削減**: データ転送料金およびNAT Gatewayの稼働コストを100%削減。
    *   **セキュリティ**: S3へのトラフィックをAWS内部ネットワーク内に閉じ込め、パブリックインターネットからの隔離を強化。
*   **IAM Role**: EC2インスタンスに対し、S3への読み取り専用（`AmazonS3ReadOnlyAccess`）など、最小権限の原則（PoLP）に基づく権限を付与。

### Phase 4: サーバーレスへの拡張 (Serverless Extension)
*   **アーキテクチャ**: S3(Static) + API Gateway + Lambda + DynamoDB
*   **実装内容**:
    *   Lambdaを用いたDynamoDBへのアトミックなデータ操作。
    *   API Gatewayにおける**CORS (Cross-Origin Resource Sharing)** 設定の最適化。
    *   環境変数とIAMポリシーを用いたセキュアな認証・認可フロー。

## 4. 技術スタック
| 分類 | 技術 |
| :--- | :--- |
| **Computing** | EC2 (Amazon Linux 2023), AWS Lambda |
| **Networking** | VPC, ALB, Internet Gateway, VPC Endpoint |
| **Storage/DB** | Amazon S3, Amazon DynamoDB |
| **Security** | IAM (Role/Policy), Security Group, Route Table |

## 5. 構築成果
*   **可用性**: マルチAZ構成とALBにより、単一障害点（SPOF）を排除した堅牢なインフラを構築。
*   **コスト**: NAT Gatewayの廃止により、固定費およびデータ転送コストを劇的に削減。
*   **セキュリティ**: 核心的なビジネスロジックとデータベースを完全にプライベート化し、攻撃面を最小化。
```

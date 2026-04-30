1. アーキテクチャ構成図


2. プロジェクト概要
既存のインフラ構成における「NAT Gatewayによる通信コストの増大」および「公衆網を経由するセキュリティリスク」という課題を解決するため、VPC Endpointを活用したセキュアかつ低コストな構成への移行と、Serverless技術による機能拡張を実証しました。

3. 実装の詳細（4つのフェーズ）
Phase 1: ネットワーク基盤の構築 (Network Foundation)
VPC設計: 10.0.0.0/16 のCIDRブロックを採用し、拡張性を確保。

サブネット分離: 外部露出を最小限に抑えるため、Public/Privateサブネットを厳密に分離。

IGW設定: パブリックサブネットのみにIGWを紐付け、インターネットへの出口を制御。

Phase 2: 高可用バックエンドの構築 (HA Compute)
Multi-AZ構成: 2つのアベイラビリティゾーンにEC2を分散配置し、耐障害性を向上。

ALB (Application Load Balancer): ヘルスチェックによる自動フェイルオーバー機能を実装。

セキュリティグループ: ALBからのHTTP(80)通信のみを許可するインバウンドルールを適用。

Phase 3: 内網セキュリティとコスト最適化 (Cost Optimization)
S3 Gateway Endpoint: NAT Gatewayを廃止し、VPCエンドポイントを導入。

メリット:

コスト削減: データ転送料金およびNAT Gatewayの稼働コストを100%削減。

セキュリティ: S3へのアクセスをAWS内部ネットワークに限定。

IAM Role: EC2に対し、S3への読み取り専用（AmazonS3ReadOnlyAccess）の最小権限を付与。

Phase 4: サーバーレスへの拡張 (Serverless Extension)
構成: API Gateway + Lambda + DynamoDB

実装内容:

Lambdaを用いたDynamoDBへのデータ読み書き処理。

API GatewayでのCORS（跨域資源共有）設定によるフロントエンド連携の解決。

IAMポリシーによるリソースレベルでのアクセス制限。

4. 技術スタック
Computing: EC2, Lambda

Networking: VPC, ALB, Internet Gateway, VPC Endpoint (Gateway type)

Storage/DB: S3, DynamoDB

Security: IAM, Security Group, Route Table

5. 構築成果
可用性: マルチAZ構成により単一障害点(SPOF)を排除。

コスト: NAT Gatewayの廃止により、固定費およびデータ通信費を大幅に削減。

セキュリティ: 核心リソースの完全プライベート化を実現。

# Cloud Services Selection Guide

## 雲端平台選擇

### 平台特性比較

**AWS (Amazon Web Services)**
- **優勢**: 市場領導者、服務最完整、全球覆蓋最廣
- **適用**: 需要多樣化服務、全球部署、成熟生態系
- **成本**: 中等偏高，但有豐富的成本優化選項

**Azure (Microsoft Azure)**
- **優勢**: 與Microsoft生態系緊密整合、企業級支援
- **適用**: Windows/.NET 應用、混合雲、企業市場
- **成本**: 與 AWS 相近，企業方案優惠多

**GCP (Google Cloud Platform)**
- **優勢**: 機器學習/大數據強項、Kubernetes 原生、網路效能佳
- **適用**: 數據分析、ML/AI 應用、容器化應用
- **成本**: 通常比 AWS/Azure 便宜 10-20%

### 選擇決策因素

1. **團隊經驗**: 團隊最熟悉的平台
2. **現有基礎設施**: 是否已有雲端投資
3. **特定服務需求**: 需要哪些獨特服務
4. **成本預算**: 預算限制與成本優化需求
5. **合規要求**: 資料駐留、合規認證
6. **技術生態系**: 第三方整合需求

## 雲端服務選型指南

### 1. Compute Services (運算服務)

#### Serverless Functions
- **AWS**: Lambda
- **Azure**: Azure Functions
- **GCP**: Cloud Functions
- **適用場景**:
  - 事件驅動應用
  - 短時間運行任務
  - 不規則流量模式
  - 微服務架構
- **優點**: 無需管理伺服器、按需計費、自動擴展
- **缺點**: 冷啟動延遲、執行時間限制、供應商鎖定
- **成本考量**: 適合低頻率或突發性工作負載

#### Container Services
- **AWS**: ECS (Elastic Container Service), EKS (Elastic Kubernetes Service)
- **Azure**: Container Instances, AKS (Azure Kubernetes Service)
- **GCP**: Cloud Run, GKE (Google Kubernetes Engine)
- **適用場景**:
  - 微服務架構
  - 需要完整控制運行環境
  - 持續運行的應用
  - 需要複雜編排
- **優點**: 靈活性高、可移植性好、成熟生態系
- **缺點**: 管理複雜度較高、需要 Kubernetes 知識
- **成本考量**: 適合中到高頻率穩定負載

#### Virtual Machines
- **AWS**: EC2 (Elastic Compute Cloud)
- **Azure**: Virtual Machines
- **GCP**: Compute Engine
- **適用場景**:
  - 傳統應用遷移
  - 需要完全控制作業系統
  - 特殊軟體需求
- **優點**: 最大控制度、相容性最佳
- **缺點**: 管理負擔最重、擴展較慢
- **成本考量**: Reserved Instances 可節省 30-70%

### 2. Database Services (資料庫服務)

#### Relational Databases (關聯式資料庫)
- **AWS**: RDS (PostgreSQL, MySQL, MariaDB, Oracle, SQL Server), Aurora
- **Azure**: Azure SQL Database, Azure Database for PostgreSQL/MySQL
- **GCP**: Cloud SQL (PostgreSQL, MySQL), Cloud Spanner
- **適用場景**:
  - 交易型應用 (OLTP)
  - 複雜查詢與 JOIN
  - ACID 要求嚴格
  - 傳統應用遷移
- **選擇建議**:
  - PostgreSQL: 功能豐富、開源、適合複雜應用
  - MySQL: 簡單易用、生態系成熟
  - Aurora: AWS 獨有、高效能、MySQL/PostgreSQL 相容
  - Cloud Spanner: GCP 獨有、全球分散式、強一致性

#### NoSQL Databases
- **AWS**: DynamoDB (Key-Value/Document), DocumentDB (MongoDB-compatible)
- **Azure**: Cosmos DB (Multi-model), Table Storage
- **GCP**: Firestore (Document), Bigtable (Wide-column)
- **適用場景**:
  - 高吞吐量讀寫
  - 彈性 Schema
  - 橫向擴展需求
  - 簡單查詢模式
- **選擇建議**:
  - DynamoDB: 極致效能、按需擴展、適合 Key-Value
  - Cosmos DB: 多模型、全球分散、SLA 保證
  - Firestore: 即時同步、適合行動應用

#### Cache Services (快取服務)
- **AWS**: ElastiCache (Redis, Memcached)
- **Azure**: Azure Cache for Redis
- **GCP**: Memorystore (Redis, Memcached)
- **適用場景**:
  - 減少資料庫負載
  - Session 儲存
  - 即時排行榜
  - 發布/訂閱

### 3. Storage Services (儲存服務)

#### Object Storage
- **AWS**: S3 (Simple Storage Service)
- **Azure**: Blob Storage
- **GCP**: Cloud Storage
- **適用場景**:
  - 靜態檔案儲存 (圖片、影片、文件)
  - 備份與歸檔
  - 資料湖 (Data Lake)
  - CDN 來源
- **成本優化**:
  - Lifecycle Policies (自動轉移至低成本儲存層)
  - Intelligent Tiering
  - 壓縮與去重

#### File Storage
- **AWS**: EFS (Elastic File System)
- **Azure**: Azure Files
- **GCP**: Filestore
- **適用場景**:
  - 共享檔案系統
  - 需要 POSIX 相容
  - 容器間共享資料

### 4. Messaging & Event Services (訊息與事件服務)

#### Message Queue
- **AWS**: SQS (Simple Queue Service)
- **Azure**: Service Bus
- **GCP**: Pub/Sub (也可當 Queue 使用)
- **適用場景**:
  - 非同步處理
  - 工作佇列
  - 解耦微服務
  - 流量削峰

#### Event Bus / Streaming
- **AWS**: EventBridge, Kinesis
- **Azure**: Event Grid, Event Hubs
- **GCP**: Pub/Sub, Dataflow
- **適用場景**:
  - 事件驅動架構
  - 即時資料流處理
  - 微服務間通訊
  - IoT 資料收集

### 5. API Gateway & Networking

#### API Gateway
- **AWS**: API Gateway (REST/WebSocket/HTTP)
- **Azure**: API Management, Application Gateway
- **GCP**: Cloud Endpoints, API Gateway
- **功能**:
  - 路由與負載平衡
  - 認證與授權
  - Rate Limiting & Throttling
  - 請求/回應轉換
  - API 版本管理

#### Load Balancer
- **AWS**: ALB (Application Load Balancer), NLB (Network Load Balancer)
- **Azure**: Application Gateway, Load Balancer
- **GCP**: Cloud Load Balancing
- **選擇建議**:
  - Layer 7 (HTTP/HTTPS): ALB, Application Gateway
  - Layer 4 (TCP/UDP): NLB, Network Load Balancer

#### CDN (Content Delivery Network)
- **AWS**: CloudFront
- **Azure**: Azure CDN
- **GCP**: Cloud CDN
- **用途**:
  - 加速靜態內容傳輸
  - 降低延遲
  - 減少源站負載
  - DDoS 防護

### 6. VPC & Network Security

#### VPC Design Best Practices
- **Public Subnet**: 放置 Load Balancer, NAT Gateway, Bastion Host
- **Private Subnet**: 放置應用伺服器、資料庫
- **Network ACLs**: 子網層級防火牆
- **Security Groups**: 實例層級防火牆
- **NAT Gateway**: 允許私有子網訪問網際網路

#### Security Groups 設計原則
- **最小權限原則**: 只開放必要的埠
- **明確來源**: 指定 CIDR 範圍，避免 0.0.0.0/0
- **分層設計**: Web Tier, App Tier, DB Tier 分別設定
- **動態調整**: 使用標籤與自動化管理

## 成本優化策略

### 1. Compute 成本優化
- **Reserved Instances**: 長期承諾換取 30-70% 折扣
- **Savings Plans**: 靈活的承諾方案
- **Spot Instances**: 競標閒置容量，節省 50-90%（適合非關鍵任務）
- **Auto Scaling**: 依需求自動調整資源

### 2. Storage 成本優化
- **Lifecycle Policies**: 自動轉移至低成本儲存層
- **Compression**: 壓縮資料減少儲存空間
- **Deduplication**: 去除重複資料
- **定期清理**: 刪除過期資料

### 3. Network 成本優化
- **CDN 使用**: 減少跨區域流量
- **VPC Peering**: 降低跨區域通訊成本
- **Private Endpoints**: 避免公網流量費用
- **Data Transfer 規劃**: 最小化跨區域傳輸

### 4. 監控與告警
- **AWS**: Cost Explorer, Budgets, CloudWatch
- **Azure**: Cost Management, Advisor
- **GCP**: Cost Management, Billing Alerts
- **建議**: 設定預算告警、定期檢視成本報告、使用成本分配標籤

## 高可用性設計

### Multi-AZ Deployment
- **資料庫**: 主備切換 (Primary-Standby)
- **應用層**: 跨可用區部署
- **Load Balancer**: 自動偵測健康狀況
- **目標**: RTO < 5 分鐘, RPO < 1 分鐘

### Disaster Recovery 策略
- **Backup Strategy**: 自動化定期備份
- **Cross-Region Replication**: 跨區域複製關鍵資料
- **Failover Plan**: 明確的切換程序
- **定期演練**: 每季進行 DR 演練

## 安全架構最佳實踐

### IAM (Identity and Access Management)
- **最小權限原則**: 只授予必要權限
- **Service Roles**: 為服務建立專用角色
- **MFA**: 啟用多因子認證
- **定期審計**: 檢視存取日誌

### 加密策略
- **Data at Rest**: RDS 加密、S3 加密
- **Data in Transit**: TLS/SSL 加密
- **Key Management**: 使用雲端原生 KMS
- **加密範圍**: 資料庫、儲存、備份

### 合規要求
- **GDPR**: 資料駐留、隱私保護
- **HIPAA**: 醫療資料保護
- **SOC 2**: 安全控管框架
- **PCI DSS**: 支付卡產業資料安全標準

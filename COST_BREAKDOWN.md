# Azure CAF Module - Cost Breakdown Analysis

This document provides comprehensive cost analysis for all Azure services supported by the Cloud Adoption Framework (CAF) Terraform module. Use this guide to estimate, optimize, and manage costs for your Azure infrastructure deployments.

> **Cost Disclaimer**: Prices shown are estimates based on US East 2 region as of 2024 and may vary by region, commitment level, and current Azure pricing. Always use the [Azure Pricing Calculator](https://azure.microsoft.com/en-us/pricing/calculator/) for accurate, up-to-date pricing.

## 📊 Cost Categories Overview

| Category | Monthly Cost Range | Primary Cost Drivers | Optimization Opportunities |
|----------|-------------------|---------------------|---------------------------|
| **Compute** | $50 - $10,000+ | VM size, hours, licensing | Reserved instances, spot VMs, right-sizing |
| **Networking** | $20 - $2,000+ | Data transfer, gateway hours | Traffic optimization, peering |
| **Storage** | $10 - $1,000+ | Capacity, access tier, redundancy | Lifecycle policies, compression |
| **Databases** | $100 - $5,000+ | Compute tier, storage, backup | Serverless, elastic pools |
| **Security** | $50 - $500+ | Key Vault transactions, Sentinel | Usage optimization, log retention |
| **Analytics** | $200 - $3,000+ | Data processing, query units | Reserved capacity, partitioning |

---

## 💻 Compute Services

### Virtual Machines (`compute_virtual_machines.tf`)

#### Cost Structure
```yaml
Base VM Costs (per month):
  Standard_B1s:    $7.59   # 1 vCPU, 1GB RAM - Development
  Standard_B2s:    $15.18  # 2 vCPU, 4GB RAM - Small workloads
  Standard_D2s_v3: $70.08  # 2 vCPU, 8GB RAM - General purpose
  Standard_D4s_v3: $140.16 # 4 vCPU, 16GB RAM - Production
  Standard_E8s_v3: $438.00 # 8 vCPU, 64GB RAM - Memory intensive

Additional Costs:
  OS Disk (P10 - 128GB):     $19.71/month
  Data Disk (P20 - 512GB):   $65.57/month
  Windows License:           +$0.096/hour per core
  Public IP (Static):        $3.65/month
```

#### Optimization Strategies
- **Reserved Instances**: Save 40-60% with 1-3 year commitments
- **Spot VMs**: Save up to 90% for fault-tolerant workloads
- **Hybrid Benefit**: Use existing Windows licenses to save licensing costs
- **Auto-shutdown**: Implement scheduled shutdown for dev/test environments

#### Example Monthly Costs
```yaml
Development Environment:
  - VM: Standard_B2s = $15.18
  - OS Disk: P10 = $19.71
  - Total: ~$35/month

Production Environment:
  - VM: Standard_D4s_v3 = $140.16
  - OS Disk: P30 = $122.88
  - Data Disk: P40 = $153.60
  - Load Balancer: $18.25
  - Total: ~$435/month
```

### Azure Kubernetes Service (`compute_aks_clusters.tf`)

#### Cost Structure
```yaml
Control Plane: FREE (managed by Azure)

Node Pools (per node):
  Standard_DS2_v2: $73.00/month  # 2 vCPU, 7GB RAM
  Standard_DS3_v2: $146.00/month # 4 vCPU, 14GB RAM
  Standard_DS4_v2: $292.00/month # 8 vCPU, 28GB RAM

Additional Services:
  Load Balancer Standard:    $18.25/month
  Public IP:                 $3.65/month per IP
  Managed Disk per node:     $19.71/month (P10)
```

#### Cost Optimization
- **Node Autoscaling**: Scale based on demand
- **Spot Node Pools**: Use for batch workloads (up to 90% savings)
- **Reserved Instances**: Apply to node pools
- **Right-sizing**: Monitor resource utilization

### Container Apps (`container_apps.tf`)

#### Cost Structure
```yaml
Pricing Model: Pay-per-use (vCPU-second and GB-second)

vCPU: $0.000014 per vCPU-second
Memory: $0.0000015 per GB-second

Monthly Examples (730 hours):
  0.25 vCPU, 0.5GB: ~$8.50/month
  1 vCPU, 2GB:      ~$40/month
  2 vCPU, 4GB:      ~$80/month
```

### Function Apps (`modules/webapps/function_app/`)

#### Cost Structure
```yaml
Consumption Plan:
  - First 1M executions: FREE
  - Additional: $0.0000002 per execution
  - Execution time: $0.000016 per GB-second

Premium Plan:
  EP1: $146/month  # 1 vCPU, 3.5GB RAM
  EP2: $292/month  # 2 vCPU, 7GB RAM
  EP3: $584/month  # 4 vCPU, 14GB RAM

Dedicated Plan: Same as App Service Plan pricing
```

---

## 🌐 Networking Services

### Virtual Networks (`networking_virtual_networks.tf`)
```yaml
VNet: FREE
Subnets: FREE
NSGs: FREE
Route Tables: FREE

Peering Costs:
  Inbound:  $0.01 per GB
  Outbound: $0.01 per GB
```

### Application Gateway (`application_gateways.tf`)
```yaml
Gateway Hours:
  Standard_v2: $0.0225/hour = $16.43/month
  WAF_v2:      $0.0315/hour = $23/month

Data Processing:
  $0.008 per GB processed

Capacity Units (auto-scaling):
  $0.0072 per Capacity Unit hour

Monthly Estimate (medium load):
  Gateway: $23/month
  Processing (100GB): $0.80/month
  Capacity Units (avg 5): $26.28/month
  Total: ~$50/month
```

### Load Balancer (`networking_load_balancers.tf`)
```yaml
Standard Load Balancer:
  Base: $18.25/month
  Data Processed: $0.005 per GB

Basic Load Balancer:
  FREE (limited features, no SLA)

Example Monthly Cost:
  Standard LB: $18.25
  Data (500GB): $2.50
  Total: ~$21/month
```

### VPN Gateway (`networking_virtual_network_gateways.tf`)
```yaml
Gateway SKUs (per hour):
  Basic:     $0.04   = $29.20/month   # Max 10 tunnels
  VpnGw1:    $0.19   = $138.70/month  # Max 30 tunnels
  VpnGw2:    $0.49   = $357.70/month  # Max 30 tunnels
  VpnGw3:    $1.25   = $912.50/month  # Max 30 tunnels

Data Transfer:
  Ingress: FREE
  Egress: $0.087 per GB (first 5GB free)
```

### Azure Firewall (`networking_firewall.tf`)
```yaml
Deployment:
  Standard: $1.25/hour = $912.50/month
  Premium:  $0.875/hour = $638.75/month

Data Processing:
  $0.016 per GB processed

Example Monthly Cost (1TB data):
  Firewall Premium: $638.75
  Data Processing: $16.00
  Total: ~$655/month
```

---

## 🗄️ Storage Services

### Storage Accounts (`storage_accounts.tf`)

#### Cost Structure by Tier
```yaml
Hot Tier (Frequently Accessed):
  LRS: $0.0184 per GB/month
  ZRS: $0.023 per GB/month
  GRS: $0.037 per GB/month

Cool Tier (30+ days):
  LRS: $0.01 per GB/month
  ZRS: $0.0125 per GB/month
  GRS: $0.02 per GB/month

Archive Tier (180+ days):
  LRS: $0.00099 per GB/month
  GRS: $0.00198 per GB/month

Transactions:
  Hot: $0.0004 per 10,000 operations
  Cool: $0.001 per 10,000 operations
  Archive: $0.005 per 10,000 operations
```

#### Monthly Examples
```yaml
Small Business (100GB Hot, LRS):
  Storage: $1.84
  Transactions (1M): $0.04
  Total: ~$2/month

Enterprise (10TB Mixed):
  Hot (2TB): $36.80
  Cool (5TB): $50.00
  Archive (3TB): $2.97
  Total: ~$90/month
```

### NetApp Files (`netapp.tf`)
```yaml
Service Levels (per GB/month):
  Standard: $0.000202 = $0.148/GB/month
  Premium:  $0.000347 = $0.254/GB/month
  Ultra:    $0.000694 = $0.508/GB/month

Minimum: 4TB pool
Example 4TB Premium: 4,096GB × $0.254 = $1,040/month
```

---

## 🛢️ Database Services

### Azure SQL Database (`databases_mssql_database.tf`)

#### Serverless Compute
```yaml
General Purpose (per vCore/hour when active):
  Gen5: $0.52/hour

Minimum: 0.5 vCore
Auto-pause: After 1 hour idle (configurable)

Monthly Examples:
  0.5 vCore (50% utilization): ~$95/month
  2 vCore (30% utilization):   ~$228/month
```

#### Provisioned Compute
```yaml
General Purpose DTU:
  S0 (10 DTU):   $14.70/month
  S2 (50 DTU):   $73.51/month
  S4 (200 DTU):  $294.05/month

Business Critical:
  BC_Gen5_2:     $1,022.50/month
  BC_Gen5_4:     $2,045.00/month
```

### Azure Database for PostgreSQL (`databases_postgresql_server.tf`)
```yaml
Flexible Server (per vCore/month):
  Burstable B1ms: $12.41   # 1 vCore, 2GB RAM
  General Purpose D2s_v3: $140.16  # 2 vCore, 8GB RAM
  Memory Optimized E2s_v3: $175.20  # 2 vCore, 16GB RAM

Storage: $0.115 per GB/month
Backup: $0.095 per GB/month (beyond 7-day retention)
```

### Cosmos DB (`cosmos_db.tf`)

#### Provisioned Throughput
```yaml
Request Units (RU/s):
  100 RU/s:    $5.84/month
  400 RU/s:    $23.36/month (minimum for container)
  1,000 RU/s:  $58.40/month
  10,000 RU/s: $584.00/month

Storage: $0.25 per GB/month
```

#### Serverless
```yaml
Request Units: $0.285 per million RUs consumed
Storage: $0.25 per GB/month

Good for: Unpredictable workloads, dev/test
Cost effective under 5,000 RU/s average
```

---

## 🔐 Security Services

### Key Vault (`keyvault.tf`)
```yaml
Operations:
  Key Operations: $0.03 per 10,000 operations
  Secret Operations: $0.03 per 10,000 operations
  Certificate Operations: $3.00 per request

HSM-backed Keys:
  RSA 2048-bit: $1.00 per key per month
  EC P-256:     $1.00 per key per month

Monthly Examples:
  Basic Usage (10K operations): $0.03/month
  Heavy Usage (1M operations): $3.00/month
  With HSM keys (10 keys): $10.00/month
```

### Azure Sentinel (`modules/security/sentinel/`)
```yaml
Data Ingestion:
  First 5GB/day: FREE for first 31 days
  After free tier: $2.30 per GB/day

Data Retention:
  First 90 days: FREE
  Beyond 90 days: $0.12 per GB/month

Example Monthly Costs:
  10GB/day: $230/month (after free trial)
  50GB/day: $1,150/month
  100GB/day: $2,300/month
```

---

## 📊 Analytics Services

### Azure Synapse (`modules/databases/synapse/`)
```yaml
SQL Pool (DWU per hour):
  DW100c: $1.20/hour = $876/month
  DW500c: $6.00/hour = $4,380/month
  DW1000c: $12.00/hour = $8,760/month

Serverless SQL Pool:
  Query Processing: $5.00 per TB processed
  Storage: Standard rates

Spark Pools:
  Small (4 vCores): $0.33/hour
  Medium (8 vCores): $0.66/hour
  Large (16 vCores): $1.32/hour
```

### Data Factory (`data_factory.tf`)
```yaml
Orchestration:
  Pipeline Activity: $1.00 per 1,000 activities

Data Movement:
  Cloud: $0.25 per DIU-hour
  Hybrid: $2.50 per hour

Data Flow:
  General Purpose: $0.274 per vCore-hour
  Memory Optimized: $0.35 per vCore-hour
```

### Databricks (`databricks.tf`)
```yaml
DBU (Databricks Unit) Pricing:
  Standard: $0.15 per DBU
  Premium: $0.55 per DBU

VM Costs (additional):
  Standard_DS3_v2: $0.192/hour
  Standard_DS4_v2: $0.384/hour

Example 3-node cluster (8 hours/day):
  DBUs (Premium): 24 DBU × $0.55 = $13.20/day
  VMs: 3 × $0.192 × 8 = $4.61/day
  Total: ~$530/month
```

---

## 🌐 Web Applications

### App Service (`app_services.tf`)
```yaml
Service Plans:
  F1 (Free):        FREE (60 min/day CPU, 1GB RAM)
  D1 (Shared):      $9.67/month
  B1 (Basic):       $54.75/month  # 1 core, 1.75GB RAM
  B2 (Basic):       $109.50/month # 2 core, 3.5GB RAM
  S1 (Standard):    $73.00/month  # Auto-scale, staging slots
  P1v3 (Premium):   $146.00/month # Advanced features

Custom Domains: FREE
SSL Certificates: FREE (App Service Managed)
```

### Logic Apps (`logic_app.tf`)

#### Consumption Plan
```yaml
Actions:
  Standard: $0.000025 per action
  Enterprise: $0.00008 per action

Monthly Examples (10,000 executions):
  Simple workflow (5 actions): $1.25/month
  Complex workflow (20 actions): $5.00/month
```

#### Standard Plan
```yaml
WS1: $192.05/month  # 1 vCPU, 3.5GB RAM
WS2: $384.10/month  # 2 vCPU, 7GB RAM
WS3: $576.15/month  # 4 vCPU, 14GB RAM
```

---

## 📨 Messaging Services

### Service Bus (`modules/messaging/servicebus/`)
```yaml
Messaging Tiers:
  Basic:    $0.05 per million operations
  Standard: $0.80 base + $0.10 per million operations
  Premium:  $687.77/month (1 messaging unit)

Storage (Standard/Premium):
  $0.10 per GB per month

Example Monthly Costs:
  Basic (1M operations): $0.05/month
  Standard (10M operations): $1.80/month
  Premium: $687.77/month (includes 1GB storage)
```

### Event Hub (`event_hubs.tf`)
```yaml
Throughput Units (Standard):
  1 TU: $22.36/month (1MB/s ingress, 2MB/s egress)
  10 TU: $223.60/month
  20 TU: $447.20/month (maximum)

Basic Tier:
  $11.18/month (limited features)

Dedicated:
  $8,766.44/month (1 Capacity Unit = 200 TU equivalent)

Storage:
  $0.10 per GB per month
```

---

## 📈 Monitoring Services

### Log Analytics (`log_analytics.tf`)
```yaml
Data Ingestion:
  Pay-per-GB: $2.30 per GB

Commitment Tiers (monthly):
  100GB: $196/month ($1.96/GB)
  200GB: $374/month ($1.87/GB)
  300GB: $546/month ($1.82/GB)
  400GB: $704/month ($1.76/GB)
  500GB: $850/month ($1.70/GB)

Data Retention:
  First 31 days: FREE
  31-730 days: $0.12 per GB per month
  730+ days: $0.05 per GB per month
```

### Application Insights (`azurerm_application_insights.tf`)
```yaml
Data Volume:
  First 5GB/month: FREE
  Beyond 5GB: $2.30 per GB

Web Tests:
  Multi-step: $3.00 per test per month
  Ping tests: FREE (up to 5 tests)

Example Monthly Costs:
  Small app (2GB): FREE
  Medium app (20GB): $34.50/month
  Large app (100GB): $218.50/month
```

---

## 💰 Cost Optimization Strategies

### 1. Reserved Instances & Savings Plans
```yaml
Savings Potential:
  1-year Reserved: 20-40% savings
  3-year Reserved: 40-60% savings
  Spot VMs: Up to 90% savings

Best for:
  ✅ Predictable workloads
  ✅ Long-term projects (1+ years)
  ✅ Production environments
  ❌ Development/testing (use auto-shutdown instead)
```

### 2. Auto-Scaling & Scheduling
```yaml
Strategies:
  - VM auto-shutdown: 50-70% savings for dev/test
  - AKS node autoscaling: 20-40% savings
  - App Service auto-scale: 15-30% savings
  - Function Apps consumption: Pay only for execution

Implementation:
  - Scale down after business hours
  - Weekend shutdowns for non-prod
  - Seasonal scaling adjustments
```

### 3. Storage Optimization
```yaml
Lifecycle Policies:
  Hot → Cool (30 days): 46% storage savings
  Cool → Archive (90 days): 90% storage savings

Techniques:
  - Blob lifecycle management
  - Unused disk deletion
  - Compress data before storage
  - Use appropriate redundancy level
```

### 4. Right-Sizing Resources
```yaml
Common Over-Provisioning:
  VM CPUs: Often 2-4x oversized
  Database DTUs: 30-50% overprovisioned
  App Service Plans: Wrong tier selection

Monitoring Tools:
  - Azure Advisor recommendations
  - Azure Cost Management
  - Third-party cost optimization tools
```

### 5. Network Cost Optimization
```yaml
Data Transfer Costs:
  Inbound: Usually FREE
  Outbound: $0.087+ per GB

Optimization:
  - Use CDN for global content
  - Optimize data transfer patterns
  - Leverage Azure regions strategically
  - Use ExpressRoute for high-volume transfers
```

---

## 📊 Sample Environment Costs

### Startup/Development Environment
```yaml
Resources:
  - 2x B2s VMs: $30/month
  - Basic Load Balancer: FREE
  - 100GB Storage (Hot): $2/month
  - Basic SQL Database (S0): $15/month
  - Application Insights: FREE (under 5GB)

Total: ~$50/month
```

### Small Business Environment
```yaml
Resources:
  - 3x D2s_v3 VMs: $210/month
  - Standard Load Balancer: $20/month
  - 1TB Storage (mixed tiers): $25/month
  - SQL Database (S2): $74/month
  - App Service (B1): $55/month
  - Key Vault: $5/month

Total: ~$390/month
```

### Enterprise Environment
```yaml
Resources:
  - 10x D4s_v3 VMs (Reserved): $840/month
  - Application Gateway: $75/month
  - 10TB Storage (lifecycle managed): $200/month
  - SQL Database (Business Critical): $1,000/month
  - AKS Cluster (3 nodes): $220/month
  - Azure Firewall: $650/month
  - Log Analytics (500GB): $850/month
  - Cosmos DB: $300/month

Total: ~$4,135/month
```

---

## 🛠️ Cost Management Tools

### Built-in Azure Tools
- **Azure Cost Management**: Free cost analysis and budgets
- **Azure Advisor**: Right-sizing recommendations
- **Azure Pricing Calculator**: Estimate costs before deployment
- **Azure TCO Calculator**: Compare on-premises vs. Azure costs

### Third-Party Tools
- **CloudHealth**: Multi-cloud cost management
- **Spot.io**: Automated cost optimization
- **Densify**: Resource optimization recommendations

### Monitoring & Alerts
```hcl
# Example: Set budget alerts in Terraform
resource "azurerm_consumption_budget_subscription" "budget" {
  name            = "monthly-budget"
  subscription_id = data.azurerm_subscription.current.id

  amount     = 1000
  time_grain = "Monthly"

  notification {
    enabled   = true
    threshold = 80
    operator  = "GreaterThan"

    contact_emails = ["admin@company.com"]
  }
}
```

---

## 📋 Cost Checklist

### Before Deployment
- [ ] Use Azure Pricing Calculator for estimates
- [ ] Choose appropriate regions (price varies by location)
- [ ] Select correct VM sizes and storage tiers
- [ ] Plan for Reserved Instances for predictable workloads
- [ ] Configure auto-shutdown for dev/test resources

### During Deployment
- [ ] Implement tagging strategy for cost allocation
- [ ] Set up budget alerts and spending limits
- [ ] Configure auto-scaling where appropriate
- [ ] Use managed services to reduce operational overhead

### After Deployment
- [ ] Monitor costs weekly using Cost Management
- [ ] Review Azure Advisor recommendations monthly
- [ ] Optimize storage using lifecycle policies
- [ ] Right-size resources based on utilization metrics
- [ ] Consider Reserved Instances for stable workloads

---

## 📞 Cost Support Resources

- **Azure Cost Management Documentation**: [Link](https://docs.microsoft.com/en-us/azure/cost-management-billing/)
- **Azure Pricing**: [Link](https://azure.microsoft.com/en-us/pricing/)
- **Azure Cost Optimization Guide**: [Link](https://docs.microsoft.com/en-us/azure/architecture/framework/cost/optimize-checklist)
- **Azure Well-Architected Framework - Cost**: [Link](https://docs.microsoft.com/en-us/azure/architecture/framework/cost/)

---

**Last Updated**: 2024
**Pricing Region**: US East 2
**Currency**: USD

> 💡 **Pro Tip**: Costs can vary significantly based on usage patterns, regions, and commitment levels. Always validate estimates with the official Azure Pricing Calculator and monitor actual usage regularly.
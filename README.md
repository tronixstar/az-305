# AZ-305 TECHNICAL REFERENCE GUIDE
## Designing Microsoft Azure Infrastructure Solutions

**Exam Weight Distribution**: Identity/Governance/Monitoring (25-30%) | Data Storage (25-30%) | Business Continuity (10-15%) | Infrastructure (30-35%)

---

## DOMAIN 1: IDENTITY, GOVERNANCE & MONITORING

### Microsoft Entra ID Authentication Methods

| Method | MFA Security Score | Phishing Resistant | Device Required | Offline Capable | Ideal Scenario |
|--------|-------------------|-------------------|-----------------|-----------------|----------------|
| Password + SMS | Medium | No | Mobile phone | No | Legacy compatibility |
| Password + Authenticator App | High | No | Smartphone | Yes (TOTP) | Standard MFA |
| Passwordless - Windows Hello | Very High | Yes | Windows 10+ device | Yes | Corporate Windows devices |
| Passwordless - FIDO2 | Very High | Yes | Security key | Yes | Cross-platform, high security |
| Passwordless - Authenticator | High | Partial | Smartphone | No (push) | Mobile-first workforce |
| Certificate-based | Very High | Yes | PKI infrastructure | Yes | Smart card environments |

### Hybrid Identity Synchronization Comparison

| Feature | Password Hash Sync (PHS) | Pass-Through Auth (PTA) | AD FS |
|---------|-------------------------|------------------------|-------|
| **Password Location** | Hash in cloud | On-premises only | On-premises only |
| **Authentication Location** | Cloud | On-premises (via agent) | On-premises (federation) |
| **High Availability** | Built-in | Requires 2+ agents | Requires WAP + multiple servers |
| **Latency** | Lowest | Medium | Highest |
| **Complexity** | Low | Medium | High |
| **Smart Card Support** | No | No | Yes |
| **On-Prem MFA Support** | No | Yes | Yes |
| **Password Expiry Enforcement** | No | Yes | Yes |
| **Microsoft Recommendation** | ✅ Primary | Fallback | Legacy only |

### Conditional Access Policy Components

**Grant Controls**:
- Require MFA
- Require device to be marked as compliant
- Require Hybrid Azure AD joined device  
- Require approved client app
- Require app protection policy
- Require password change

**Session Controls**:
- Use app enforced restrictions
- Use Conditional Access App Control
- Sign-in frequency (persistent browser session)
- Persistent browser session

**Policy Evaluation Flow**: User/Group → Cloud App → Conditions (Location, Device, Risk) → Access Decision → Session Control

### Azure RBAC Built-In Roles - Complete Comparison

| Role | Can Assign Roles | Create/Modify Resources | Delete Resources | Read Resources | Specific Limitations |
|------|-----------------|------------------------|------------------|---------------|---------------------|
| Owner | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes | Full control |
| Contributor | ❌ No | ✅ Yes | ✅ Yes | ✅ Yes | Cannot assign roles/manage locks |
| Reader | ❌ No | ❌ No | ❌ No | ✅ Yes | Read-only access |
| User Access Administrator | ✅ Yes | ❌ No | ❌ No | ✅ Yes | RBAC only |
| Network Contributor | ❌ No | ✅ Network only | ✅ Network only | ✅ Yes | VNets, NSGs, Load Balancers |
| Storage Account Contributor | ❌ No | ✅ Storage only | ✅ Storage only | ✅ Yes | Storage accounts only |
| Storage Blob Data Owner | ❌ No | ✅ Blob data | ✅ Blob data | ✅ Yes | Full blob access + ACLs |
| Storage Blob Data Contributor | ❌ No | ✅ Blob data | ✅ Blob data | ✅ Yes | No ACL management |
| Key Vault Administrator | ❌ No | ✅ KV objects | ✅ KV objects | ✅ Yes | All KV operations |

### Azure Policy Effects

| Effect | Behavior | Use Case | Remediation |
|--------|----------|----------|-------------|
| **Deny** | Block non-compliant resource creation | Prevent creation | N/A (preventive) |
| **Audit** | Log non-compliant resources | Visibility | Manual |
| **AuditIfNotExists** | Audit if related resource missing | Check configurations | Manual |
| **DeployIfNotExists** | Deploy resource if missing | Auto-deploy extensions | Auto (with MI) |
| **Modify** | Add/update/remove tags/properties | Tag enforcement | Auto (with MI) |
| **Disabled** | Policy inactive | Testing | N/A |
| **Append** | Add additional properties | Add tags to resources | N/A (creation time) |

**Remediation Requirements**:
- Managed Identity with appropriate permissions
- Manual trigger for existing resources
- Automatic for new resources (Deny, Append, Modify)

### Azure Monitor Diagnostic Settings

**Maximum**: 5 diagnostic settings per resource

**Destination Options**:
| Destination | Retention | Query Capability | Cost Model | Use Case |
|-------------|-----------|------------------|------------|----------|
| Log Analytics | 30-730 days | KQL queries | Per GB ingested + retention | Active querying, alerting |
| Storage Account | Unlimited | No (archive only) | Storage tier pricing | Long-term archival, compliance |
| Event Hub | Real-time stream | No | Per throughput unit | SIEM integration, real-time |
| Partner Solutions | Varies | Partner-dependent | Partner pricing | Splunk, Datadog, Elastic |

### Access Reviews Configuration

| Review Type | Minimum Frequency | Reviewers | Auto-Apply Results | Fallback Reviewer |
|-------------|------------------|-----------|-------------------|------------------|
| Group Membership | Weekly | Group owners, Members, Managers | Yes | Specified user |
| Application Access | Weekly | App owners, Members, Managers | Yes | Specified user |
| Azure AD Roles | Weekly | Global Admin, Privileged Role Admin | Yes | Required |
| Azure Resources | Weekly | Resource owners | Yes | Subscription admins |
| Guest Access | Weekly | Sponsor, Self-review | Yes | Tenant admin |

**Auto-Actions**:
- Remove user access if denied
- Remove user access if no response
- Remove user access if inactive
- Apply recommendations

---

## DOMAIN 2: DATA STORAGE SOLUTIONS

### Azure SQL Database Tier-Specific Features

| Feature | Basic | Standard | Premium (DTU) | Gen Purpose (vCore) | Business Critical (vCore) | Hyperscale (vCore) |
|---------|-------|----------|---------------|--------------------|--------------------------|--------------------|
| **In-Memory OLTP** | ❌ | ❌ | ✅ | ❌ | ✅ | Partial |
| **Columnstore Index** | ❌ | S3+ only | ✅ | ✅ | ✅ | ✅ |
| **Max IOPS** | 5 | 3,000 | 48,000 | 7,000 (80 vCores) | 204,800 (80 vCores) | 327,680 |
| **Storage Type** | Remote | Remote | Remote | Remote SSD | Local SSD | Multi-tier (page servers) |
| **Read Replicas** | ❌ | ❌ | ❌ | Via geo-replication | ✅ Up to 4 built-in | ✅ Up to 4 HA + named |
| **Max Database Size** | 2 GB | 1 TB | 4 TB | 4 TB | 4 TB | 100 TB |
| **Backup Retention** | 7 days | 7-35 days | 7-35 days | 7-35 days | 7-35 days | 7-35 days |
| **Point-in-Time Restore** | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ (7 days+) |
| **Zone Redundancy** | ❌ | ❌ | ✅ (optional) | ✅ (optional) | ✅ (optional) | ❌ |
| **Local Data Redundancy** | No | 3 replicas | 3 replicas | 3 replicas | 4 replicas (1 read) | Multiple page servers |
| **RPO/RTO (HA)** | Minutes/Minutes | Seconds/Minutes | Seconds/30 sec | Seconds/Minutes | 0 (sync)/Seconds | Seconds/Minutes |
| **TDE** | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Always Encrypted** | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |

**In-Memory OLTP Performance**:
- 9x-11x transaction throughput improvement for OLTP workloads
- Up to 30x reduced latency for transaction processing
- 1.4M sustained inserts per second (P15 tier with optimized design)

**Columnstore Index Compression**:
- Up to 10x compression ratio
- Up to 10x query performance for aggregations
- 9x performance gain on P2, 57x on P15

**Hyperscale Unique Features**:
- Fast database copy (minutes vs hours)
- Fast backup (file-snapshot based, seconds)
- Fast restore (minutes vs hours)
- Named replicas for read scale-out
- Support for up to 1000s of concurrent users

### SQL Database Purchasing Model Decision Matrix

| Scenario | Recommended Model | Rationale |
|----------|------------------|-----------|
| Predictable, steady workload | DTU | Simple pricing, bundled resources |
| Need compute/storage independence | vCore General Purpose | Flexibility in scaling |
| High-performance OLTP, < 4 TB | vCore Business Critical | Local SSD, read replicas, low latency |
| Database > 4 TB | vCore Hyperscale | Up to 100 TB support |
| Intermittent usage, dev/test | vCore Serverless | Auto-pause, per-second billing |
| High concurrent users (200-300) | Hyperscale | Not limited to 128 like Synapse |
| Analytics, < 128 concurrent queries | Azure Synapse Analytics | Columnar, MPP architecture |

### Synapse vs Hyperscale - Critical Distinction

| Aspect | Hyperscale | Synapse Analytics |
|--------|-----------|-------------------|
| **Architecture** | SMP (single node processes query) | MPP (distributed compute) |
| **Workload** | OLTP (transactional) | OLAP (analytical) |
| **Query Pattern** | Point lookups, updates, inserts | Scans, aggregations, joins on large datasets |
| **Concurrent Users** | 1000s | 128 max concurrent queries |
| **Max DB Size** | 100 TB | Unlimited (petabytes) |
| **Storage Format** | Row-based (B-tree indexes) | Column-based (columnstore) |
| **DML Performance** | Excellent | Moderate (batch optimized) |
| **Analytics Performance** | Good | Excellent (10-100x vs row-store) |
| **Use Case** | Large OLTP database, SaaS, multi-tenant | Data warehousing, BI, reporting |

**Exam Trap**: "Support 200-300 concurrent read operations, 50 TB data" → **Hyperscale** (not Synapse - which has 128 concurrent query limit)

### Azure Storage Performance Tiers

| Tier | IOPS (per GB) | Throughput | Latency | Price (relative) | Use Case |
|------|--------------|------------|---------|------------------|----------|
| **Premium SSD (P80)** | 20,000 | 900 MBps | < 2ms | Highest | Mission-critical, low latency |
| **Premium SSD v2** | 80,000 max | 1,200 MBps | < 1ms | High + performance units | Predictable performance |
| **Standard SSD** | 6,000 max | 750 MBps | <10ms | Medium | General purpose |
| **Standard HDD** | 2,000 max | 500 MBps | Variable | Lowest | Infrequent access, backup |

**Premium SSD v2 Advantages**:
- Per-GB IOPS/throughput configuration
- No need to overprovision capacity
- Adjust performance independently of capacity

### Blob Storage Tier Decision Matrix

| Data Access Pattern | First 30 Days | 30-90 Days | 90-180 Days | 180+ Days |
|--------------------|---------------|------------|-------------|-----------|
| **Frequent (daily)** | Hot | Hot | Hot | Cool* |
| **Regular (weekly)** | Hot | Cool | Cool | Cold |
| **Occasional (monthly)** | Cool | Cool | Cold | Archive |
| **Rare (quarterly)** | Cool | Cold | Archive | Archive |
| **Compliance only** | Cold/Archive | Archive | Archive | Archive |

*Use lifecycle management for automatic tiering

**Early Deletion Cost Calculation**:
```
Cool tier (30 days minimum):
- Upload 100 GB on day 1
- Delete on day 10
- Charged for: 100 GB × 20 days remaining = pro-rated cost

Archive tier (180 days minimum):
- Upload 1 TB on Jan 1
- Delete on March 1 (60 days)
- Charged for: 1 TB × 120 days remaining = substantial penalty
```

### Data Lake Gen2 Requirements Matrix

| Requirement | Need Data Lake Gen2? | Alternative |
|-------------|---------------------|-------------|
| Hierarchical folders (directories) | ✅ Yes | Flat namespace insufficient |
| POSIX ACLs (file/folder permissions) | ✅ Yes | No alternative |
| Hadoop compatibility | ✅ Yes | Blob + Hadoop not optimized |
| Azure Databricks credential passthrough | ✅ Yes | No alternative |
| NFS v3.0 protocol | ✅ Yes | No alternative |
| Big data analytics (Spark, HDInsight) | ✅ Yes | Can use blob, but not optimized |
| Simple blob storage | ❌ No | Use standard blob storage |

**Cannot Change After Creation**:
- Hierarchical namespace setting (enabled/disabled)
- Once enabled, cannot be disabled
- Once disabled (standard blob), cannot enable on existing account

### Cosmos DB Consistency vs Performance Trade-offs

| Consistency | Read Latency | Write Latency | Availability | RU Cost | Data Staleness | Use Case |
|-------------|--------------|---------------|--------------|---------|----------------|----------|
| **Strong** | Higher (2 regions) | Higher (quorum) | Lower (outage impact) | 2x | None | Financial transactions, inventory |
| **Bounded Staleness** | Medium | Medium | Medium | 2x | K versions or T seconds | Stock prices, leaderboards |
| **Session** | Low | Low | High | 1x | Read-your-writes only | Shopping carts, user profiles |
| **Consistent Prefix** | Low | Low | High | 1x | Ordered, but delayed | Social feeds, comments |
| **Eventual** | Lowest | Lowest | Highest | 1x | Unpredictable | Caching, telemetry, non-critical |

**Multi-Region Writes**:
- Strong: Not supported
- Bounded Staleness: Supported (most common choice)
- Session/Prefix/Eventual: Supported

**Exam Scenario Mapping**:
- "Zero data loss, synchronous replication" → **Strong** (but expensive, 2x RU)
- "Multi-region writes, low latency" → **Bounded Staleness** or **Session**
- "User sees their own updates immediately" → **Session** (default, recommended)
- "Telemetry data, highest throughput" → **Eventual**

### Cosmos DB Partition Key Selection - Performance Impact

| Partition Key Type | Throughput | Hot Partitions | Example | Result |
|-------------------|------------|----------------|---------|--------|
| **High Cardinality, Even Distribution** | Excellent | No | UserID, DeviceID, TenantID | ✅ Ideal |
| **Low Cardinality** | Poor | Yes | Country (if imbalanced), Boolean | ❌ Avoid |
| **Time-based (Date)** | Poor | Yes (current date) | CreatedDate, Timestamp | ❌ Avoid |
| **Synthetic Composite** | Excellent | No | TenantID + UserID, Hash(Email) | ✅ Good |
| **Hot Value (Most Writes)** | Poor | Yes | Status="Active" getting 90% of writes | ❌ Avoid |

**Partition Limits (Know These)**:
- Logical partition max: 20 GB
- Physical partition max: 50 GB
- Throughput per logical partition: 10,000 RU/s
- Cannot split logical partitions (design carefully!)

---

## DOMAIN 3: BUSINESS CONTINUITY

### Azure Backup Agent/Extension Comparison

| Method | Backs Up | Platform | App-Consistent | Cross-Region | Use Case |
|--------|----------|----------|----------------|--------------|----------|
| **MARS Agent** | Files, folders, system state | Windows only | File-level only | Yes (vault replication) | Simple file backup |
| **Azure VM Extension** | Entire VM | Windows/Linux Azure VMs | Yes (VSS/pre-post scripts) | Yes | Azure VM backup |
| **MABS** | VMs, SQL, Exchange, SharePoint, Files | Windows Server, Hyper-V, VMware | Yes (app-aware) | Yes | Complex on-prem workloads |
| **DPM** | Same as MABS | Windows Server | Yes | Yes | Existing DPM customers |
| **Azure Backup Server for SQL** | SQL databases | SQL Server on Azure VM | Yes | Yes | SQL-specific backup |

### SQL Database Backup Granularity

| Backup Type | Frequency | Retention Default | Retention Max | RPO | Use Case |
|-------------|-----------|------------------|---------------|-----|----------|
| **Full** | Weekly | 7-35 days | 10 years (LTR) | 7 days | Baseline |
| **Differential** | Every 12-24 hours | 7-35 days | N/A | 12-24 hours | Incremental |
| **Transaction Log** | Every 5-10 minutes | 7-35 days | N/A | 5-10 minutes | PITR |
| **Long-Term (LTR)** | Weekly/Monthly/Yearly | Up to 10 years | 10 years | N/A | Compliance |

**Point-in-Time Restore (PITR)**:
- Can restore to any second within retention period
- Retention: 7 days (default), can configure 1-35 days
- Restored database is a new database (doesn't overwrite existing)

**Geo-Restore**:
- Restore from geo-replicated backup
- Any Azure region
- RPO: < 1 hour
- RTO: < 12 hours (typically much faster)

### Storage Redundancy - Detailed Comparison

| Redundancy | Copies | Sync/Async | Durability | Read Access | Failover Type | Scenarios Protected Against |
|------------|--------|------------|------------|-------------|---------------|----------------------------|
| **LRS** | 3 | Sync | 11 nines | Primary only | N/A | Disk, rack failure |
| **ZRS** | 3 | Sync | 12 nines | Primary only | N/A | Datacenter failure |
| **GRS** | 6 (3+3) | Async to secondary | 16 nines | Primary only | Microsoft-initiated | Region failure |
| **RA-GRS** | 6 (3+3) | Async to secondary | 16 nines | Primary + secondary (read) | Customer-initiated | Region failure + read scaling |
| **GZRS** | 6 (ZRS+LRS) | Async to secondary | 16 nines | Primary only | Microsoft-initiated | Datacenter + region failure |
| **RA-GZRS** | 6 (ZRS+LRS) | Async to secondary | 16 nines | Primary + secondary (read) | Customer-initiated | Datacenter + region failure + read scaling |

**Failover Characteristics**:
- **GRS/GZRS**: Microsoft-initiated only (rare, major disaster)
- **RA-GRS/RA-GZRS**: Customer-initiated or Microsoft-initiated
- **Failover Time**: 15 minutes to hours (varies)
- **RPO**: ~15 minutes (async replication lag)

**Premium Storage Limitation**:
- LRS and ZRS only
- No GRS/RA-GRS/GZRS support
- Design for HA with Availability Zones + ZRS

### Azure Site Recovery Configuration

| Source | Target | RTO | RPO | Network | Use Case |
|--------|--------|-----|-----|---------|----------|
| **Azure VM → Azure Region** | Different region | ~2 hours | < 15 min (app-consistent: 60 min) | VNet-to-VNet | Region DR |
| **On-prem VMware → Azure** | Azure VMs | ~2 hours | < 15 min | VPN/ExpressRoute | Datacenter DR |
| **On-prem Hyper-V → Azure** | Azure VMs | ~2 hours | 5-15 min | VPN/ExpressRoute | Datacenter DR |
| **Physical Servers → Azure** | Azure VMs | ~2 hours | 5-15 min | VPN/ExpressRoute | Legacy systems DR |
| **AWS/GCP VM → Azure** | Azure VMs | ~2 hours | 5-15 min | Internet/VPN | Cloud migration + DR |

**ASR Replication Frequency**:
- Azure-to-Azure: Continuous replication (~5 seconds)
- VMware: Continuous replication
- Hyper-V: 30 seconds, 5 minutes, or 15 minutes

---

## DOMAIN 4: INFRASTRUCTURE SOLUTIONS

### Data Migration Tools - Detailed Thresholds

| Data Size | Network Bandwidth | Timeline | Recommended Solution | Cost Consideration |
|-----------|------------------|----------|---------------------|-------------------|
| **< 10 GB** | Any | Hours | AzCopy, Portal Upload, Storage Explorer | Network egress |
| **10-100 GB** | > 25 Mbps | Hours-Days | AzCopy, Azure CLI, Storage Explorer | Network egress |
| **100 GB - 1 TB** | > 100 Mbps | Days | Azure Data Factory, AzCopy (parallel) | Network + ADF pipeline costs |
| **1-10 TB** | < 1 Gbps | Weeks | **Data Box Disk** (8 TB per order, 35 TB total) | Device rental (~$200) |
| **10-40 TB** | < 1 Gbps or unreliable | 7-10 days | **Data Box** (80 TB usable per device) | Device rental (~$300) |
| **40-500 TB** | < 1 Gbps | 2-3 weeks | **Data Box Heavy** (770 TB usable) | Device rental (~$1000) |
| **> 500 TB** | > 1 Gbps | Ongoing | Azure Data Factory + ExpressRoute | ExpressRoute + egress costs |
| **Continuous sync** | Any | Ongoing | **Data Box Gateway** (virtual appliance) | Gateway + egress costs |
| **Hybrid (bulk + incremental)** | Variable | Mixed | Data Box (seed) + Data Box Gateway (delta) | Combined costs |

**Data Box Family Detailed Comparison**:

| Device | Usable Capacity | Physical Form | Interfaces | Transfer Speed | Weight | Encryption | Use Case |
|--------|----------------|---------------|------------|----------------|--------|------------|----------|
| **Data Box Disk** | 8 TB (per disk), 5 disks max (40 TB) | SSD disk | USB 3.1 | Up to 430 MBps | < 1 lb per disk | 128-bit AES | Small datasets, multiple locations |
| **Data Box** | 80 TB | Ruggedized appliance | 10 GbE, 1 GbE | Up to 80 GBps (10 GbE) | ~50 lbs | 256-bit AES | Medium datasets, datacenter |
| **Data Box Heavy** | 770 TB | Wheeled appliance | 4x 40 GbE, 1 GbE | Up to 400 GBps | ~500 lbs | 256-bit AES | Large datasets, single location |
| **Data Box Gateway** | Unlimited (cloud gateway) | Virtual appliance (Hyper-V/VMware) | Virtual NICs | Network-dependent | N/A (VM) | TLS + storage encryption | Continuous ingestion, hybrid |

**Data Box Gateway Features**:
- Virtual appliance (runs on-premises in hypervisor)
- Continuous data ingestion to cloud
- Local cache (50-60% threshold before cloud upload)
- SMB/NFS protocol support
- Automatic tiering to cloud storage
- No capacity limit (uses cloud storage)

**Migration Strategy Selection**:
```
Data Size < 1 TB AND Good Network? → AzCopy/ADF
Data Size < 1 TB AND Poor Network? → Data Box Disk

Data Size 1-40 TB AND Poor Network? → Data Box
Data Size 1-40 TB AND Good Network? → ADF + ExpressRoute

Data Size 40-500 TB AND Poor Network? → Data Box Heavy  
Data Size 40-500 TB AND Good Network? → ADF + ExpressRoute (or Data Box for initial seed)

Data Size > 500 TB? → Hybrid approach:
  1. Data Box Heavy for initial bulk
  2. Data Box Gateway for ongoing delta
  3. Or ExpressRoute + ADF if > 1 Gbps network
```

### ExpressRoute SKU Comparison

| Feature | Local | Standard | Premium | Direct (10 Gbps) | Direct (100 Gbps) |
|---------|-------|----------|---------|------------------|-------------------|
| **Regional Access** | Single metro area | Geopolitical region | Global | Global | Global |
| **VNet Connections** | 10 | 10 | 100 | Unlimited circuits | Unlimited circuits |
| **Route Advertisement Limit** | 4,000 | 4,000 | 10,000 | 10,000 per circuit | 10,000 per circuit |
| **Bandwidth Options** | 50 Mbps - 10 Gbps | 50 Mbps - 10 Gbps | 50 Mbps - 10 Gbps | 10 Gbps port pair | 100 Gbps port pair |
| **BGP Communities** | Limited | Yes | Yes | Yes | Yes |
| **Office 365 Support** | No | No | Yes | Yes | Yes |
| **Dynamics 365 Support** | No | No | Yes | Yes | Yes |
| **ExpressRoute Global Reach** | No | Add-on | Included | Included | Included |
| **Private Peering** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Microsoft Peering** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Public Peering** | ❌ Deprecated | ❌ Deprecated | ❌ Deprecated | ❌ | ❌ |
| **Data Egress Charges** | Free (within region) | Metered or Unlimited plan | Metered or Unlimited plan | Metered or Unlimited | Metered or Unlimited |
| **Monthly Port Fee** | Included in circuit | Included in circuit | Additional fee | ~$5,000-8,500/month | ~$51,300/month |
| **FastPath Support** | No | Yes | Yes | Yes | Yes |

**ExpressRoute Direct Unique Features**:
- Direct connection to Microsoft global network at peering location
- Dual 10 Gbps or 100 Gbps ports (Active/Active)
- Create multiple circuits on same port pair (up to bandwidth limit)
- Massive data ingestion (TB/PB scale)
- Physical isolation (compliance: banking, government)
- MACsec support (Layer 2 encryption)
- Rate limiting per circuit

**Circuit Bandwidth on ExpressRoute Direct**:
- 10 Gbps Direct: Can create multiple circuits totaling 20 Gbps (2x oversubscription)
- 100 Gbps Direct: Can create multiple circuits totaling 200 Gbps
- Examples: Five 2 Gbps circuits + Two 5 Gbps circuits on 10 Gbps port

**Exam Decision Tree - ExpressRoute**:
```
Need access to ALL Azure regions globally? → Premium
Access to single metro area only? → Local (cheapest)
Access to regions in same geo (e.g., all US regions)? → Standard

Need > 10 Gbps total bandwidth? → ExpressRoute Direct (10 or 100 Gbps)
Need physical isolation (compliance)? → ExpressRoute Direct
Need Layer 2 encryption (MACsec)? → ExpressRoute Direct

Connect multiple on-prem sites privately (bypass internet)? → Global Reach (add-on or included in Premium)
```

### Load Balancer vs Application Gateway vs Front Door vs Traffic Manager

| Feature | Load Balancer | Application Gateway | Front Door | Traffic Manager |
|---------|--------------|-------------------|------------|----------------|
| **OSI Layer** | Layer 4 (Transport) | Layer 7 (Application) | Layer 7 (Application) | Layer 3 (DNS) |
| **Scope** | Regional or Global (Cross-region LB) | Regional | Global | Global |
| **Protocol** | TCP, UDP (any) | HTTP, HTTPS, WebSocket | HTTP, HTTPS, WebSocket | Any (DNS-based) |
| **SSL Termination** | No | Yes | Yes | No (DNS-based) |
| **WAF** | No | Yes (optional) | Yes (optional) | No |
| **URL Path-Based Routing** | No | Yes | Yes | No |
| **Cookie Affinity** | No | Yes | Yes | No (DNS) |
| **Rewrite HTTP Headers** | No | Yes | Yes | No |
| **Health Probes** | TCP/HTTP | HTTP/HTTPS | HTTP/HTTPS | HTTP/HTTPS/TCP |
| **Routing Methods** | Hash-based, Source IP | Round robin, URL path, host header | Priority, weighted, latency, geographic | Priority, weighted, performance, geographic, subnet |
| **Backend Pool** | Azure VMs, VMSS, IPs | Azure VMs, VMSS, App Service, IPs | Any public endpoint, App Service, VMs | Any public endpoint |
| **Auto-scaling** | No (SKU-based capacity) | Yes (v2) | Yes | N/A (DNS-based) |
| **Zone Redundancy** | Yes (Standard SKU) | Yes (v2) | Yes | N/A (global service) |
| **Pricing Model** | Per rule + data processed | Per hour + capacity units | Per policy + data processed | Per DNS query + health checks |
| **WebSocket Support** | Yes (TCP pass-through) | Yes | Yes | N/A |
| **Multi-site Hosting** | No | Yes | Yes | No (single backend per endpoint) |
| **URL Rewrite** | No | Yes | Yes | No |
| **Connection Draining** | No | Yes | No (automatic) | N/A |

**Exam Decision Matrix - Load Balancing**:
```
Layer 4 load balancing (TCP/UDP, non-HTTP)? 
├─ Regional? → Load Balancer (Standard SKU)
└─ Cross-region? → Cross-region Load Balancer OR Traffic Manager

Layer 7 load balancing (HTTP/HTTPS)?
├─ Regional?
│   ├─ Need WAF? → Application Gateway (v2)
│   ├─ Path-based routing? → Application Gateway
│   └─ Simple HTTP load balancing? → Application Gateway
│
└─ Global?
    ├─ HTTP/HTTPS with WAF + SSL offload + session affinity? → Front Door
    ├─ Any protocol (including non-HTTP)? → Traffic Manager
    ├─ Need CDN + acceleration? → Front Door
    └─ Simple DNS failover? → Traffic Manager

Multi-region HA for web application:
├─ Need active-active with session affinity? → Front Door
├─ Need active-passive failover? → Traffic Manager (Priority)
└─ Need lowest latency routing? → Front Door (Performance) or Traffic Manager (Performance)
```

**Critical Exam Traps**:
1. "Global HTTP routing + WAF" → **Front Door** (NOT Traffic Manager - no WAF)
2. "Global, any protocol, DNS-based" → **Traffic Manager** (NOT Front Door - HTTP only)
3. "Regional web app, path-based routing" → **Application Gateway** (NOT Front Door - regional)
4. "TCP/UDP load balancing for VMs" → **Load Balancer** (NOT Application Gateway)
5. "Cookie-based session affinity, global" → **Front Door** (Traffic Manager is DNS-based, no cookie support)

### Azure VPN Gateway SKUs

| SKU | Max Throughput | Max Tunnels (S2S) | Max P2S Connections | BGP | Active-Active | Zone Redundant | Use Case |
|-----|----------------|-------------------|-------------------|-----|---------------|----------------|----------|
| **Basic** | 100 Mbps | 10 | 128 | ❌ No | ❌ No | ❌ No | Dev/test only |
| **VpnGw1** | 650 Mbps | 30 | 250 | ✅ Yes | ✅ Yes | ❌ No | Small production |
| **VpnGw2** | 1 Gbps | 30 | 500 | ✅ Yes | ✅ Yes | ❌ No | Medium production |
| **VpnGw3** | 1.25 Gbps | 30 | 1000 | ✅ Yes | ✅ Yes | ❌ No | High bandwidth |
| **VpnGw4** | 5 Gbps | 100 | 5000 | ✅ Yes | ✅ Yes | ❌ No | Very high bandwidth |
| **VpnGw5** | 10 Gbps | 100 | 10000 | ✅ Yes | ✅ Yes | ❌ No | Maximum bandwidth |
| **VpnGw1AZ** | 650 Mbps | 30 | 250 | ✅ Yes | ✅ Yes | ✅ Yes | Production + HA |
| **VpnGw2AZ** | 1 Gbps | 30 | 500 | ✅ Yes | ✅ Yes | ✅ Yes | Production + HA |
| **VpnGw3AZ** | 1.25 Gbps | 30 | 1000 | ✅ Yes | ✅ Yes | ✅ Yes | Production + HA |
| **VpnGw4AZ** | 5 Gbps | 100 | 5000 | ✅ Yes | ✅ Yes | ✅ Yes | High bandwidth + HA |
| **VpnGw5AZ** | 10 Gbps | 100 | 10000 | ✅ Yes | ✅ Yes | ✅ Yes | Maximum + HA |

**Generation 2 VPN Gateway**:
- IKEv2 support
- Higher performance per vCore
- Available for VpnGw2 and above

**VPN Gateway vs ExpressRoute Decision**:

| Requirement | VPN Gateway | ExpressRoute |
|-------------|-------------|--------------|
| < 10 Gbps bandwidth | ✅ VpnGw5AZ (10 Gbps) | ✅ Standard circuit |
| > 10 Gbps bandwidth | ❌ Max 10 Gbps | ✅ ExpressRoute Direct |
| Predictable latency | ❌ Internet-based | ✅ Private connection |
| Cost-sensitive | ✅ Lower cost | ❌ Higher cost |
| Quick setup (days) | ✅ Yes | ❌ Weeks (provider provisioning) |
| SLA > 99.9% | ✅ 99.95% (AZ SKU) | ✅ 99.95% |
| Private IP only (no Internet) | ✅ Tunnel | ✅ Private peering |
| Office 365/Dynamics 365 | ❌ Use Internet | ✅ Microsoft peering (Premium) |

### Messaging Services - Detailed Feature Comparison

| Feature | Queue Storage | Service Bus Queue | Service Bus Topic | Event Grid | Event Hubs |
|---------|---------------|-------------------|-------------------|------------|------------|
| **Max Message Size** | 64 KB | 256 KB (Std), 100 MB (Prem) | Same as queue | 1 MB | 1 MB |
| **Max Queue/Topic Size** | 500 TB (account limit) | 1-80 GB (Std), 1 TB (Prem) | Same | N/A | Varies by TU |
| **Retention** | 7 days max | 1-90 days | 1-90 days | 24 hours | 1-90 days |
| **Ordering** | No | FIFO (with sessions) | FIFO (with sessions) | No | Partition-level |
| **Duplicate Detection** | No | Yes | Yes | No | No (manual) |
| **Transactions** | No | Yes | Yes | No | No |
| **Dead Letter Queue** | No | Yes | Yes | Yes | No |
| **Message Sessions** | No | Yes | Yes | No | No (consumer groups) |
| **Scheduled Messages** | No | Yes | Yes | No | No |
| **Batch Operations** | Limited | Yes | Yes | Yes (batch publish) | Yes |
| **Auto-Forwarding** | No | Yes | Yes | No | No |
| **Auto-Delete on Idle** | No | Yes | Yes | N/A | N/A |
| **Throughput** | Moderate (20K msg/sec) | High | High | Very High (10M msg/sec) | Highest (MBps-GBps) |
| **Protocol** | REST | AMQP, HTTP | AMQP, HTTP | HTTP/HTTPS (webhook) | AMQP, Kafka |
| **Message Pattern** | Point-to-point | Point-to-point | Pub/Sub | Pub/Sub (push) | Streaming (pull) |
| **Max Subscribers** | 1 (queue) | 1 (queue) | 2000 (subscriptions) | Unlimited (webhooks) | Unlimited (partitions × consumers) |
| **Filtering** | No | No (manual) | Yes (SQL filters) | Yes (event type, subject) | No (manual) |
| **Price** | Lowest | Medium | Medium | Very Low | Medium-High |

**Service Bus Premium Features**:
- 1 TB storage (vs 1-80 GB Standard)
- 100 MB message size (vs 256 KB)
- VNet integration
- Geo-disaster recovery
- Dedicated resources (no noisy neighbor)

**Exam Scenarios - Messaging Selection**:

| Scenario | Solution | Rationale |
|----------|----------|-----------|
| Simple async processing, cost-sensitive, > 80 GB | Queue Storage | Cheapest, large capacity |
| FIFO order critical | Service Bus Queue + Sessions | Only option with ordering |
| One message → Multiple subscribers | Service Bus Topic | Pub/Sub pattern |
| Filter messages by properties | Service Bus Topic + Filters | SQL-based filtering |
| Transactions, exactly-once delivery | Service Bus | Transaction support |
| React to Azure resource changes | Event Grid | Event-driven, push-based |
| Millions of IoT events per second | Event Hubs | Streaming, highest throughput |
| Replay events, process historical data | Event Hubs (Capture) | Retention + capture to storage |
| Sub-second latency required | Service Bus Premium | Dedicated resources |

### Messaging Throughput and Limits

| Service | Throughput (Messages/sec) | Throughput (Data) | Scale Unit |
|---------|-------------------------|------------------|------------|
| **Queue Storage** | ~20,000 msg/sec | ~2,000 MBps (2 GBps) | Storage account |
| **Service Bus Standard** | ~2,000 msg/sec | ~200 MBps | Namespace (shared) |
| **Service Bus Premium** | ~10,000+ msg/sec | ~1,000+ MBps | Messaging Units (MU) - dedicated |
| **Event Grid** | ~10,000,000 events/sec | Varies | Automatic scaling |
| **Event Hubs Basic** | ~1 MB/sec | 1 MBps | Throughput Unit (TU) |
| **Event Hubs Standard** | ~1 MB/sec per TU | 1 MBps per TU | Up to 40 TUs (auto-inflate) |
| **Event Hubs Premium** | ~1 MB/sec per PU | 1 MBps per PU | Processing Units (PU) - dedicated |
| **Event Hubs Dedicated** | GB/sec range | GBps | Capacity Units (CU) |

### Azure Kubernetes Service (AKS) - Network Plugins

| Plugin | Pod IP Addressing | IP Consumption | Performance | Network Policy Support | Use Case |
|--------|------------------|----------------|-------------|----------------------|----------|
| **kubenet** | Internal (10.244.0.0/16 default) | Low (1 IP per node) | Good | Calico only | IP-constrained VNets |
| **Azure CNI** | VNet IPs | High (1 IP per pod) | Better | Azure Network Policy or Calico | Enterprise, full VNet integration |
| **Azure CNI Overlay** | Overlay network (separate CIDR) | Low (1 IP per node) | Best | Azure Network Policy or Calico | Large clusters, IP conservation |
| **Azure CNI Pod Subnet** | Dedicated pod subnet | Medium (separate subnet) | Better | Azure Network Policy or Calico | Organized IP management |

**IP Address Calculation**:
```
kubenet:
- Node IPs: 1 per node
- Pod IPs: Internal (non-VNet)
- Total VNet IPs: Number of nodes

Azure CNI:
- Node IPs: 1 per node  
- Pod IPs: Max pods per node (default 30)
- Total VNet IPs: Nodes × (MaxPods + 1)
- Example: 10 nodes × 31 = 310 IPs required

Azure CNI Overlay:
- Node IPs: 1 per node
- Pod IPs: Overlay network (not from VNet)
- Total VNet IPs: Number of nodes
```

**Exam Scenario**: "AKS cluster, 100 nodes, limited VNet IP space" → **kubenet** or **Azure CNI Overlay** (NOT Azure CNI - would need 3,100 IPs!)

---

## KEY SERVICE LIMITS AND QUOTAS

### Compute Limits

| Resource | Limit | Scope |
|----------|-------|-------|
| VMs per Availability Set | 200 | Per set |
| VMs per region (default) | 10,000 vCPUs | Per subscription |
| VMs per scale set | 1,000 (platform image, single placement) | Per VMSS |
| VMs per scale set | 600 (custom image, single placement) | Per VMSS |
| Managed Disks per subscription | 50,000 | Per region |
| Fault Domains (Availability Set) | 2-3 | Per region |
| Update Domains (Availability Set) | 20 max (default 5) | Per set |
| AKS clusters per subscription | 5,000 | Per region |
| AKS nodes per cluster | 5,000 | Per cluster |

### Networking Limits

| Resource | Limit | Scope |
|----------|-------|-------|
| VNets per subscription | 1,000 | Per region |
| Subnets per VNet | 3,000 | Per VNet |
| VNet peerings per VNet | 500 | Per VNet |
| Private endpoints per subscription | 1,000 | Per region |
| NSG rules per NSG | 1,000 | Per NSG |
| NSGs per subscription | 5,000 | Per region |
| Application Gateway instances | 125 | Per subscription |
| Front Door backends | 500 | Per profile |
| Traffic Manager endpoints | 200 | Per profile |
| ExpressRoute circuits | 50 | Per subscription |

### Storage Limits

| Resource | Limit | Notes |
|----------|-------|-------|
| Storage accounts per subscription | 250 | Per region |
| Max capacity per storage account | 5 PB | US/Europe, 500 TB others |
| Max blob size (Block blob) | 190.7 TB | 4.75 TB per block × 50,000 |
| Max blob size (Page blob) | 8 TB | |
| Max blob size (Append blob) | 195 GB | |
| Max request rate | 20,000 requests/sec | Per storage account |
| Max egress (US/Europe) | 50 Gbps | Per storage account |
| Blobs per container | Unlimited | |
| Containers per storage account | Unlimited | |

### Database Limits

| Resource | Limit | Tier/Notes |
|----------|-------|------------|
| Max DB size (Gen Purpose) | 4 TB | vCore |
| Max DB size (Business Critical) | 4 TB | vCore |
| Max DB size (Hyperscale) | 100 TB | vCore |
| Max vCores (Gen Purpose) | 128 | vCore |
| Max vCores (Business Critical) | 128 | vCore |
| Max concurrent workers (GP) | 6,300 | 80 vCores |
| Max concurrent workers (BC) | 6,300 | 80 vCores |
| Databases per elastic pool | 500 | |
| Synapse concurrent queries | 128 | Critical limit! |
| Cosmos DB throughput per partition | 10,000 RU/s | Logical partition |
| Cosmos DB partition size | 20 GB | Logical partition |

---

## SLA REFERENCE TABLE

| Service | Configuration | SLA | Monthly Downtime |
|---------|---------------|-----|------------------|
| **Compute** |
| Single VM (Premium SSD) | 1 instance | 99.9% | 43.8 min |
| VM + Availability Set | 2+ instances | 99.95% | 21.9 min |
| VM + Availability Zones | 2+ instances across zones | 99.99% | 4.38 min |
| AKS | Single node pool | 99.5% | 3.65 hours |
| AKS | Multiple node pools, zones | 99.95% | 21.9 min |
| App Service | Single instance | 99.95% | 21.9 min |
| Functions (Premium) | With zones | 99.95% | 21.9 min |
| **Database** |
| SQL Database (all tiers) | Standard config | 99.99% | 4.38 min |
| SQL MI (all tiers) | Standard config | 99.99% | 4.38 min |
| Cosmos DB (single region) | Single region | 99.99% | 4.38 min |
| Cosmos DB (multi-region) | Multi-region, single-master | 99.999% | 26.3 sec |
| Cosmos DB (multi-master) | Multi-region writes | 99.999% | 26.3 sec |
| **Storage** |
| Storage (LRS/ZRS) | - | 99.9%/99.99% (write) | 43.8/4.38 min |
| Storage (RA-GRS/RA-GZRS) | Read access | 99.99% (read) | 4.38 min |
| **Networking** |
| VPN Gateway | Standard SKU | 99.95% | 21.9 min |
| VPN Gateway | AZ SKU | 99.95% | 21.9 min |
| ExpressRoute | Any | 99.95% | 21.9 min |
| Load Balancer (Standard) | Zone-redundant | 99.99% | 4.38 min |
| Application Gateway v2 | 2+ instances | 99.95% | 21.9 min |
| Front Door | - | 99.99% | 4.38 min |
| Traffic Manager | - | 99.99% | 4.38 min |

---

## EXAM CRITICAL TRAPS & GOTCHAS

### Most Common Mistakes (by Frequency)

1. **RBAC Contributor Role**: Cannot assign roles (needs Owner or User Access Administrator)
2. **Data Lake Hierarchical Namespace**: Cannot be changed after storage account creation
3. **Availability Sets vs Zones**: Cannot use both; Sets = same datacenter, Zones = different datacenters
4. **Premium Storage Redundancy**: Only LRS and ZRS (no GRS/RA-GRS)
5. **Hyperscale vs Synapse**: Hyperscale = 1000s concurrent (OLTP), Synapse = 128 max (OLAP)
6. **VNet Peering**: NOT transitive (A-B + B-C ≠ A-C)
7. **Azure Policy Remediation**: Does NOT auto-fix existing resources (needs manual remediation task with MI)
8. **Front Door vs Traffic Manager**: FD = HTTP/HTTPS Layer 7, TM = Any protocol Layer 3 DNS
9. **Access Reviews**: Best for "auto-revoke if not approved" scenarios (not Automation or PIM alone)
10. **Managed Identity vs Service Principal**: Always prefer MI for Azure resources

### Scenario Keywords - Translation Guide

| Phrase in Question | Actual Meaning | Typical Answer Direction |
|-------------------|----------------|-------------------------|
| "Minimize cost" | Choose cheapest tier/option that meets requirements | DTU > vCore, Standard > Premium, LRS > GRS |
| "Minimize administrative effort" | Choose PaaS > IaaS, Managed services, Automation | SQL DB > SQL MI > SQL VM, Managed Identity > Service Principal |
| "Minimize latency" | Choose Premium tiers, Availability Zones, ExpressRoute | Business Critical, Premium Storage, ExpressRoute not VPN |
| "Ensure compliance" | Logging, Policy, Private Endpoints, Specific regions | Diagnostic Settings, Azure Policy, Private Endpoint |
| "Zero data loss" | Synchronous replication, Strong consistency | Business Critical + Failover Groups, Cosmos DB Strong |
| "Automatically" | Look for built-in automation features | Access Reviews (not scripts), Lifecycle Management, Auto-scaling |
| "Regional outage" | Need DR across regions | GRS/RA-GRS/GZRS, Active Geo-Replication, ASR |
| "Datacenter failure" | Need HA within region | Availability Zones, ZRS, Zone-redundant services |

---

*This is a technical reference document. For official exam registration and policies, visit Microsoft Learn.*

# Lab 9: Cloud Cost Estimate for ShopPro International

**Course:** CST8913 Cloud Migration 

**Student:** Dhruvansh Zala (041214130) 

**Date:** November 5, 2026

## 1. Introduction

ShopPro International is an online store that sells to people in North America, Europe and Asia-Pacific. The company wants to move its own servers to the cloud so the website stays fast, stays up, and can handle busy days.

In this lab I worked out how much the move will cost and how to keep the bill low. I picked **Microsoft Azure** because we studied it in class, and because Azure Hybrid Benefit lets a company reuse Windows Server and SQL Server licenses it already owns. That saves real money. I give a short comparison with AWS and GCP in Section 7.

**How I made the estimate**
- I priced each service per hour or per GB and multiplied it out. One month is 730 hours.
- I used normal pay-as-you-go list prices for East US and added 8% to VM prices because Europe and Asia-Pacific cost a bit more.
- These are estimates. Taxes, support plans and special company discounts are not included. I explain how to check them in the Azure Pricing Calculator in Section 8.

**What I assumed**

| Part | Assumption |
|---|---|
| Web servers | 10 VMs in each of 3 regions = 30 VMs (D4s_v5), always on. 15 extra VMs start up at busy times |
| API servers | 20 VMs (D8s_v5) run the 50 microservices. 10 extra start up at busy times |
| Busy time | About 20% of the month (around 146 hours) |
| Payments | 4 Windows VMs in a separate, locked-down network for PCI rules, plus a firewall and a key vault for encryption |
| Databases | 5 TB SQL (SQL Managed Instance), 10 TB NoSQL (Cosmos DB), 15 TB data warehouse (Synapse, paused 12 hours a day) |
| ML and batch jobs | 4 GPU VMs for 8 hours a day, 2 stronger GPU VMs for 100 hours a month, 4 batch VMs for 8 hours a day |
| Backup and disaster recovery | 40 TB of geo-redundant backup, Azure Backup for 74 VMs, Site Recovery for 20 important VMs |
| Traffic | 60 TB a month to customers, 20 TB a month between regions |
| Growth | 20% more traffic and data each year |

## 2. Cost Estimate Report

### 2.1 One-time migration cost

This is the cost to move everything over once.

| What | Cost |
|---|---|
| ExpressRoute 1 Gbps metered + gateway, 3 months | $1,725 |
| Replication staging storage (74 VMs, 1 month) | $947 |
| Database Migration Service and staging VMs (3 x D8s_v5, 2 months) | $1,682 |
| Data Box (30 TB seed) service fee | $250 |
| Parallel run: 1 month at 50% of steady-state operations | $23,208 |
| Azure Migrate and Server Migration (free first 180 days) | $0 |
| **Total** | **$27,812** |

Azure Migrate and the Server Migration tool are free for the first 180 days. The biggest cost is running the old and new systems side by side for one month. It costs more, but it is the safest way to move a payment system because we can switch back if something goes wrong.

### 2.2 Monthly operational cost

This is what we pay every month to run the system, before any savings.

| Group | What | Per month |
|---|---|---|
| Compute (VMs) | Web VMs baseline (30 x D4s_v5, 24x7) | $4,541 |
| Compute (VMs) | Web VMs scale-out (15 x D4s_v5, peak 20% of hours) | $454 |
| Compute (VMs) | API VMs baseline (20 x D8s_v5, 24x7) | $6,055 |
| Compute (VMs) | API VMs scale-out (10 x D8s_v5, peak 20% of hours) | $605 |
| Compute (VMs) | Payment VMs (4 x D8s_v5 Windows, PCI subnet) | $2,372 |
| Compute (VMs) | GPU training, T4 (4 x NC4as_T4_v3, 8 h/day) | $505 |
| Compute (VMs) | GPU training, A100 bursts (2 x NC24ads_A100_v4, 100 h) | $735 |
| Compute (VMs) | Batch processing (4 x D16s_v5, 8 h/day) | $737 |
| **Compute (VMs)** | **Subtotal** | **$16,004** |
| Storage | VM managed disks (74 x 128 GB Premium SSD) | $1,459 |
| **Storage** | **Subtotal** | **$1,459** |
| Databases | Primary SQL: Managed Instance GP 16 vCore (license included) | $2,920 |
| Databases | Primary SQL: 5 TB storage | $589 |
| Databases | Primary SQL: backup storage (geo-redundant, about 5 TB) | $1,024 |
| Databases | Analytics NoSQL: Cosmos DB 10 TB storage | $2,560 |
| Databases | Analytics NoSQL: Cosmos DB autoscale throughput (20,000 RU/s max, avg 50%) | $876 |
| Databases | Data warehouse: Synapse DW500c (paused 12 h/day) | $2,206 |
| Databases | Data warehouse: 15 TB storage | $345 |
| **Databases** | **Subtotal** | **$10,520** |
| Networking | Front Door Premium (base fee, CDN, WAF) | $330 |
| Networking | Front Door egress (60 TB/month, avg $0.055/GB) | $3,379 |
| Networking | Application Gateway WAF v2 (3 regions) | $1,290 |
| Networking | Standard load balancers (3 regions) + data | $157 |
| Networking | Azure Firewall Premium (payment zone) | $1,278 |
| Networking | Inter-region data transfer (20 TB) | $1,024 |
| Networking | VNet peering and private endpoints | $300 |
| **Networking** | **Subtotal** | **$7,758** |
| Backup and disaster recovery | Geo-redundant backup storage (40 TB, cool GRS) | $1,638 |
| Backup and disaster recovery | Azure Backup protected-instance fees (74 x $10) | $740 |
| Backup and disaster recovery | Site Recovery (20 critical VMs x $25) | $500 |
| Backup and disaster recovery | Disk snapshots and replicas | $300 |
| **Backup and disaster recovery** | **Subtotal** | **$3,178** |
| | **Total operational** | **$38,919** |

### 2.3 Monthly management cost

These are the tools that watch the system, keep it secure and control the budget.

| What | Per month |
|---|---|
| Log Analytics (50 GB/day ingestion) | $3,450 |
| Azure Monitor metrics and alerts | $200 |
| Defender for Cloud (servers, SQL, storage) | $1,510 |
| Managed HSM (PCI key management) | $2,336 |
| Cost Management, Policy, tagging | $0 |
| **Total management** | **$7,496** |

Cost Management, Azure Policy and tagging are free. I would tag every resource with things like Environment, Region, CostCenter and Workload so we can see who is spending what. Budget alerts at 80% and 100% warn us before we overspend.

### 2.4 Summary

| Cost | Amount |
|---|---|
| One-time migration | $27,812 |
| Monthly operational | $38,919 |
| Monthly management | $7,496 |
| **Total per month (pay-as-you-go)** | **$46,415** |
| **Total per year (pay-as-you-go)** | **$556,981** |

## 3. Cost Optimization Strategy

Here is how ShopPro can pay less without hurting performance.

| # | Idea | Saving per month | How it works |
|---|---|---|---|
| 1 | Auto-scaling (already in my estimate) | $4,238 compared to running peak size all month | The 25 extra VMs only run at busy times, not 24/7 |
| 2 | Azure Hybrid Benefit for Windows Server | $1,161 | We reuse licenses we already own, so we do not pay for them twice |
| 3 | Azure Hybrid Benefit for SQL Server | $1,460 (about 50%) | Same idea for the database license |
| 4 | Right-sizing | $1,748 (15% of web and API VM cost) | Azure Advisor shows VMs that are too big, and we move them to a smaller size |
| 5 | Auto-shutdown and pausing | $2,206 | The data warehouse is paused 12 hours a day. GPU, batch and test VMs only run on a schedule |
| 6 | 1-year reserved instances | $4,014 | About 35% off for servers and databases that run all the time |
| 7 | 3-year reserved instances | $6,204 | About 55% off the same things, but we commit for 3 years |

**Reserved or pay-as-you-go?** I would reserve only the things that run all day and will not change soon: the web VMs, API VMs, payment VMs and the main database. Things that run only a few hours (extra VMs, GPU training, batch jobs) stay pay-as-you-go, because a reservation would be paid for even while they sit idle.

**Monthly cost with each plan**

| Plan | Per month | Saved |
|---|---|---|
| A PAYG, no optimization | $46,415 | 0.0% |
| B AHB + right-sizing, PAYG | $42,046 | 9.4% |
| C B + 1-year reservations | $38,032 | 18.1% |
| D B + 3-year reservations | $35,842 | 22.8% |

**Other good habits**
- Tag everything and review the bill every month.
- Move old backups from hot to cool to archive storage, and delete what is expired.
- Use Front Door caching so fewer requests reach the servers, and keep services in the same region to avoid transfer fees.
- Use Dev/Test pricing and turn off test machines after work hours.
- Use Spot VMs for batch and ML jobs that can be restarted.
- Move to newer VM types and serverless (Azure Functions, Cosmos DB autoscale) over time.

## 4. Future Growth and Budget Plan (3 Years)

I assumed 20% growth every year. The optimized plan uses Hybrid Benefit, right-sizing and 3-year reservations for the Year 1 baseline. Extra growth is pay-as-you-go. In Year 3 I assumed another 10% saving from newer instance types and serverless.

| Year | Without optimization | With optimization | Saved |
|---|---|---|---|
| Year 1 | $556,981 | $430,110 | $126,871 (22.8%) |
| Year 2 | $638,293 | $507,226 | $131,067 (20.5%) |
| Year 3 | $735,868 | $539,789 | $196,079 (26.6%) |
| **3-year run total** | **$1,931,141** | **$1,477,124** | **$454,017 (23.5%)** |
| Migration (one time, Year 1) | $27,812 | $27,812 | |
| **3-year total** | **$1,958,953** | **$1,504,936** | **$454,017** |

**Budget plan**
- **Year 1:** about $457,921 (running cost plus migration). Keep 10% extra for surprises during cutover.
- **Year 2:** about $507,226. Check usage again and right-size.
- **Year 3:** about $539,789. Renew reservations and finish moving to newer types and serverless.
- Set budget alerts at 80%, 100% and 110% and check forecasts monthly.

**Risks:** growth higher than 20%, GPU prices changing, data transfer costs growing, and reservations going unused if we shrink. Monthly reviews and exchangeable reservations help with these.

## 5. Management and Monitoring Tools
- **Azure Monitor and Log Analytics:** alerts for uptime, speed and security.
- **Defender for Cloud:** security checks and PCI DSS compliance.
- **Cost Management and Budgets:** forecasts, alerts and cost by tag.
- **Azure Advisor:** tips for right-sizing and reservations.
- **Azure Policy:** makes sure resources have tags and use approved sizes.

## 6. Reliability and Performance
- Web servers are spread across availability zones in each region, behind Application Gateway and Front Door (load balancing, CDN and firewall).
- API servers auto-scale when sales or events bring more traffic.
- Geo-redundant backup and Site Recovery let us recover in another region if one goes down.
- The payment system is isolated, encrypted and logged for PCI DSS.

## 7. Azure, AWS and GCP

| Need | Azure | AWS | GCP |
|---|---|---|---|
| VMs | Virtual Machines | EC2 | Compute Engine |
| SQL database | SQL Managed Instance | RDS / Aurora | Cloud SQL |
| NoSQL | Cosmos DB | DynamoDB | Firestore / Bigtable |
| Data warehouse | Synapse | Redshift | BigQuery |
| CDN and load balancing | Front Door, App Gateway | CloudFront, ALB | Cloud CDN, Load Balancing |
| Migration tools | Azure Migrate, DMS | Migration Hub, DMS | Migrate to Virtual Machines |
| Reuse old licenses | Azure Hybrid Benefit | License Mobility (limited) | Bring your own license |
| Commitment discounts | Reservations, Savings Plans | Reserved Instances, Savings Plans | Committed Use Discounts |

I chose Azure because Hybrid Benefit gives the clearest savings for Windows and SQL Server. In a real project we would enter the same inputs in all three calculators and compare.

## 8. Pricing Calculator Screenshots

Add screenshots from the [Azure Pricing Calculator](https://azure.microsoft.com/en-ca/pricing/calculator/) here:

- [ ] Web and API VMs, pay-as-you-go vs reserved
- [ ] SQL Managed Instance with Hybrid Benefit
- [ ] Cosmos DB, Synapse and storage
- [ ] Front Door, Application Gateway and bandwidth

My totals are based on list prices, so the calculator may differ by a few percent.

## 9. Conclusion

Running ShopPro on Azure would cost about $46,415 a month ($556,981 a year) with pay-as-you-go, plus about $27,812 once to migrate. With Hybrid Benefit, right-sizing, scheduling and 3-year reservations, the monthly cost drops by about 23% to $35,842. Over three years the plan saves about $454,017 while still handling 20% growth each year.

## References
- Azure Pricing Calculator: https://azure.microsoft.com/en-ca/pricing/calculator/
- Azure Hybrid Benefit: https://learn.microsoft.com/en-us/azure/azure-sql/azure-hybrid-benefit
- Azure Reservations: https://learn.microsoft.com/en-us/azure/cost-management-billing/reservations/save-compute-costs-reservations
- Azure Cost Management: https://learn.microsoft.com/en-us/azure/cost-management-billing/costs/
- Azure Migrate: https://learn.microsoft.com/en-us/azure/migrate/migrate-services-overview

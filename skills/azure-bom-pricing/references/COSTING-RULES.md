# Costing and Architecture Rules

## Time

| Use | Hours/month |
|---|---:|
| Production or always-on service | 730 |
| Non-production service that can pause/stop/scale to zero | 310 |

Do not multiply a fixed monthly meter by hours.

## Compute

```text
monthly compute = hourly rate * active hours * instance quantity
```

For a non-production service that cannot pause:

- price 730 hours; or
- select a serverless/scale-to-zero alternative that meets the same functional and
  security requirements.

Never force 310 hours onto an always-billed plan.

## Azure SQL Database

Price compute and storage separately:

```text
monthly compute = selected compute meter * hours
monthly storage = GB provisioned * price per GB/month
monthly backup = chargeable backup GB * backup price per GB/month
```

- Provisioned vCore Azure SQL Database cannot be paused.
- Use General Purpose Serverless with auto-pause for suitable non-production databases.
- Hyperscale can be serverless but cannot auto-pause.
- Include zone-redundancy meters where the selected configuration bills them separately.
- Validate that the tier's SLA meets the availability target.
- For active geo-replication or failover groups, price the secondary database compute and
  storage; backup geo-restore alone is not low-RTO DR.

### Azure Hybrid Benefit for Azure SQL

The Retail Prices API separates General Purpose SQL compute and SQL licence meters.
When eligible SQL Server licences with Software Assurance are supplied:

- continue pricing the regional PAYG compute meter;
- remove the covered global SQL licence charge through AHUB;
- retain the SQL licence charge for uncovered vCores.

General Purpose SQL licence lookup:

```text
serviceName eq 'SQL Database'
and productName eq 'SQL Database Single/Elastic Pool General Purpose - SQL License'
and armRegionName eq 'Global'
```

Record the AHUB-covered licence charge as a zero-cost line for auditability. Validate
core-to-vCore eligibility and ensure the same licence pool is not assigned to multiple
simultaneous resources.

## App Service

```text
monthly = plan hourly price * 730 * plan instances
```

An App Service Plan does not stop billing when apps are stopped. Premium v3 may be
required for zone redundancy, Private Endpoints or VNet integration. Use the minimum
instance count required by the selected zone-redundancy feature.

## Functions

Consumption/Flex Consumption:

```text
execution cost = billable executions * rate per execution unit
compute cost = billable GB-seconds * rate per GB-second
```

Apply current free grants only when the selected meter and plan support them.

Elastic Premium:

```text
monthly = instances * 730 *
          (vCPU count * vCPU hourly rate + memory GiB * memory hourly rate)
```

Ask for executions, duration, memory and always-ready instances when absent.

## Storage

```text
capacity = stored GB * rate per GB/month
operations = operation count / meter unit * operation rate
retrieval = retrieved GB * retrieval rate
egress = billable outbound GB * egress rate
```

Price access tier and redundancy independently. Include operations and retrieval where
material. Do not double-count Front Door/CDN edge egress and origin egress.

## Front Door and CDN

Front Door uses traffic-zone rates, not Azure-region rates. Price:

- base fee;
- requests;
- edge egress;
- WAF or rules-engine dimensions when separately billed.

Ask for request volume and egress, or label conservative assumptions.

## Service Bus and Event Hubs

- Basic/Standard are usage based.
- Premium is generally per messaging/processing unit per hour.
- Private networking requirements may force Premium.
- Confirm throughput before assuming the minimum unit count.

## Monitoring

Price:

- ingestion;
- retention beyond included periods;
- query/analysis where separately billed;
- archive/search if required.

Treat implausibly low logging volume as a major assumption, particularly for regulated
production workloads.

## Fabric capacity and Power BI licensing

Fabric F SKUs are priced per capacity unit (CU). Multiply the live per-CU rate by the
capacity size:

```text
F8  = 8 CUs
F32 = 32 CUs
F64 = 64 CUs
monthly PAYG = CU count * active hours * per-CU hourly rate
```

Fabric capacity can be paused, so non-production may use 310 active hours when
pause/resume automation is assumed. Production normally uses 730 hours.

Capacity and user licensing are separate:

- F2-F32: Power BI publishers and consumers require Power BI Pro or PPU when Power BI
  content is in scope.
- F64+: eligible free-licensed viewers can consume Power BI content, while publishers
  still require Pro or PPU.
- PPU-only content/features can require PPU regardless of capacity.

Interpret `PowerBIProPPULicenses` as an owned-licence count: `0` means none owned, a
positive integer means that many licences are already owned, and blank means unknown.
Legacy `no` also means none owned. Ask for distinct named users and role/feature
requirements; do not sum environment user counts when the same people use both
environments.

Power BI licences are non-Azure costs and are not returned by the Azure Retail Prices
API. Price them from the current country-localised official Microsoft page described in
LICENSING-SOURCES.md:

```text
required licences = max(distinct named users requiring the licence - licences owned, 0)
monthly licence cost = required licences * current per-user/month list price
annual licence cost = monthly licence cost * 12
```

Keep the result in a separate non-Azure licence subtotal. Do not merge it into the Azure
subtotal, although a combined estimate can be shown.

## Egress

Build a traffic map:

1. source region/service;
2. destination region/internet;
3. monthly GB;
4. whether another service already bills the transfer.

Price internet, inter-region, inter-zone and Private Link processing separately when
applicable. Apply current free allowances only once.

## Backup

Use actual change rate and compression when supplied. Otherwise state assumptions.

For regulated workloads, distinguish:

- operational point-in-time restore;
- weekly/monthly/yearly long-term retention;
- geo-redundant restore;
- application-level DR.

Backup is not automatically equivalent to low-RTO disaster recovery.

## Security dependencies

When the user confirms private isolation, consider:

- Private Endpoints for supported PaaS services;
- VNet and subnets;
- Private DNS zones and query volume;
- VNet integration or equivalent for callers;
- Bastion, VPN, ExpressRoute or another confirmed admin path;
- jump box only when a managed/admin path is actually required;
- firewall/WAF/Defender services explicitly required by the security posture.

Price shared components once, not once per workload row.

## Confidence

| Confidence | Meaning |
|---|---|
| High | Exact live meter and user-supplied quantity |
| Medium | Exact live meter, reasonable sizing/usage assumption |
| Low | Proxy SKU, manual/conversion source, or material missing input |
| Unpriced | No trustworthy current price |

The total should separately identify Unpriced items so a number is never presented as
complete when material services are missing.

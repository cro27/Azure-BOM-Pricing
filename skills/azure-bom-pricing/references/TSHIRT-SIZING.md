# T-Shirt Pricing

## Purpose

T-shirt pricing provides directional Small, Medium and Large Azure PAYG retail scenarios
when an architecture or BOM identifies services but has no credible sizing, workload or
usage inputs.

It is not a capacity recommendation, formal quote or customer-ready estimate.

## Trigger and consent

Trigger this mode when:

- Azure services or architectural roles can be identified; and
- material capacity, transactions, throughput, users, storage or traffic are absent.

Prompt the user before using T-shirt sizing. Do not treat a bare architecture diagram or
service list as consent. T-shirt-mode consent is not confirmation of the default profiles.

If accepted:

1. ask for deployment region if missing;
2. resolve currency;
3. confirm production versus non-production scope;
4. confirm security, availability, backup and DR posture;
5. show the proposed driver and service-profile matrix, including every material assumption;
6. obtain explicit confirmation of that exact matrix before querying prices, regardless of
   mapping confidence.

If the user changes any driver, scenario, SKU, quantity, usage value or architectural mapping,
show the revised matrix and confirm it again. Do not interpret "yes, use T-shirt sizing" as
"yes, accept the defaults."

If declined, use the detailed intake workflow.

## Scenario meaning

| Size | Meaning |
|---|---|
| Small | Low-volume production baseline using the minimum valid architecture for the confirmed requirements |
| Medium | Moderate production workload using balanced capacity and headroom |
| Large | High-volume production workload using larger tiers or scale-out capacity |

Security, compliance, availability and DR requirements do not become weaker in Small.
They remain constant across sizes unless the user explicitly requests different posture.

By default, price all three sizes. A user may request a subset.

## Construction rules

- Build a valid architecture independently for each size.
- Select exact current PAYG `Consumption` meters for every scenario.
- Do not multiply the Small total to derive Medium or Large.
- Use supported SKU boundaries, minimum instance counts and tier-specific features.
- Price shared services once per scenario.
- Include monitoring, backup, transactions and egress.
- Do not introduce Dev, Test or DR environments unless present or confirmed.
- Keep architecture changes visible. For example, state when Medium changes SKU and Large
  adds instances.
- Assign Low confidence to assumed compute, capacity and throughput lines. Fixed
  architecture quantities may be Medium confidence when the service mapping is exact.

## Default workload drivers

Use these only when the user accepts T-shirt pricing, supplies no better values, and then
explicitly confirms the displayed matrix. Each value is an assumption and must appear in the
workbook.

| Driver | Small | Medium | Large |
|---|---:|---:|---:|
| Application requests/month | 1 million | 10 million | 100 million |
| Internet egress/month | 100 GB | 1 TB | 10 TB |
| Log ingestion/day | 1 GB | 5 GB | 25 GB |
| Primary stored data | 250 GB | 2 TB | 20 TB |
| Private Link data per direction/month | 100 GB | 1 TB | 10 TB |
| Read operations/month | 1 million | 10 million | 100 million |
| Write operations/month | 250,000 | 2.5 million | 25 million |
| Queue or event messages/month | 1 million | 10 million | 100 million |
| Serverless executions/month | 1 million | 10 million | 100 million |
| Average serverless memory | 0.5 GB | 1 GB | 2 GB |
| Average serverless duration | 1 second | 1 second | 2 seconds |
| Monthly backup change rate | 5% | 10% | 20% |

## Common service profiles

These are starting points, not universal recommendations. Validate regional availability
and every feature dependency.

| Service pattern | Small | Medium | Large |
|---|---|---|---|
| Always-on web/app compute | Minimum production tier and resilient instance count | Mid-tier or additional scale-out | Higher tier plus scale-out |
| Azure SQL Database General Purpose | 2 vCores, 128 GB | 8 vCores, 512 GB | 16 vCores, 1 TB |
| Azure Managed Redis Balanced | B1, two nodes | B5, two nodes | B20, two nodes |
| Service Bus Premium when private networking is required | 1 MU | 2 MUs | 4 MUs |
| Microsoft Fabric capacity | F2 | F16 | F64 |

For services not listed, choose the minimum valid production tier for Small, a balanced
supported tier for Medium and a materially larger supported tier or scale-out pattern for
Large. Document the selection logic.

AI model pricing requires a specified model and current official PAYG token meter. If the
model cannot be resolved, mark the model line Unpriced rather than selecting one.

## Resilience and security

Normal architecture rules still apply:

- confirm public, private or hybrid exposure;
- include Private Endpoints, Private DNS and integration dependencies when confirmed;
- apply the minimum instance count needed for zone resilience;
- distinguish backup from disaster recovery;
- include a secondary region only when confirmed;
- surface WAF, DDoS and administration-path gaps.

## Readiness and output

T-shirt mode is always **Indicative** because workload evidence is absent. Do not label it
Qualified or Customer-ready.

The workbook must include:

- a T-Shirt Sizing sheet with the profile matrix;
- a Scenario column in Cost Detail;
- separate Small, Medium and Large monthly and annual totals;
- service-level differences across sizes;
- every profile assumption and confidence;
- exclusions and the exact inputs needed to replace T-shirt assumptions.

The final response must state that the scenarios are directional PAYG retail estimates,
not capacity guarantees.

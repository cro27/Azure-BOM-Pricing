# Azure Retail Prices API Guide

## Source

Use:

```text
https://prices.azure.com/api/retail/prices
```

Request the user's currency through `currencyCode` and query with OData `$filter`.
The preview API may expose additional fields:

```text
api-version=2023-01-01-preview
```

Example:

```text
https://prices.azure.com/api/retail/prices?
currencyCode='AUD'&
$filter=serviceName eq 'Virtual Machines'
  and armRegionName eq 'australiaeast'
  and armSkuName eq 'Standard_D2s_v5'
```

URL-encode the query. Follow `NextPageLink` until empty.

## Tool order

1. If purpose-built `azure_price_lookup` or `azure_monthly_estimate` tools are available,
   use them for narrow quick lookups.
2. Use the Retail Prices API directly for full BOM work, broad meter discovery,
   pagination, or filtering by `productName`/`meterName`.
3. Never fall back to remembered prices. If no official current price can be retrieved,
   mark the line Unpriced.

## Meter selection

For each selected price, retain:

- `meterId`
- `serviceName`
- `productName`
- `skuName`
- `armSkuName`
- `meterName`
- `armRegionName`
- `location`
- `retailPrice`
- `unitPrice`
- `currencyCode`
- `unitOfMeasure`
- `type`
- `effectiveStartDate`
- the exact filter or request URL
- retrieval timestamp

Do not select a meter solely because its SKU looks correct. Check the product, meter,
unit and price type together. Use only currently effective rows where `type` is
`Consumption`; ignore commitment and savings-plan rows.

## Query strategy

1. Start with exact `armSkuName` for VMs and other ARM-addressable compute.
2. For service tiers, query `serviceName` plus `skuName`.
3. If no result, drop `skuName`, inspect the returned naming, then narrow.
4. For SQL, storage and other multi-meter services, select compute, capacity, operations,
   backup and egress separately.
5. If a result is duplicated, compare `effectiveStartDate`, type, product and meter.
   Select the currently effective row that matches the design.

### Private Link

Private Link is not exposed as `serviceName: "Azure Private Link"` and its standard
meters are not regional. Use:

```text
productName eq 'Virtual Network Private Link' and armRegionName eq 'Global'
```

The returned `serviceName` is `Virtual Network`. Select:

- `Standard Private Endpoint` (`1 Hour`);
- `Standard Data Processed - Ingress` (`1 GB`);
- `Standard Data Processed - Egress` (`1 GB`).

Data processing is billed per GB using progressive petabyte ranges. The API expresses
the PB boundaries as `tierMinimumUnits` in GB:

| Pricing-page range | API `tierMinimumUnits` | Apply the returned per-GB rate to |
|---|---:|---|
| First 1 PB/month | 0 | GB from 0 through 1,000,000 |
| Next 4 PB/month | 1,000,000 | GB above 1 PB through 5 PB |
| Above 5 PB/month | 5,000,000 | GB above 5 PB |

Use 1 PB = 1,000,000 GB for these billing bands. Apply each rate only to the usage in
its band; do not apply the lowest reached rate to the entire volume. Ingress and egress
have separate meters and must be calculated separately.

A query filtered to the deployment region (for example `australiaeast`) will not return
these global meters.

### Free and tiered meters

Some services return a zero-price allowance row and paid rows under the same meter ID.
Always inspect `tierMinimumUnits`:

- Log Analytics Analytics Logs ingestion currently exposes a free allowance tier before
  the paid ingestion tier.
- Internet bandwidth exposes the free allowance at tier 0 and the first paid rate at the
  next threshold.

Apply each tier progressively. Do not discard a zero-price row when the workload remains
inside its allowance, and do not select it for usage above the allowance.

## Currency

Ask the API for the required currency. Do not convert USD if the API can return the
target currency.

If an official service rate is absent from the API:

1. look for a current official Microsoft pricing source;
2. record its URL and retrieval date;
3. convert only if necessary using an explicit, dated exchange-rate source;
4. assign Low confidence;
5. otherwise leave the line Unpriced.

## PAYG retail scope

Use the API's public PAYG retail `Consumption` rows only. Azure Hybrid Benefit eligibility
and owned licences are not proven by the API and must be supplied by the user. The
workbook must state that its Azure subtotal uses public PAYG retail pricing.

## Reproducibility

Cache raw responses during the estimate so repeated calculations use the same price
snapshot. Keep the generated workbook's selected-meter lineage even if temporary cache
files are later removed.

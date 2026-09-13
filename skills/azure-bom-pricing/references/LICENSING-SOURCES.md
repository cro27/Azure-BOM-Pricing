# Non-Azure Microsoft Licensing Sources

## Power BI

Power BI Pro and Premium Per User are not Azure Retail Prices API meters. Retrieve their
current list price from Microsoft's country-localised Power BI pricing page:

```text
https://www.microsoft.com/<locale>/power-platform/products/power-bi/pricing
```

Examples:

- Australia: `https://www.microsoft.com/en-au/power-platform/products/power-bi/pricing`
- United States: `https://www.microsoft.com/en-us/power-platform/products/power-bi/pricing`

The price cards are dynamically rendered, so a plain HTTP text extraction may omit the
amounts. Use browser rendering when necessary. Run Playwright headlessly/in the
background by default when the runtime supports it; open a visible browser only when
authentication, consent or another user interaction is required.

Record:

- product: Power BI Pro or Power BI Premium Per User;
- displayed price and currency;
- `user/month` billing unit;
- payment cadence, such as `paid yearly`;
- whether tax is excluded;
- country/locale;
- page URL;
- retrieval timestamp.

Do not convert the US price when a localised page displays the target currency. Microsoft
states that displayed prices are marketing/list prices and the actual checkout price may
differ.

## Licensing logic for Fabric

- A Power BI Pro licence is required to publish Power BI content to all Fabric F SKUs.
- F2-F32: Power BI consumers also require a paid per-user licence.
- F64+: eligible free-licensed consumers can view and interact with Power BI content;
  publishers still require Pro or PPU.
- PPU workspaces or PPU-only features require PPU as applicable.

Count distinct named users, not concurrent users. Deduplicate people who use multiple
environments.

```text
licences to buy = max(distinct users requiring licence - licences already owned, 0)
```

Present three totals:

1. Azure subtotal — Retail Prices API meters only.
2. Non-Azure Microsoft licence subtotal.
3. Combined estimate.

Never label the combined amount as the Azure subtotal.

## Verified Australian baseline

On 24 August 2026, the official Australian page displayed:

- Power BI Pro: **AU$21.00 user/month, paid yearly**, excluding GST.
- Power BI Premium Per User: **AU$35.90 user/month, paid yearly**, excluding GST.

These values are a dated baseline, not a permanent hard-coded rate. Re-read the official
page for every new estimate.

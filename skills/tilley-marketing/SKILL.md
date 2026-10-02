---
name: tilley-marketing
description: Look up grain marketing in Tilley: crop profiles, sales and contracts, buyers, and position summary. Use when the user asks about cash sales, HTA, basis, futures contracts, buyers, or how much of the crop is sold.
---

# Tilley marketing

If `Marketing_PositionSummary_Search` is not in your tool list, stop and add the Tilley MCP at https://mcp.farmtilley.com/ (see the repo README). Then start at step 1.

## Rules

- A sale, bushel, price, or position figure enters the answer only after a Tilley tool returned it in this session. Name that tool.
- An empty result means nothing matched those filters.
- Keep the sale category the row returned: cash, HTA, basis, futures, or OTC. Do not relabel one category as another.
- Call `GetLookupOptions` before a `commodity_name` filter and pass a value from that list exactly.
- Signed-in producer: omit `producer_token` and `agent_token`. Agent for a client: pass that client's producer GUID. Never send `""`.
- Sale, buyer, and crop-profile create/update tools are not on this server. Say so if the user asks to book or edit a sale.
- `Futures` is a quote. It is not the producer's position.

## Workflow

### 1. Pick the question

| They asked about | Call |
| --- | --- |
| What is sold, or the position | `Marketing_PositionSummary_Search` with the marketing year, then `Marketing_PositionSummarySalesByCategory_Search` |
| Individual contracts | `Marketing_Sale_Select` |
| Buyers | `Marketing_Buyer_Search` |
| Crop profiles | `Marketing_CropProfile_Search`, then `tilley_CropProfile_Select` for full rows |
| Their own transaction history | `Marketing_UserTransactionHistory_Search` |
| Market transaction history | `Marketing_MarketTransactionHistory_Search` |
| A futures or options quote | `Futures`, after `GetLookupOptions` for commodity, type, practice, state, and county |

`Marketing_PositionSummary_Search` needs the marketing year.

**Done when** the matching tool has returned, or you have told the user the year is missing.

### 2. Answer

Lead with the marketing year and commodity. Show bushels and prices by the category on the row. Put the tool name beside each block.

Farm acres belong to `tilley-farm`. The enterprise budget belongs to `tilley-budget`.

**Done when** every number in the answer came from a tool result in this session, and no sale was relabeled.

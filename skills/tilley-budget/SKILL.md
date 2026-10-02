---
name: tilley-budget
description: Look up or update a Tilley enterprise budget: cost of production, line items, and line-item details. Use when the user asks about a budget, cost of production, input costs, or budget line items.
---

# Tilley budgets

If `Budget_EnterpriseBudget_Search` is not in your tool list, stop and add the Tilley MCP at https://mcp.farmtilley.com/ (see the repo README). Then start at step 1.

## Rules

- A budget figure enters the answer only after a Tilley tool returned it in this session. Name that tool.
- An empty result means nothing matched those filters.
- Call `GetLookupOptions` before commodity, type, practice, state, or county filters. Pass a value from that list exactly. Budget rows use `state_name` and `county_name` (the farm tools call the same place `location_state_name` and `location_county_name`).
- Signed-in producer: omit `producer_token` and `agent_token`. Agent for a client: pass that client's producer GUID. Never send `""`.
- Only an Enterprise Budget with `status` active feeds insurance, marketing, and ARC/PLC.
- Writes exist. Ask the user to confirm the exact change before `Budget_EnterpriseBudget_Update`, `Budget_CostOfProduction_Update`, `Budget_LineItem_Update`, or `Budget_LineItemDetail_Update`. For a new id, call `Util_NewGuid` first. On an update, pass the existing id from search.

## Workflow

The stack is four levels. Drill down. Do not skip a level to guess an id.

### 1. Enterprise budget

`Budget_EnterpriseBudget_Search`. Each row is the year header (`name`, `year`, `status`, rollup acreage and dollars). The row `id` is `budget_id`.

**Done when** you have `budget_id`, or the search returned no rows.

### 2. Cost of production

`Budget_CostOfProduction_Search` with that `budget_id`. Each row is one practice: state, county, commodity, type, practice, reported acreage, yield, projected price. The row `id` is `budget_county_practice_id`.

**Done when** you have the practice rows the user asked about.

### 3. Line items, if they asked for costs

`Budget_LineItem_Search` with `budget_county_practice_id` for categories such as seed, chemicals, or rent.

`Budget_LineItemDetail_Search` with the same id for the inputs inside a line item.

**Done when** the requested level has been read.

### 4. Answer

Show year, status, then the practice, then line items. Put the tool name beside each block.

Field crops that share the same state, county, commodity, type, and practice belong to `tilley-farm`.

**Done when** every number in the answer came from a tool result in this session, and any write was confirmed first.

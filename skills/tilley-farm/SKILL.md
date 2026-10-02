---
name: tilley-farm
description: Look up a producer's farms, fields, tracts, crops, base acres, and yield history in Tilley. Use when the user asks about farms, fields, acres planted, crops on a farm, APH, or base acres.
---

# Tilley farms

If `Farms_ProducerFarm_Search` is not in your tool list, stop and add the Tilley MCP at https://mcp.farmtilley.com/ (see the repo README). Then start at step 1.

## Rules

- A farm fact enters the answer only after a Tilley tool returned it in this session. Name that tool.
- An empty result means nothing matched those filters.
- Call `GetLookupOptions` before filtering on `commodity_name`, `type_name`, `practice_name`, state, or county. Pass a value from that list exactly. County lookup requires `state_name` from `lookup=state_name`. Farm tools name the same strings `location_state_name` and `location_county_name`.
- Signed-in producer: omit `producer_token` and `agent_token`. Agent for a client: pass that client's producer GUID. Never send `""`.
- `*_Search` is the short list. `tilley_*_Select` is the full row. Search first. Select when the user needs columns the search left out.
- Farm create and update tools are not on this server. Say so if the user asks to edit a farm.

## Workflow

### 1. Farms

Call `Farms_ProducerFarm_Search`. Keep `producer_farm_id` from each row. Fields and base acres hang off that id.

**Done when** you have the farm ids, or the search returned no rows.

### 2. The layer they asked for

| They asked about | Call |
| --- | --- |
| The farm list, or FSA identifiers | `tilley_ProducerFarm_Select` |
| Fields or tracts | `Farms_ProducerFarmField_Search` with `producer_farm_id`. The row `id` is `producer_farm_field_id` |
| Crops on a field | `Farms_ProducerFarmFieldCrop_Search` |
| Yield history or APH | `Farms_ProducerFarmFieldCropHistory_Search` |
| Base acres or PLC yield | `Farms_ProducerFarmBaseAcres_Search` |

Full rows use the matching `tilley_*_Select`: `tilley_ProducerFarmField_Select`, `tilley_ProducerFarmFieldCrop_Select`, `tilley_ProducerFarmFieldCropHistory_Select`, `tilley_ProducerFarmBaseAcres_Select`.

**Done when** each asked layer has been searched, or you stopped because the parent id was missing.

### 3. Answer

Lead with the farm and the crop year. List acres, crops, and yields from the rows. Put the tool name beside each block.

ARC/PLC payments belong to `tilley-arc-plc`. Grain sales belong to `tilley-marketing`. Enterprise budgets belong to `tilley-budget`.

**Done when** every number in the answer came from a tool result in this session.

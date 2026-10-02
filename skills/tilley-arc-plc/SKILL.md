---
name: tilley-arc-plc
description: Look up ARC/PLC program elections, base acres, county yield overrides, and ARC-CO or PLC payment estimates in Tilley. Use when the user asks about ARC, PLC, ARC-CO, program election, or farm-bill program payments.
---

# Tilley ARC/PLC

If `ArcPlc_ProgramElection_Search` is not in your tool list, stop and add the Tilley MCP at https://mcp.farmtilley.com/ (see the repo README). Then start at step 1.

## Rules

- An election, base-acre, yield, or payment figure enters the answer only after a Tilley tool returned it in this session. Name that tool.
- An empty result means nothing matched those filters.
- `ArcPlc_Calculate_Payments` is a scenario model. It does not read the producer's saved election. Say that when you use it.
- Call `GetLookupOptions` before commodity, type, practice, state, or county filters. Pass a value from that list exactly. The calculator uses `location_state_name` and `location_county_name` for those lookup strings.
- Signed-in producer: omit `producer_token` and `agent_token`. Agent for a client: pass that client's producer GUID. Never send `""`.
- Election and yield-override create/update tools are not on this server. Say so if the user asks to change an election.

## Workflow

### 1. What is on file

| They asked about | Call |
| --- | --- |
| Which program they elected | `ArcPlc_ProgramElection_Search`, then `tilley_ProgramElection_Select` for the full row |
| Base acres or PLC yield on the farm | `Farms_ProducerFarmBaseAcres_Search` |
| An ARC-CO county yield override | `ArcPlc_ArcCoCountyYieldOverride_Search` |

**Done when** the on-file tools have returned, including an empty election list.

### 2. Payment estimate, only if they asked

Call `ArcPlc_Calculate_Payments` with `crop_year`, `location_state_name`, `location_county_name`, `commodity_name`, `program_election`, `base_acreage`, `share_percent`, and `plc_yield`. Optional: `type_name`, `program_category`, `arc_co_practice`, `historical_irrigation_percentage`.

Take base acres, PLC yield, and share from step 1 when those rows exist. Ask for any required input the tools did not return. Do not invent them.

Label the result as a model estimate, and state the `program_election` you passed.

**Done when** the calculator has returned, or you stopped because a required input is missing.

### 3. Answer

Show the election on file first, then any model estimate under its own heading. Put the tool name beside each block.

**Done when** every number in the answer came from a tool result in this session, and a model payment is not described as the saved election.

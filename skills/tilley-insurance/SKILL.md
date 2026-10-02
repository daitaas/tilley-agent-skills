---
name: tilley-insurance
description: Look up crop insurance in Tilley: the producer's plans, claims, scenarios, and stress tests, plus RMA dates, offers, coverage levels, and summary of business. Use when the user asks about insurance, coverage, premiums, claims, or RMA offers.
---

# Tilley insurance

If `Insurance_InsurancePlan_Search` is not in your tool list, stop and add the Tilley MCP at https://mcp.farmtilley.com/ (see the repo README). Then start at step 1.

## Rules

- A coverage, premium, claim, or loss figure enters the answer only after a Tilley tool returned it in this session. Name that tool.
- An empty result means nothing matched those filters.
- Producer records and public RMA reference data are different. Say which one you used.
- Call `GetLookupOptions` before filtering on commodity, type, practice, state, or county. Pass a value from that list exactly.
- Signed-in producer: omit `producer_token` and `agent_token`. Agent for a client: pass that client's producer GUID. Never send `""`.
- Insurance create and update tools are not on this server. Say so if the user asks to change a plan or scenario.

## Workflow

### 1. Producer records, public RMA, or both

| They asked about | Call |
| --- | --- |
| Their plans | `Insurance_InsurancePlan_Search` |
| Claims on a plan | `Insurance_InsurancePlanClaim_Search` |
| Scenarios they saved | `Insurance_InsuranceScenario_Select` |
| Stress-test results | `Insurance_StressTestResult_Search` |
| Offers stored for them | `Insurance_InsuranceOffer_Select` |
| Sales closing, end of insurance, or other dates | `Insurance_Dates` |
| RMA actuarial offers | `RMA_ADM_Insurance_Offers` |
| Coverage levels for an offer | `RMA_ADM_Insurance_Coverage_Levels` |
| County or state loss experience for one year | `RMA_Summary_of_Business_by_Year` |
| Loss experience across years | `RMA_Summary_of_Business` |
| Cause of loss for one year | `RMA_Summary_of_Business_Cause_of_Loss_by_Year` |
| Cause of loss across years | `RMA_Summary_of_Business_Cause_of_Loss` |

**Done when** each part of the question has a tool result, labeled producer or RMA.

### 2. Answer

Separate "your Tilley records" from "RMA reference". Put the tool name beside each block.

The farm the policy sits on belongs to `tilley-farm`.

**Done when** every number in the answer came from a tool result in this session.

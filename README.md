# Tilley Agent Skills

Agent skills for [Tilley](https://www.farmtilley.com/) farms, marketing, crop insurance, budgets, and ARC/PLC.

Each skill reads the signed-in account through the Tilley MCP and reports figures from that session. If a tool returns no rows, the skill says so.

Works with **Cursor**, **Claude Code**, **Codex**, and any agent that reads `SKILL.md` files.

## Add the Tilley MCP

The producer server is remote HTTP. There is no package to install. When the client asks you to sign in, use your Tilley account. Leave client id and client secret empty: the server publishes its own OAuth metadata. Do not paste access tokens, passwords, or client secrets into a config file or into this repo.

**https://mcp.farmtilley.com/**

MCP calls are `POST /` and require that sign-in. A browser open of the same URL is a read-only tool list.

## Hey Tilley

Hey Tilley is the Tilley chat client. Use it when you do not have your own agent, such as Cursor, Copilot, Claude, or ChatGPT.

**https://chat.totilley.com/**

Sign in with your Tilley account. Hey Tilley is already connected to the Tilley MCP, so there is no server to add. The skill install steps further down are for an agent you run yourself.

## Connect your agent

### Cursor

Settings, MCP, or a config file. Project: `.cursor/mcp.json`. Every project: `~/.cursor/mcp.json`.

```json
{
  "mcpServers": {
    "tilley": {
      "url": "https://mcp.farmtilley.com/"
    }
  }
}
```

Reload MCP servers, then finish the Tilley sign-in.

### Claude Code

```bash
claude mcp add --transport http tilley https://mcp.farmtilley.com/
```

### Codex

In `~/.codex/config.toml`:

```toml
[mcp_servers.tilley]
url = "https://mcp.farmtilley.com/"
```

### VS Code

```json
{
  "servers": {
    "tilley": {
      "type": "http",
      "url": "https://mcp.farmtilley.com/"
    }
  }
}
```

### Tilley employees: admin MCP

Employees only. Add this on your own agent. Do not add it for a customer.

**https://admin-mcp.farmtilley.com/**

Same pattern as the producer server: `POST /` requires an employee admin sign-in, and opening the URL in a browser shows the tool list. It is a different host. Producer tools stay on `https://mcp.farmtilley.com/`.

```json
{
  "mcpServers": {
    "tilley-admin": {
      "url": "https://admin-mcp.farmtilley.com/"
    }
  }
}
```

After an employee sign-in, that host adds three modules (19 tools):

| Module | What it is for |
| --- | --- |
| `adhoc-sql` | Read-only SQL on Tilley catalogs. Start with `AdHocQuery_ChooseCatalog`, then `AdHocQuery_ExploreSchema`, then a query tool. |
| `aip-pass` | AIP PASS vendor folders and raw file links: `AipPass_Vendor_Search`, `AipPass_Vendor_Summary`, `AipPass_Files_Search`, `AipPass_Files_FileGetDocumentLink`. |
| `stripe` | Support lookups for charges, payment intents, checkout sessions, receipts, and refunds. Start with `Stripe_ChooseTask`. |

The skills in this repo call the producer server only. They do not call the admin tools.

## Install the skills

### Cursor

```bash
npx skills@latest add daitaas/tilley-agent-skills -a cursor
```

A project install lands in `.agents/skills/`. Add `-g` to install into `~/.cursor/skills/` for every project.

### Codex and other agents

```bash
npx skills@latest add daitaas/tilley-agent-skills
```

The installer asks which skills and which agents. Pass `-a codex`, `-a claude-code`, or `-a '*'` to skip the prompt.

### Claude Code

Use the installer with `-a claude-code`, or the plugin (one or the other; both installs every skill twice):

```
/plugin marketplace add daitaas/tilley-agent-skills
/plugin install tilley-skills@daitaas
```

### Manual

Copy a folder from `skills/` into `.cursor/skills/`, `.claude/skills/`, `.codex/skills/`, or `.agents/skills/`. Each folder stands alone.

## Skills

All skills are **model-invoked**: the agent reaches for them when a question fits, and you can name them directly.

- **[tilley-farm](./skills/tilley-farm/SKILL.md)**: Farms, fields, tracts, crops, base acres, and yield history (APH). Search first, then select full rows.
- **[tilley-marketing](./skills/tilley-marketing/SKILL.md)**: Crop profiles, grain sales, buyers, and position summary. Sale category stays as the row returned it (cash, HTA, basis, futures, OTC).
- **[tilley-insurance](./skills/tilley-insurance/SKILL.md)**: The producer's plans, claims, scenarios, and stress tests, plus public RMA dates, offers, coverage levels, and summary of business.
- **[tilley-arc-plc](./skills/tilley-arc-plc/SKILL.md)**: Program elections, base acres, county yield overrides, and the ARC-CO / PLC payment model.
- **[tilley-budget](./skills/tilley-budget/SKILL.md)**: Enterprise budget, cost of production, line items, and line-item details. Asks before any budget write.

## The grounding rule

Each skill carries the same contract:

1. **Allowed sources**: Tilley MCP tool output from this session, and files already in the user's repository.
2. **Figures**: acres, yields, prices, elections, and ids appear in the answer only after a tool returned them. Name the tool.
3. **Empty results**: "nothing matched those filters" is a valid answer.
4. **Lookups**: call `GetLookupOptions` before any filter on commodity, type, practice, state, or county, and pass a returned value exactly. County lookup needs the full state name from `lookup=state_name`.
5. **Scope**: for the signed-in producer, omit `producer_token` and `agent_token`. When acting for a client, pass that client's producer GUID. Never send an empty string.
6. **Missing server**: if the Tilley tools are absent, stop and add [the MCP server](#add-the-tilley-mcp). Do not answer a farm question from memory.

`search_web` on the producer server is for public pages. It is not a source for the user's acres, sales, or elections.

## Other producer tools

These live on `https://mcp.farmtilley.com/` and have no skill of their own yet.

| Area | Tools |
| --- | --- |
| Documents | `Document_Search`, `DocumentDictionary_Search`, `DocumentContent_Search`, `Document_DocumentGetContent`, `Document_FileGetLink`, `Document_Transform_BaseAcres` |
| ToTilley inbox | `Emails_Search`, `Emails_AttachmentFileGetLink` |
| Mail to yourself | `SendEmail_GetMyAddress`, `SendEmail_Send` |
| Account | `Member_Select`, `Member_Subscription_Search`, `Client_Search`, `Member_Payment_GetSessionUrl`, `Member_Invoices_Payment_GetSessionUrl`, `Member_Payment_GetPaymentLinkUrl`, `Member_Payment_GetReceiptUrl` |
| Quotes | `Futures` |
| Soil | `SCN_Soybean_Risk_Calculator`, `soil_composition_lookup` |
| Helpers | `GetLookupOptions`, `Util_NewGuid`, `code_interpreter` |

Farm, insurance, and marketing create/update tools are not advertised. Budget `*_Update` tools are, and `tilley-budget` asks before it calls one.

## Repository layout

```
skills/
  tilley-farm/
    SKILL.md
    agents/openai.yaml
  tilley-marketing/
  tilley-insurance/
  tilley-arc-plc/
  tilley-budget/
.claude-plugin/
  plugin.json          Claude Code plugin manifest
  marketplace.json     this repo as its own marketplace
AGENTS.md              conventions for agents editing this repo
```

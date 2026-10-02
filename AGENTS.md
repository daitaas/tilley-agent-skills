Skills live in `skills/<name>/SKILL.md`, one folder per skill, flat.

Every skill must appear in three places: the `## Skills` list in `README.md` (name linked to its `SKILL.md`, one-line description), the `skills` array in `.claude-plugin/plugin.json`, and its own `agents/openai.yaml` beside `SKILL.md` (`interface.display_name`, `interface.short_description`).

Every skill in this repo is model-invoked: omit `disable-model-invocation` from the frontmatter and omit the `policy` block from `agents/openai.yaml`. The `description` stays model-facing and keeps its trigger phrasing ("Use when the user asks...") so auto-invocation fires.

Every skill carries the grounding rule from `README.md`. Farm, sale, insurance, budget, and program figures come from Tilley MCP output in the current session. Change that rule in every skill and in `README.md` together.

Tool names in skills must match the live Tilley MCP (`Farms_ProducerFarm_Search`, `GetLookupOptions`, `ArcPlc_Calculate_Payments`, and the rest listed in the README). Do not invent tool names. Farm, insurance, and marketing create/update tools are not advertised on the producer server. Budget `*_Update` tools are. A skill that writes must ask the user to confirm first.

Each `SKILL.md` stays under 500 lines and self-contained: no `../other-skill/` links. If two skills need the same rule, duplicate the few lines.

No em dashes in prose anywhere in this repo. Rewrite with a comma, colon, period, parentheses, or a conjunction.

Do not commit secrets, tokens, passwords, or client secrets.

# V+ Versicherung — Voice agent structure (CILM project)

**One dispatcher, two worker agents.** German-first (Austrian market), Lena voice (de-AT), branded **V+ Versicherung**. Pricing + case creation run through n8n → Pega Infinity (DX API).

```
Caller
  │
  ▼
[ Empfang / Dispatcher ]  agent_3201kw8trpk1fhzr6xqkqq5bf9h8
  │  "Angebot oder Schaden melden?"
  ├──► [ Angebot / Offer ]  agent_3701kvdy0a45eywvg4yn9kw8pqv6   (existing lead agent — creates offers)
  └──► [ Schaden / Claim ]  agent_2301kw8tntcnfc3tfmdac3qwde3d   (creates claim cases)
```

## Agents
| Role | Agent | File | Creates |
|---|---|---|---|
| Dispatcher (Empfang) | `agent_3201kw8trpk1fhzr6xqkqq5bf9h8` | `vplus-router-agent.json` | — routes only |
| Offer (Angebot) — EXISTING | `agent_3701kvdy0a45eywvg4yn9kw8pqv6` | `vplus-lead-agent.json` | indicative quote + lead case |
| Claim (Schadenmeldung) | `agent_2301kw8tntcnfc3tfmdac3qwde3d` | `vplus-claim-agent.json` | claim case (returns Pega case ID) |

All three share: voice **Lena** `BtJhEZecBTSpKQ8EHRCJ`, model `eleven_multilingual_v2`, brand **V+ Versicherung**, German with umlauts.

## Backup / restore
`backup/vplus-lead-agent.restore.json` — pre-extension restore point of the existing offer agent. Restore via `PATCH /v1/convai/agents/agent_3701kvdy0a45eywvg4yn9kw8pqv6`.

## Backend
- Pricing: n8n `unica-quote-v3` webhook.
- Lead case: n8n `pega-lead-intake` → Pega DX API.
- Claim case: n8n `vplus-claim-create` (workflow `m6LMKzppEW1YxLUu`) → Pega DX API → `MyOrg-CILM-Work-LeadIntake` (pyLabel + pyDescription).
- Pega: `https://2s8uoi5i.pegace.net/prweb/` (DX API v1, operator `admin@cilm.com`). Wakes from hibernation via UI on 503.

## Known constraint
n8n Cloud monthly execution quota: when exceeded, all webhooks (pricing, lead, claim) stop until the plan/quota is restored. Agent build/edit (ElevenLabs API) is unaffected.

## Demo
Landing page + widget: https://andreytry.github.io/unica-demo/  (entry point = dispatcher)

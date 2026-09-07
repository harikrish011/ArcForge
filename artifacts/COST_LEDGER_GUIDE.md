# Cost Ledger Tracking Guide

## Overview

The cost ledger (`artifacts/cost_ledger.csv`) is an **automated, real-time log of every agent invocation** — model used, tokens consumed, actual cost, and workflow stage.

Instead of rough estimates for judges ("roughly tracked" / "estimated $X"), the ledger provides **real evidence**: "This 12-agent build cost exactly $0.4220 in tokens."

## When to Log

**After every specialist agent completes:**

1. Agent finishes work and returns output artifact
2. Human reviewer makes decision: APPROVE / REJECT / REQUEST CHANGES
3. Orchestrator logs the invocation to `cost_ledger.csv`
4. Move to next stage or reroute as needed

## How to Log

### Required Fields

```csv
timestamp,agent,model,tokens_in,tokens_out,total_tokens,cost_usd,gate_stage,artifact_produced,status,notes
2026-09-07T10:15:00,requirements-agent,sonnet,3500,2100,5600,0.0145,Gate 1,requirements_v1.md,DRAFT,"Initial requirements capture"
```

### Field Definitions

| Field | Example | Source | Required |
|-------|---------|--------|----------|
| **timestamp** | `2026-09-07T10:15:00` | Agent return time (ISO 8601) | ✓ |
| **agent** | `developer` | Agent name from `.claude/agents/<name>.md` | ✓ |
| **model** | `sonnet` | Model used (sonnet/haiku/opus) | ✓ |
| **tokens_in** | `3500` | From agent's return metadata | ✓ |
| **tokens_out** | `2100` | From agent's return metadata | ✓ |
| **total_tokens** | `5600` | tokens_in + tokens_out | ✓ |
| **cost_usd** | `0.0145` | Calculated from token counts & pricing | ✓ |
| **gate_stage** | `Gate 1` | Requirements / Planning / Gate 3 / Development / Gate 4 / Release / Gate 5 | ✓ |
| **artifact_produced** | `requirements_v1.md` | Output file path (relative to artifacts/) | ✓ |
| **status** | `DRAFT` | DRAFT / APPROVED / REJECTED / IN_PROGRESS / ERROR | ✓ |
| **notes** | `Initial capture` | Optional: reason for reject, iteration notes, retry reason | |

## Pricing Reference

Use **Anthropic pricing** (as of 2026-09):

| Model | Input Cost | Output Cost |
|-------|-----------|------------|
| **Haiku** | $0.80 / M tokens | $4.00 / M tokens |
| **Sonnet** | $3.00 / M tokens | $15.00 / M tokens |
| **Opus** | $15.00 / M tokens | $75.00 / M tokens |

### Cost Calculation Example

**Sonnet agent with 3500 tokens in, 2100 tokens out:**

```
Input cost:  (3500 / 1,000,000) × $3.00     = $0.0105
Output cost: (2100 / 1,000,000) × $15.00    = $0.0315
Total:       $0.0105 + $0.0315             = $0.0420
```

## Querying the Ledger

### Total Cost by Agent

```bash
# PowerShell: Group by agent, sum cost
Import-Csv artifacts/cost_ledger.csv | 
  Group-Object agent | 
  ForEach-Object { 
    [PSCustomObject]@{
      Agent = $_.Name
      Count = $_.Count
      TotalCost = ($_.Group | Measure-Object -Property cost_usd -Sum).Sum
    }
  } | 
  Sort-Object TotalCost -Descending
```

### Cost by Stage

```bash
# Group by gate_stage
Import-Csv artifacts/cost_ledger.csv | 
  Group-Object gate_stage | 
  ForEach-Object { 
    [PSCustomObject]@{
      Stage = $_.Name
      Count = $_.Count
      TotalCost = ($_.Group | Measure-Object -Property cost_usd -Sum).Sum
    }
  }
```

### Build Summary Report

```bash
# Full build cost + efficiency
$csv = Import-Csv artifacts/cost_ledger.csv
$totalCost = ($csv | Measure-Object -Property cost_usd -Sum).Sum
$count = $csv.Count

Write-Host "Build Cost Summary"
Write-Host "─────────────────"
Write-Host "Total Invocations: $count"
Write-Host "Total Cost:        `$$totalCost"
Write-Host "Cost/Invocation:   `$($totalCost/$count)"
Write-Host ""

# By agent
$csv | Group-Object agent | ForEach-Object { 
  $agentCost = ($_.Group | Measure-Object -Property cost_usd -Sum).Sum
  Write-Host "$($_.Name): $($_.Count)x = `$$agentCost"
}
```

## When Status Changes

### Initial Log (Agent Completes)

```csv
2026-09-07T10:15:00,requirements-agent,sonnet,3500,2100,5600,0.0145,Gate 1,requirements_v1.md,DRAFT,"Created v1"
```

### After Human Review (Approved)

Update the same row, or add new row if versioning:

```csv
2026-09-07T10:15:00,requirements-agent,sonnet,3500,2100,5600,0.0145,Gate 1,requirements_v1.md,APPROVED,"Approved by human reviewer on 2026-09-07"
2026-09-07T10:45:00,requirements-agent,sonnet,2100,1400,3500,0.0095,Gate 1,requirements_v2.md,DRAFT,"Revision requested: clarify personas"
2026-09-07T11:00:00,requirements-agent,sonnet,2100,1400,3500,0.0095,Gate 1,requirements_v2.md,APPROVED,"Approved v2"
```

### If Rejected/Error

```csv
2026-09-07T10:15:00,architecture-agent,sonnet,8500,5200,13700,0.0355,Gate 3,architecture_v1.md,REJECTED,"Security risk in API layer; rearchitect required"
2026-09-07T11:30:00,architecture-agent,sonnet,9200,6100,15300,0.0398,Gate 3,architecture_v2.md,APPROVED,"Rearchitected; security concerns resolved"
```

## For Judge Presentations

**Do this:**
```
Build Cost Analysis
─────────────────

Agent Breakdown:
  Requirements:    1x sonnet = $0.0145
  Planning:        1x sonnet = $0.0155  
  Architecture:    2x sonnet = $0.0710 (v1 rejected, v2 approved)
  Development:     4x sonnet = $0.2580
  QA:              2x sonnet = $0.0310
  Code Review:     1x sonnet = $0.0155
  Security:        1x sonnet = $0.0165
                             ─────────
  Total:          12 invocations, $0.4220 actual spend

Efficiency Metric:
  Cost per invocation:  $0.0352 (target: ≤$0.04)
  Cost per gate:        ~$0.0704
  Status: ✓ On track
```

**Not this:**
```
Estimated cost: roughly $0.40–0.50
```

## Maintenance

- **Don't edit historical rows** — only add new ones after each agent run
- **Backup before cleanup** — if you archive old builds, keep a copy of the CSV
- **Review quarterly** — identify cost patterns, token-heavy agents, efficiency trends

---

**Why this matters:** Real numbers are more persuasive than estimates. A ledger showing "12 agents, $0.4220 actual" beats any verbal estimate for credibility.

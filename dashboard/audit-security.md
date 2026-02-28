# Audit & Security

The Audit & Security section provides a unified view of your dApp's health, data integrity, and security posture.

## Overview Tab

The overview displays a composite **dApp Health Score** (0–100) calculated from 8 components:

| Component | Weight | What it measures |
|-----------|--------|-----------------|
| Conversion | 15% | Funnel efficiency from visit to transaction |
| Integrity | 20% | K-PPE proof anchoring success rate |
| Reliability | 15% | Uptime, error rates, response times |
| Growth | 10% | User acquisition trend, DAU/WAU/MAU |
| UX | 10% | Bounce rate, session duration, rage clicks |
| Engagement | 10% | Pages per session, return rate |
| Retention | 10% | Wallet cohort retention over time |
| Volume | 10% | Event throughput vs plan limits |

The score is displayed as a circular ring with color coding: green (80+), amber (50–79), red (below 50).

## Audit Tabs

### Conversion
Funnel analysis with conversion rates at each step. Highlights the biggest drop-off points and provides actionable recommendations (e.g., "40% of users disconnect their wallet before completing a transaction — consider adding a progress indicator").

### Integrity
K-PPE proof status: number of batches anchored, anchoring frequency, last anchor timestamp, and any failed anchoring attempts. Shows the verification status for recent batches.

### Reliability
System health metrics: relayer uptime, average response time, error rate by endpoint, and failed webhook deliveries. Alerts on degraded performance.

### Growth
User acquisition metrics: new wallets per day, traffic sources, referrer analysis, and growth rate trends.

### UX
User experience indicators: average load time, bounce rate by page, scroll depth distribution, rage click detection, and error impact on sessions.

### K-PPE
Detailed K-PPE engine status: proof batches in queue, anchoring schedule, gas costs, and Merkle tree sizes. Technical view for developers monitoring the proof pipeline.

### Sentinel
Anomaly detection: statistical analysis of all metrics to detect unusual patterns. Flags traffic spikes, conversion drops, suspicious wallet activity, and potential bot behavior. Each anomaly includes severity, affected metric, detection timestamp, and recommended action.

## Findings

Each audit tab may generate **findings** — actionable insights ranked by severity:

* 🔴 **Critical** — Immediate action required (e.g., proof anchoring has failed for 24+ hours)
* 🟠 **Warning** — Should be addressed soon (e.g., error rate above 5%)
* 🟡 **Info** — Optimization opportunity (e.g., adding a CTA could improve conversion by ~15%)

Findings include a description, impact assessment, and a recommended fix.

## Action Board

The Action Board aggregates all findings across tabs into a prioritized to-do list. Mark items as "In Progress" or "Resolved" to track your dApp's improvement over time.

---
layout: default
title: "Intelligence Cycle"
parent: "How do I prepare for the profession of cyber security analyst and CTI?"
nav_order: 2
---

# Intelligence Cycle (CTI Perspective – Notes)

Intelligence is a process, not a single step. In CTI, the goal is to understand attacker behavior, intentions, and capabilities, then produce useful outputs (reports, alerts, etc.).

---

## Basic Flow (Simplified)

1. Define the problem  
2. Identify requirements  
3. Collect data (logs, OSINT, intel feeds)  
4. Verify reliability  
5. If insufficient → continue collecting  
6. Analyze and correlate data  
7. Share internally  
8. Deliver to decision-makers (SOC / security team)  


---

## Problem Definition

Everything starts with a question.

Examples in CTI:
- Is this malware campaign active?  
- Is this actor targeting us?  
- Is this normal or an anomaly?  

Tasks may come from SOC or management, but sometimes analysts identify anomalies themselves.

---

## Intelligence Warning (Early Detection)

Main goal: prevent surprise.

Analysts should detect abnormal signals.

Key questions:
- Is this relevant?  
- Is this certain?  
- Can it be tracked?  

Types:
- **Technical warning:** logs, indicators, evidence  
- **Analytical warning:** prediction, not always certain  


---

## Requirements & Hypothesis

Before collecting data:

- What do I need to know?  
- What questions should be answered?  

Then form hypotheses:

- Hypothesis 1 → targeted activity  
- Hypothesis 2 → random/noise  

Data collection should support these hypotheses.

---

## Data Collection (CTI View)

Sources:
- Logs (SIEM, EDR)  
- OSINT  
- Threat intelligence feeds  
- Malware analysis  
- Network traffic  

Types:
- **Primary:** raw data  
- **Secondary:** reports, articles  

Also:
- Open sources  
- Controlled/closed sources  



---

## Key Idea

Not all data is useful.

Analysts must:
- Filter  
- Prioritize  
- Correlate  

Otherwise → information overload.

---

## Final Thought

CTI analysis is not only technical.

It requires:
- Critical thinking  
- Questioning  
- Pattern recognition  

Goal:
- Reduce uncertainty  
- Support decision-making in SOC environments  

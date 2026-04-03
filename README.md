# **SQL Injection Playbook**

### Methodology, Decision-Making, and Testing Patterns

This repository is a structured knowledge base focused exclusively on **SQL injection testing**, built from hands-on work with PortSwigger labs and aligned with real-world web application behavior.

It is intentionally **methodology-driven**.

Rather than cataloging payloads or step-by-step exploits, this playbook documents how SQL injection is approached in practice:

* how input points are identified and prioritized
* how query behavior is inferred from limited feedback
* how attack paths are selected under constraints
* how testing adapts when visibility is restricted

---

## **Purpose**

This repository exists to capture **decision-making patterns** during SQLi testing.

It answers questions like:

* *What do I test first when I see a parameter?*
* *How do I distinguish between filtered input vs. non-executable input?*
* *When do I switch from in-band to blind techniques?*
* *How do I confirm injection when there is no visible output?*

The goal is not memorization.

The goal is **repeatable reasoning under uncertainty**.

---

## **How to Use This Repository**

This is not meant to be read linearly.

Each section is a **mental model** you can replay during:

* live testing sessions
* lab environments
* technical interviews

Focus on:

* signals and indicators
* decision branches
* fallback strategies

---

## **Repository Structure**

```
sqli-playbook/
├─ 00-SQLi-Global_Reference.md
│
├─ In-Band/
│ ├── Union-Based.md
│ └── Error-Based.md
│
├─ Blind/
│ ├── Boolean-Based.md
│ ├── Error-Based.md
│ └── Time-Based.md
│
├─ Out-of-Band/
│ └── Concepts.md   # conceptual only (no full OAST execution)
│
└─ methodology.md
```

---

## **What This Repository Is**

* A **pattern library** for SQL injection testing
* A **decision framework** for navigating unknown behavior
* A **compressed reference** for retaining hands-on experience
* A demonstration of practical testing workflow using tools like Burp Suite

---

## **What This Repository Is Not**

❌ A beginner-friendly SQLi tutorial  
❌ A walkthrough or lab solution archive  
❌ A payload list or cheat sheet  
❌ A replacement for hands-on exploitation

Detailed attack chains and full exploitation flows are documented separately in lab writeups.

---

## **Relationship to Other Work**

* **Lab writeups** → full exploitation paths and results
* **Scripts/tools** → automation and experimentation
* **This repository** → the reasoning that connects them

---

## **Guiding Principle**

Payloads change.
Filters evolve.
Defenses improve.

**Behavioral patterns remain consistent.**

This playbook focuses on those patterns.

---
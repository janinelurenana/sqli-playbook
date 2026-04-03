# **Out-of-Band SQL Injection (OAST) — Concepts**

### Detection Logic Under Limited Visibility

Out-of-band (OOB) SQL injection is used when traditional techniques fail due to **lack of direct or inferable feedback**.

This includes scenarios where:

* responses are static or heavily normalized
* errors are suppressed
* timing differences are unreliable or too noisy
* application behavior does not visibly change

In these cases, confirmation relies on **external interaction** triggered by the database.

---

## **Core Idea**

Instead of observing the application’s response,
you observe **side effects outside the application boundary**.

The database is coerced into making:

* DNS requests
* HTTP requests
* or other outbound connections

to an attacker-controlled system.

If the interaction is received, injection is confirmed.

---

## **When to Consider OOB**

OOB is not a first-line technique.

It becomes relevant when:

* in-band techniques (UNION, error-based) fail
* blind techniques (boolean/time-based) produce ambiguous results
* application responses are fully static
* input appears to reach backend logic but cannot be verified

**Decision trigger:**

> “I suspect injection, but I have no reliable signal to confirm it.”

---

## **Detection Model**

The process is conceptually simple:

1. Inject payload that forces the database to initiate an outbound request
2. Include a **unique, trackable identifier** in the request
3. Monitor external system for interaction
4. Correlate interaction with injection attempt

**Signal = external callback**

---

## **Database Capabilities Matter**

OOB feasibility depends entirely on what the database can do.

Key questions:

* Can it resolve external domains (DNS)?
* Can it initiate HTTP requests?
* Are there built-in functions or procedures enabling outbound interaction?
* Is outbound traffic restricted by network policy?

Not all environments support OOB.

Even when SQLi exists, OOB may fail due to:

* firewall rules
* egress filtering
* disabled database features

---

## **Channels of Interaction**

### **DNS-based**

* Most common and reliable
* Often allowed even when HTTP is blocked
* Used for detection, sometimes limited exfiltration

### **HTTP-based**

* Higher bandwidth than DNS
* Enables more flexible data exfiltration
* More likely to be blocked in hardened environments

---

## **Use Cases**

### 1. **Detection Only**

Confirming SQL injection exists when no other signal is available.

### 2. **Data Exfiltration**

Extracting small amounts of data via:

* subdomains (DNS)
* request parameters (HTTP)

### 3. **Privilege/Capability Testing**

Determining:

* database permissions
* network access level
* available outbound channels

---

## **Constraints and Trade-offs**

| Constraint               | Impact                  |
| ------------------------ | ----------------------- |
| No outbound connectivity | OOB impossible          |
| DNS only allowed         | Limited data throughput |
| High latency             | Slower verification     |
| Logging/monitoring       | Higher detection risk   |
| Limited DB functions     | Payload constraints     |

---

## **Common Failure Interpretation**

Failure does **not** mean:

* SQL injection is absent

Failure may mean:

* outbound traffic is blocked
* payload did not execute in a reachable context
* database lacks required functionality

**Important distinction:**

> OOB tests *environment capability*, not just vulnerability presence.

---

## **Relationship to Other SQLi Techniques**

| Technique            | Feedback Source   |
| -------------------- | ----------------- |
| In-band              | Direct response   |
| Blind (boolean/time) | Indirect response |
| OOB                  | External system   |

OOB is typically:

* slower to set up
* more environment-dependent
* but sometimes the **only viable confirmation method**

---

## **Tooling Context**

In practice, OOB testing is commonly performed using:

* Burp Suite Collaborator (for DNS/HTTP interaction tracking)

However, full OAST workflows require:

* controlled infrastructure
* reliable monitoring endpoints
* and (in many cases) professional tooling access

This repository focuses on:

* **decision-making and detection logic**, not full OAST execution

---

## **Mental Model**

When everything is silent:

* no errors
* no output
* no timing difference

Shift the question from:

> “What is the app telling me?”

to:

> “Can I make the database talk to something I control?”

---

## **Key Takeaway**

Out-of-band SQLi is not about payload complexity.

It is about recognizing when:

* traditional feedback loops are gone
* and confirmation must come from **outside the system**

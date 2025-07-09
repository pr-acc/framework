# **Framework for Privacy Robustness Accelerationism (PR/ACC)**

## **1. Introduction**

### 1.1. Purpose

This document specifies the Privacy Robustness Accelerationism (PR/ACC) framework. The framework's objective is to provide a systematic and standardized methodology for evaluating the privacy and robustness guarantees of applications, particularly those in the decentralized space whose privacy properties are user-visible. We also evaluate infrastructure protocols, but treat them as single applications themselves rather than as general-purpose platforms

### 1.2. Scope

The framework is designed to assess an application's resistance to information leakage by an external observer (the attacker). It evaluates applications based on two primary categories: **Privacy**, which measures the effectiveness of information obfuscation, and **Robustness**, which measures the protocol's resilience and architectural soundness.

### 1.3. Intended Audience

This specification is intended for protocol developers, security auditors, researchers, and informed users who wish to conduct a structured analysis of a privacy-centric application.

---

## **2. Core Concepts and Terminology**

For the purpose of this framework, the following terms are defined:

- **User:** An entity, typically a human, who interacts with the application to perform one or more actions. The user's goal is to perform these actions while preserving their privacy.
- **Attacker:** An entity, either an individual or an organization, that observes public data flows (e.g., on-chain transactions, network-level metadata) in an attempt to infer private information about a user's identity or actions. The attacker is assumed to be law abiding (can access public data, no hacking) but capable of sophisticated data analysis.
- **Action:** A discrete, user-initiated interaction with the application's protocol. An action can be a single transaction or a sequence of related transactions that constitute a stateful process (e.g., depositing, swapping, and then withdrawing funds).
- **Protocol:** The set of rules, cryptographic methods, smart contracts, and infrastructure that collectively enable the application's functionality.

---

## **3. Evaluation Category I: Privacy**

This category assesses the degree to which an application obscures sensitive information from an attacker. The evaluation is focused on the difficulty of breaking the obfuscation and linking information back to a user.

Note that here we only concern the information within the application, not about during the process of joining the application that might be varied across different protocol categories, for example

- **For DeFi:** It may be acceptable to reveal the set of all addresses that have "joined" the protocol's anonymity set, as this is often a prerequisite for participation.
- **For Web2-style Credentials:** The protocol should ideally hide who has even joined. This can be achieved with technologies like a salted server or Verifiable Oblivious Pseudorandom Functions (vOPRF).

### **3.1. Criterion: Identity Leakage (Who)**

**Description:** This criterion measures the amount of information an attacker can extract about a user's identity from observing public data flows. It assesses the linkability between actions and a user's persistent or pseudonymous identifier.

- **Level 0: Direct Identity Linkage**
  - **Definition:** The protocol makes no significant attempt to obfuscate the user’s originating address (e.g., their primary wallet). All actions are publicly and directly attributable to this single, persistent identifier.
  - **Characteristics:**
    - Actions (e.g., transactions) originate from or are sent to the user's primary, long-term wallet address.
    - An attacker can easily trace a complete history of a user's interactions with the protocol, all linked to one address.
    - _Example:_ An on-chain interaction where `[address A]` takes `action 1`, followed by `action 2.1`, `action 2.2`, and `action 2.3`.
- **Level 1: Pseudonymous Linkage**
  - **Definition:** The protocol attempts to break the link to the user's primary identity by using one (or more) temporary addresses (pseudonyms) or provide plausible deniability (e.g. with ring signature). However, if interactions are stateful (multiple related steps), those steps can be correlated to the same address, making it possible for attacker to deduce/form user’s personas from this usage pattern.
  - **Characteristics:**
    - Each new, independent stateless interaction may use a new pseudonym, but multi-step stateful processes are linked.
    - Each action come from a set of possible actors
    - _Example:_ `[pseudonym_address_1]` takes `action 1`. Later, the same user performs a multi-step action where `[pseudonym_address_2]` takes `action 2.1`, `action 2.2`, and `action 2.3`. The link between `pseudonym_1` and `pseudonym_2` is not connected, but the actions under `pseudonym_2` are linked.
    - Example: `[one of addresses a,b,c,d]` takes `action 1`
  - **Illustrative Protocols:** Monero, Tornado Cash Classic (v1).
- **Level 2: Cryptographic Unlinkability**
  - **Definition:** The protocol employs strong cryptographic techniques to ensure that even actions performed by the same user, including sequential steps in a stateful process, are unlinkable to each other from an external observer's perspective.
  - **Characteristics:**
    - An attacker cannot correlate different actions to the same originating user.
    - _Example:_ A user performs a multi-step action where `[pseudonym_address_1]` takes `action 1`, `[pseudonym_address_2]` takes `action 2.1`, `[pseudonym_address_3]` takes `action 2.2`, and `[pseudonym_address_4]` takes `action 2.3`. There is no public data to link these pseudonyms.
  - **Illustrative Protocols:** Tornado Cash Nova, Railgun (private transfers), Aztec (internal transactions).

### **3.2. Criterion: Action Leakage (What)**

**Description:** This criterion measures how much information an attacker can extract about the _nature_ of the user's action, including its type of action and specific parameters. This is assessed independently of the user's identity. The visibility of the _result_ after the action does not affect this evaluation — for example, an action might be fully transparent while its outcome remains opaque due to the protocol’s encrypted internal state.

- **Level 0: Transparent Action and Parameters**
  - **Definition:** An attacker can see the precise type of action performed and all of its detailed parameters (e.g., asset types, amounts, destinations), even if the "who" is obfuscated. The public visibility of the action's _result_ (that maybe due to protocol encrypted state or other reasons) is irrelevant to this criterion.
  - **Characteristics:**
    - Public data reveals the specific function called and its arguments.
    - _Example:_ `[someone]` swaps exactly `5000 USDT` for `2 ETH`. Or `[someone]` sends exactly `1 ETH` to `[another pseudonym]`.
  - **Illustrative Protocols:** Tornado Cash Classic (v1).
- **Level 1: Obfuscated Action Parameters**
  - **Definition:** An attacker can discern the general _type_ of action being performed, but critical parameters are either hidden, aggregated, or sufficiently ambiguous to prevent precise inference.
  - **Characteristics:**
    - The specific values within an action are obscured.
    - Actions may be batched or aggregated, revealing a total but not individual contributions.
    - _Example:_ The public data shows that `[a bunch of addresses]` collectively lent `999 USDT`, but individual amounts are unknown. Or, `[someone]` vote for `[some options]` in a governance proposal.
- **Level 2: Cryptographically Opaque Action**
  - **Definition:** An attacker can determine neither the specific type of action performed nor any of its associated parameters from observing public data. The interaction appears as a generic, indistinguishable event.
  - **Characteristics:**
    - All transaction details beyond the proof of validity are encrypted or otherwise cryptographically hidden.
    - _Example:_ Public data only shows that `[someone]` did `[something]` by interacting with the protocol's smart contract.
  - **Illustrative Protocols:** Aztec (fully private transactions on its L2).

### **3.3. Criterion: De-anonymization Trust Assumption**

**Description:** This criterion evaluates the protocol’s reliance on trusted or privileged entities. It measures how difficult it is for such independent entities to be compromised or forced to collude to reverse the privacy guarantees (of "Who" or "What") back to Level 0.

This is framed by the **"Government Test"**: Can governments (individually or jointly) legally and practically force the protocol to reveal what was made private?

- **Level 0: Centralized Trust / Fails "Government Test"**
  - **Definition:** The ability to de-anonymize users or actions rests with a single entity or a small, non-independent group of entities. These entities can be legally compelled or technically compromised to reveal user data.
  - **Characteristics:**
    - A single server, operator, or administrator holds decryption keys or logs linking users to actions.
    - Collusion of a very small number of parties is sufficient to break privacy.
- **Level 1: Distributed Trust / Passes "Government Test"**
  - **Definition:** De-anonymization requires the active collusion or compromise of a significant number of independent, geographically and jurisdictionally diverse entities. The operational and legal overhead to compel all necessary parties is prohibitively high.
  - **Characteristics:**
    - The protocol relies on a large set of validators or sequencers distributed across many legal jurisdictions.
    - Security practices like frequent, automated rotation of cryptographic keys among a distributed set of parties are employed.
    - Compromising a minority of participants does not compromise the privacy of the system.
- **Level 2: Trustless / Self-Sovereign**
  - **Definition:** The protocol is architected such that no entity, or even the complete collusion of all protocol operators and infrastructure providers, can de-anonymize users. The information required for de-anonymization is secured exclusively by the user and is never shared.
  - **Characteristics:**
    - All sensitive user data is either encrypted client-side with keys exclusively controlled by the user, or never leaves the user’s local environment at all.

---

## **4. Evaluation Category II: Robustness**

This category assesses the protocol's operational resilience. It focuses on how difficult it is to disrupt the protocol and the protocol’s ability to recover from an attack or failure.

### **4.1. Criterion: Protocol Survivability**

**Description:** This criterion evaluates the protocol's dependency on specific infrastructure or centralized parties to function correctly and recover from failure. It asks: How difficult is it to take the protocol down, and what is required for it to continue operating?

- **Level 0: Fragile**
  - **Definition:** The protocol's core functionality relies on centralized components that represent single points of failure. If these components go offline, the protocol ceases to function or cannot be recovered.
  - **Characteristics:**
    - Core components (e.g., provers, data relays, front-ends) are operated by a single entity ("Founder & Friends").
    - The protocol is closed-source, preventing the community from redeploying or maintaining it.
    - All applications are considered Level 0 by default unless they can demonstrate otherwise.
- **Level 1: Moderately Robust**
  - **Definition:** The protocol is architected on distributed infrastructure but may have certain components that are less resilient. However, these weak points are designed for easy recovery or replacement by the community.
  - **Characteristics:**
    - The protocol's codebase **must be open-source**.
    - While some default infrastructure (e.g., a primary RPC endpoint or front-end) might be centralized, any sufficiently skilled user can bypass the failure by running their own node or interacting directly with the on-chain contracts.
- **Level 2: Highly Robust**
  - **Definition:** The protocol is exceptionally difficult to disable. Its operational dependency is minimal and relies only on its highly robust base-layer infrastructure.
  - **Characteristics:**
    - The protocol's core logic is fully autonomous and encoded in on-chain smart contracts.
    - Any user can continue to use the protocol with basic, widely available infrastructure.
    - The protocol's survivability is tied directly to the survivability of its underlying blockchain (e.g., "as long as Ethereum is alive, the protocol is alive").

### **4.2. Criterion: Underlying Technology Maturity**

**Description:** This criterion assesses how battle-tested the core cryptographic and software technologies underpinning the application are. Newer technology is not inherently less secure, but it carries a higher risk of "unknown unknowns" compared to technologies with a longer history of scrutiny and deployment.

- **Level 0: Experimental**
  - **Definition:** The application is one of the first few real-world, high-value deployments of a novel technology. The technology lacks a long track record of surviving adversarial conditions.
  - **Examples:**
    - Fully Homomorphic Encryption (FHE)
    - Novel ZKP schemes that are not variants of Groth16
- **Level 1: Battle-Tested**
  - **Definition:** The underlying technology has been rolled out at a significant scale and has a multi-year track record of operating in production with substantial value at stake. It has withstood public scrutiny and attack attempts.
  - **Examples:**
    - Groth16 (PLONK, STARK, and its variants are migrating towards this level as they accumulate more production time)
    - Production-grade TEE
- **Level 2: Mature**
  - **Definition:** The underlying technology is considered a standard primitive in the broader cryptographic and computer science communities. It has decades of academic analysis and has been successfully implemented across countless applications.
  - **Examples:**
    - Standard public-key cryptography (e.g., RSA, ECDSA)
    - Well-vetted hash functions (e.g., SHA-256).

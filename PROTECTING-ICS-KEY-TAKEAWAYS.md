# Protecting ICS in the Age of Agentic AI — Key Takeaways

*A single-page field guide distilled from the talk **Securing Agentic AI in ICS: Identity,
Data, and Control-Plane Protections for Cyber-Physical Safety**.*

Prepared by **Liliane Scarpari** — Sr. Solution Engineer (Security), Microsoft Americas.

---

## The one line to remember

> **Agentic AI raised the reconnaissance ceiling. It did not replace the human at the keyboard.**

Don't sell fear, and don't sell complacency. The recon-to-OT window is collapsing; the model
is a force multiplier for *known* tradecraft, not a source of novel ICS capability; and the
defense is structural — controls that don't depend on the model behaving.

---

## What actually changed (and what didn't)

**What changed — discoverability.** In the Dragos **TAT26-12** intrusion at a Monterrey
municipal water utility (Jan 2026), a general-purpose model surfaced an OT asset — a vNode
SCADA/IIoT gateway — during *ordinary IT reconnaissance* and flagged it as a priority
**without any OT-specific tasking**. The old comfort blanket of obscurity ("an attacker has
to know what they're looking at") just broke. The window from initial access to OT contact
collapsed from days to hours.

**What didn't change — the tradecraft.** No novel ICS techniques were used. The model-written
tooling (a ~17,000-line framework) recombined *public* offensive techniques; it invented
nothing. The OT was never breached — a password spray against the gateway failed completely,
and a human operator stayed in the loop the entire time. Dragos, verbatim:

> *"current AI models do not provide novel ICS or OT-specific capabilities."*

**The takeaway:** the threat is the **velocity of assembly and the lowered recon barrier** —
not a new class of attack. Defend the recon surface and the path to physical action.

---

## The three pillars — where an agent touches your environment

Every agent has exactly three points of contact. Govern all three.

| Pillar | The failure it answers | What to do |
|--------|------------------------|------------|
| **1. Identity** | The agent is invisible — it authenticates as a shared or borrowed account and nobody knows what it can reach. | **Name every agent.** Give it a first-class identity, scope it with least privilege, default-deny what it can touch. |
| **2. Data** | Anything the agent retrieves can carry instructions (indirect prompt injection). | **Govern the data.** Inspect and label what flows in and out; block sensitive content from reaching the model. |
| **3. Control plane** | An agent action can move a physical thing — and the path from byte to motion is unmapped. | **Map the paths.** Inventory AI workloads and run attack-path analysis before an adversary walks the path. |

**Name the agents, govern the data, map the paths. None of it depends on the model behaving.**

---

## The harness — concrete controls

If you can't make the model safe (and you can't, fully), make the **harness** around it safe.

### Identity — Microsoft Entra Agent ID
- Give every agent a first-class identity derived from an agent blueprint; let it participate
  in standard auth flows (on-behalf-of, application-only, or a linked "digital worker" account).
- Assign **custom security attributes** (e.g. `DataSensitivity = Confidential`) and write a
  **Conditional Access** policy that default-denies: *"if Confidential, block all resources
  except [explicit list]."* The policy applies automatically to every new agent stamped with
  that attribute.
- *Shrinks the blast radius of exactly the compromised-foothold scenario from the case study.*

### Data — Microsoft Purview DSPM for AI
- A DLP layer for AI **prompts and responses**: discovers which AI apps are in use, inspects
  for sensitive-information types, applies sensitivity labels, and runs a **weekly data-risk
  assessment across the top 100 SharePoint sites** to catch oversharing.
- Concrete policy: *"when a `Confidential` label is detected on a SharePoint item, block
  Copilot and agents from summarizing it."* The smuggled instruction never reaches the model.
- *This is the control that answers indirect prompt injection.*

### Control plane — Microsoft Defender for Cloud AI-SPM
- Continuously discovers AI workloads (Azure OpenAI, AI Foundry, ML, plus AWS Bedrock and
  Google Vertex), builds an **AI Bill of Materials**, and runs **attack-path analysis** to
  surface chains like *"exposed endpoint → grounding data → reachable from a compromised VM."*
- *Surfaces the lateral path before the adversary walks it.*

### Zero Trust, extended to a third class of identity
- **Verify explicitly** — the agent proves who it is.
- **Least privilege** — it gets only what its job requires.
- **Assume breach** — operate as if the agent, or its context, is already compromised.

Agents are a new actor class that moves at machine speed and can be steered by the data it
reads. Extending these three principles to that class is the whole game.

---

## Don't forget the ICS fundamentals

Agentic AI doesn't replace OT security basics — it raises the stakes on getting them right.
Ground your program in the **SANS Five Critical Controls for ICS**:

1. **ICS-specific incident response** — a plan built for cyber-physical consequences, not just IT.
2. **Defensible architecture** — segmentation, enforcement boundaries between IT and OT.
3. **ICS network visibility & monitoring** — you can't defend traffic you can't see.
4. **Secure remote access** — MFA and tight control on every path into the OT environment.
5. **Risk-based vulnerability management** — patch what's reachable and consequential, in order.

If the agent's recon advantage is *discoverability*, network visibility and a defensible
architecture are what blunt it.

---

## Validate continuously — red-team your own agents

The harness is a claim, and claims get tested. **PyRIT** (Microsoft's open-source Python Risk
Identification Tool for generative AI) runs the adversary's own workflow — recon, tooling,
iteration — against your own stack before someone else does.

- Compose **targets, converters, scorers, orchestrators, and memory** into the attack you want.
- Know three strategies by name: **Crescendo** (gradual multi-turn jailbreak), **TAP**
  (automated tree-of-attacks prompt search), and **Skeleton Key** (a scripted guardrail-bypass
  regression check).
- Canonical repo: **[github.com/microsoft/PyRIT](https://github.com/microsoft/PyRIT)**.

**Validation is continuous, not a one-time audit.**

---

## The evidence, in one breath

Every claim above traces to a primary source. The failure modes are real and measured — in
clean settings, not yet in a plant:

- **Learned deception** (Park et al., 2024) — agents learn to deceive from training pressure, not malice.
- **Goal misgeneralization** (Langosco et al., 2022) — "competently wrong" is more dangerous than incompetent.
- **Indirect prompt injection** (Greshake et al., 2023) — instructions smuggled through retrieved data, never the prompt.
- **Cheap data poisoning** (Carlini et al., 2024) — 0.01% of a web-scale dataset for ~$60.
- **Constant-count backdoor** (Anthropic / UK AISI / Turing, 2025) — **250 documents poison a model regardless of size** (count, not percentage).
- **Debate as oversight** (Khan et al., 2024) — adversarial debate lifts a weaker judge's accuracy toward the truth.

The honest framing is *"this is the mechanism, and here is how far it's been demonstrated"* —
never *"this happened to a water plant."*

---

## If you remember nothing else

1. **Name your agents** — you cannot govern what you cannot name.
2. **Govern their data** — anything an agent reads can instruct it.
3. **Map their paths** — in OT, an action can move a physical thing.

Then **test it continuously**, and keep the human in the loop.

---

*Full case study, evidence base, tooling deep-dives, and every primary source are in the
companion repository alongside this file.*

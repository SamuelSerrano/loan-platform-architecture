# Domain Documentation Baseline Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Complete issue #1 with canonical, traceable catalogs for business rules, events, and Event Storming without creating competing sources of truth.

**Architecture:** Organize rule and event catalogs by bounded context, then provide an end-to-end Event Storming view that references stable catalog IDs. Retain the existing combined document as the bounded-context map and executive summary; specialized Credit Decisioning documents keep authority over formulas, matrices, parameters, and internal design.

**Tech Stack:** Markdown, Mermaid, Bash, Ruby, ripgrep, Git.

## Global Constraints

- Write documentation in English.
- Use statuses `Confirmed`, `Derived`, `Proposed`, and `Deferred`.
- Use IDs `BR-<context>-NNN`, `DE-<context>-NNN`, `IE-<context>-NNN`, `CMD-<context>-NNN`, `POL-<context>-NNN`, `HS-NNN`, and `OQ-NNN`.
- Context codes are `AP`, `CI`, `CD`, `DP`, `ES`, `CO`, `LB`, `DS`, and `XS`.
- Do not duplicate formulas, thresholds, matrices, decision tables, or specialized Credit Decisioning internals.
- Do not introduce schemas/runtime code, delete documents, close issue #1, or mutate remote state without separate authorization.

## File Map

| Path | Responsibility |
| --- | --- |
| `docs/domain/BUSINESS_RULES.md` | Canonical transversal rules and invariants. |
| `docs/domain/DOMAIN_EVENTS.md` | Canonical semantic domain- and integration-event catalog. |
| `docs/domain/EVENT_STORMING.md` | Canonical operational flow, alternatives, systems, hotspots, and questions. |
| `docs/domain/BOUNDED_CONTEXTS_AND_EVENT_STORMING.md` | Context map, ownership, relationships, and journey summary. |
| `docs/discovery/DISCOVERY.md` | Discovery source-of-truth navigation. |
| `README.md` | Repository navigation and status. |

Approved design: `docs/superpowers/specs/2026-08-12-domain-documentation-baseline-design.md`.

---

### Task 1: Canonical business-rule catalog

**Files:**
- Create: `docs/domain/BUSINESS_RULES.md`
- Reference: `docs/domain/UBIQUITOUS_LANGUAGE.md`, `docs/domain/BOUNDED_CONTEXTS_AND_EVENT_STORMING.md`, `docs/domain/PRODUCT_AND_CREDIT_POLICY.md`, `docs/domain/CREDIT_DECISION_TABLE.md`, `docs/domain/CREDIT_DECISIONING_DESIGN.md`

**Interfaces:**
- Consumes: canonical terms, ownership, invariants, policy, and decision semantics.
- Produces: stable `BR-*` IDs for `EVENT_STORMING.md` and complete rule metadata.

- [ ] **Step 1: Inventory source rules**

Run: `rg -n '^## |^### |invariant|must |must not|cannot|only |at most|required|prohibited|expired|idempoten' docs/domain/*.md`

Expected: candidate rules and source locations for every bounded context.

- [ ] **Step 2: Create the document skeleton**

Create sections for purpose/authority, reading conventions, status model, all nine context codes, deferred rules, and governance. Each context uses this schema:

```markdown
| ID | Rule | Status | Owner | Inputs | Constraint / condition | Outcome | Reason code | Source |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
```

- [ ] **Step 3: Populate and trace rules**

Start each context at `001`. Mark direct statements `Confirmed`. Mark necessary inferences `Derived` and explain each below its table. Link the narrowest authoritative section; reference calculations and matrices instead of copying them.

- [ ] **Step 4: Validate coverage and uniqueness**

Run: `for code in AP CI CD DP ES CO LB DS XS; do rg -q "BR-${code}-[0-9]{3}" docs/domain/BUSINESS_RULES.md || exit 1; done`

Run: `rg -o 'BR-[A-Z]{2}-[0-9]{3}' docs/domain/BUSINESS_RULES.md | sort | uniq -d`

Expected: every context appears and duplicate output is empty.

- [ ] **Step 5: Commit**

Run: `git add docs/domain/BUSINESS_RULES.md && git commit -m "Document canonical business rules"`

Expected: a commit containing only the new rule catalog.

### Task 2: Canonical event catalog

**Files:**
- Create: `docs/domain/DOMAIN_EVENTS.md`
- Reference: `docs/domain/UBIQUITOUS_LANGUAGE.md`, `docs/domain/BOUNDED_CONTEXTS_AND_EVENT_STORMING.md`, `docs/domain/CREDIT_DECISIONING_DESIGN.md`, `docs/domain/BUSINESS_RULES.md`

**Interfaces:**
- Consumes: event vocabulary, ownership, terminology, and `BR-*` IDs.
- Produces: stable `DE-*` and `IE-*` IDs with producer, consumers, trigger, payload, exclusions, and versioning.

- [ ] **Step 1: Inventory existing events**

Run: `rg -n 'Started|Submitted|Verified|Rejected|Requested|Recorded|Created|Accepted|Expired|Approved|Signed|Reserved|Confirmed|Failed|Activated|Cancelled|Superseded' docs/domain/*.md`

Expected: established event names and source locations are visible before classification.

- [ ] **Step 2: Create separate catalog sections**

Create sections for purpose, classification, status/versioning, domain events by context, integration events by context, envelope/sensitive-data constraints, and governance. State that internal domain events may be unversioned, public contract names use `.v1`, and future schemas belong to `loan-platform-contracts`.

Domain-event schema:

```markdown
| ID | Event | Status | Producer | Trigger | Minimum payload | Related rules | Source |
| --- | --- | --- | --- | --- | --- | --- | --- |
```

Integration-event schema:

```markdown
| ID | Contract name | Status | Producer | Consumers | Trigger | Minimum payload | Excluded data | Versioning | Related rules | Source |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
```

- [ ] **Step 3: Populate events**

Preserve established `.v1` names. Mark inferred internal facts `Derived` with an explanation and newly named public events `Proposed`. Exclude OTP secrets, raw identity evidence, full PII, document bytes, and provider credentials.

- [ ] **Step 4: Validate**

Run: `rg -o '(DE|IE)-[A-Z]{2}-[0-9]{3}' docs/domain/DOMAIN_EVENTS.md | sort | uniq -d`

Run: `rg -n 'Domain events|Integration events|\.v1|OTP secrets|raw identity evidence|full PII|document bytes|provider credentials' docs/domain/DOMAIN_EVENTS.md`

Expected: no duplicate IDs; distinct classes, versioning, and exclusions are explicit.

- [ ] **Step 5: Commit**

Run: `git add docs/domain/DOMAIN_EVENTS.md && git commit -m "Document domain and integration events"`

### Task 3: Canonical Event Storming model

**Files:**
- Create: `docs/domain/EVENT_STORMING.md`
- Reference: `docs/domain/BUSINESS_RULES.md`, `docs/domain/DOMAIN_EVENTS.md`, `docs/discovery/ASSUMPTIONS.md`

**Interfaces:**
- Consumes: `BR-*`, `DE-*`, `IE-*`, context relationships, and open questions.
- Produces: `CMD-*`, `POL-*`, `HS-*`, `OQ-*`, five phases, and recovery paths.

- [ ] **Step 1: Create the structure**

Create sections for purpose, conventions, actors/external systems, phases A–E, alternate/recovery paths, cross-cutting policies, hotspots, open questions, and governance.

- [ ] **Step 2: Catalog actors and external systems**

Include Applicant, Customer, Signer, Credit Analyst, Administrator, Identity Provider, Disbursement Provider, Cognito, EventBridge, SQS queues, per-consumer DLQs, protected object storage, and simulated notification channels. Classify and source each entry.

- [ ] **Step 3: Model five happy-path phases**

Use:

```markdown
| Step | Actor / trigger | Command | Aggregate | Domain event | Policy | Integration event | Consumer | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
```

Give commands `CMD-*` IDs and reactions `POL-*` IDs. Reference rather than redefine `BR-*`, `DE-*`, and `IE-*`. Keep Application Process as coordinator and decisions in their owning contexts.

- [ ] **Step 4: Model alternate/recovery paths**

Cover validation/consent failure; identity rejection/unavailability; incomplete, pending, and technical assessment outcomes; unfavorable decision; offer rejection/expiry; document correction; invalid/exhausted/expired OTP; signature expiry; transient/terminal/unknown disbursement outcomes; DLQ; reconciliation; manual recovery. Record owner, detected fact, permitted reaction, forbidden shortcut, retry/idempotency, and outcome. Operational failure cannot become an unfavorable credit decision; compensation is explicit.

- [ ] **Step 5: Consolidate hotspots and questions**

Use schema `ID | Status | Owner / decision maker | Topic | Impact | Decision required | Source`. Link matching `ASSUMPTIONS.md` entries and do not invent resolutions.

- [ ] **Step 6: Validate references and IDs**

Run: `rg -o '(BR|DE|IE)-[A-Z]{2}-[0-9]{3}' docs/domain/EVENT_STORMING.md | sort -u > /tmp/issue-1-refs.txt`

Run: `rg -o '(BR|DE|IE)-[A-Z]{2}-[0-9]{3}' docs/domain/BUSINESS_RULES.md docs/domain/DOMAIN_EVENTS.md | sed 's/^.*://' | sort -u > /tmp/issue-1-defs.txt`

Run: `comm -23 /tmp/issue-1-refs.txt /tmp/issue-1-defs.txt`

Run: `rg -o '(CMD|POL)-[A-Z]{2}-[0-9]{3}|(HS|OQ)-[0-9]{3}' docs/domain/EVENT_STORMING.md | sort | uniq -d`

Expected: no unresolved or duplicate ID output.

- [ ] **Step 7: Commit**

Run: `git add docs/domain/EVENT_STORMING.md && git commit -m "Document canonical event storming model"`

### Task 4: Rebalance authority and navigation

**Files:**
- Modify: `docs/domain/BOUNDED_CONTEXTS_AND_EVENT_STORMING.md`
- Modify: `docs/discovery/DISCOVERY.md`
- Modify: `README.md`

**Interfaces:**
- Consumes: the three new canonical documents.
- Produces: a context map without parallel catalogs and complete navigation.

- [ ] **Step 1: Refocus the combined document**

Rename its title to `Bounded Context Map`. Retain classification, map, ownership, relationships, aggregate summary, read models, repository boundaries, and walking-skeleton summary. Replace detailed flow, event, policy, invariant, failure, hotspot, and state catalogs with summaries and links. Retain context-level Mermaid diagrams and add `Document authority`.

- [ ] **Step 2: Update Discovery**

Add the three new documents with one-sentence authority descriptions. Update current-position and next-deliverable language so issue #1 outputs are no longer described as missing.

- [ ] **Step 3: Update README**

Add the three links under `Domain`. Update status only as justified; do not claim architecture, contracts, or implementation completion.

- [ ] **Step 4: Validate authority and navigation**

Run: `rg -n 'Public integration-event catalog|Big-picture event storming|Key policies|Hotspots and decisions required' docs/domain/BOUNDED_CONTEXTS_AND_EVENT_STORMING.md`

Run: `rg -n 'BUSINESS_RULES.md|DOMAIN_EVENTS.md|EVENT_STORMING.md' README.md docs/discovery/DISCOVERY.md docs/domain/BOUNDED_CONTEXTS_AND_EVENT_STORMING.md`

Expected: no detailed-catalog headings remain; all three navigation documents link the new sources.

- [ ] **Step 5: Commit**

Run: `git add README.md docs/discovery/DISCOVERY.md docs/domain/BOUNDED_CONTEXTS_AND_EVENT_STORMING.md && git commit -m "Clarify domain documentation authority"`

### Task 5: Full validation and review

**Files:** Validate every changed Markdown file against issue #1 and the approved design.

**Interfaces:**
- Consumes: all deliverables.
- Produces: evidence for links, IDs, metadata, terminology, scope, and Git hygiene.

- [ ] **Step 1: Validate relative links and GitHub-style anchors**

Run:

```bash
ruby -E UTF-8:UTF-8 -e 'broken=[]; Dir.glob("**/*.md").sort.each{|f| File.read(f,encoding:"UTF-8").scan(/!?\[[^\]]*\]\(([^)]+)\)/).flatten.each{|r| t=r.strip.sub(/\A</,"").sub(/>\z/,""); next if t=~/\A(?:https?:|mailto:|#)/; p=t.split("#",2).first; next if p.empty?; broken << "#{f}: #{t}" unless File.exist?(File.expand_path(p,File.dirname(f)))}}; abort broken.join("\n") unless broken.empty?; puts "All relative Markdown links resolve."'
```

Expected: `All relative Markdown links resolve.`

Then validate every relative fragment against the target document's GitHub-style heading slug, including duplicate-heading suffixes. The check must fail for a missing file or missing anchor and print each unresolved source and target.

Run:

```bash
ruby -E UTF-8:UTF-8 -e 'files=Dir.glob("**/*.md").sort; anchors={}; files.each{|f| counts=Hash.new(0); known={}; File.foreach(f,encoding:"UTF-8"){|line| next unless line=~/^ {0,3}\#{1,6}\s+(.+?)\s*\#*\s*$/; slug=$1.downcase.gsub(/<[^>]*>/,"").gsub(/[^\p{L}\p{N}\s_-]/u,"").strip.gsub(/\s/,"-"); suffix=counts[slug]; counts[slug]+=1; slug="#{slug}-#{suffix}" if suffix>0; known[slug]=true}; anchors[f]=known}; broken=[]; files.each{|f| File.read(f,encoding:"UTF-8").scan(/!?\[[^\]]*\]\(([^)]+)\)/).flatten.each{|raw| target=raw.strip.sub(/\A</,"").sub(/>\z/,""); next if target=~/\A(?:https?:|mailto:)/; path,fragment=target.split("#",2); resolved=path.nil?||path.empty? ? f : File.expand_path(path,File.dirname(f)).sub(%r{\A#{Regexp.escape(Dir.pwd)}/},""); unless File.exist?(resolved); broken << "#{f}: #{target} (missing file)"; next; end; broken << "#{f}: #{target} (missing anchor)" if fragment&&!fragment.empty?&&!anchors[resolved]&.key?(fragment.downcase)}}; abort broken.join("\n") unless broken.empty?; puts "All relative Markdown paths and GitHub-style heading anchors resolve."'
```

Expected: all relative Markdown paths and heading anchors resolve.

- [ ] **Step 2: Validate IDs globally**

Run: `rg -o '(BR|DE|IE|CMD|POL)-[A-Z]{2}-[0-9]{3}|(HS|OQ)-[0-9]{3}' docs/domain/*.md | sed 's/^.*://' | sort | uniq -c`

Expected: definitions are unique; repeated occurrences are reviewed references to those definitions.

- [ ] **Step 3: Validate required schemas**

Run: `rg -n '^\| ID \| Rule \| Status \| Owner \| Inputs \| Constraint / condition \| Outcome \| Reason code \| Source \|$' docs/domain/BUSINESS_RULES.md`

Run: `rg -n '^\| ID \| Event \| Status \| Producer \| Trigger \| Minimum payload \| Related rules \| Source \|$' docs/domain/DOMAIN_EVENTS.md`

Run: `rg -n '^\| ID \| Contract name \| Status \| Producer \| Consumers \| Trigger \| Minimum payload \| Excluded data \| Versioning \| Related rules \| Source \|$' docs/domain/DOMAIN_EVENTS.md`

Expected: every canonical schema is found.

- [ ] **Step 4: Validate hygiene, scope, and secrets**

Run: `git diff main...HEAD --check`

Run: `git diff main...HEAD --name-only`

Run: `rg -n -i --hidden --glob '!.git/**' '(github_pat_|gh[pousr]_[A-Za-z0-9]{20,}|AKIA[0-9A-Z]{16}|-----BEGIN [A-Z ]+PRIVATE KEY-----|xox[baprs]-[A-Za-z0-9-]{10,})' .`

Expected: no new whitespace errors; only the approved spec, plan, three catalogs, and three navigation/authority files changed; no secrets.

- [ ] **Step 5: Review acceptance criteria manually**

Confirm complete rule metadata; separated event classes; complete event metadata; Event Storming coverage; preserved specialized authority; terminology matching `UBIQUITOUS_LANGUAGE.md`; valid navigation and links.

- [ ] **Step 6: Commit validation fixes only if needed**

Run when files changed: `git add README.md docs/discovery/DISCOVERY.md docs/domain/*.md && git commit -m "Validate domain documentation baseline"`

Expected: no empty commit.

- [ ] **Step 7: Present the local branch**

Run: `git status --short --branch && git log --oneline --decorate main..HEAD && git diff --stat main...HEAD`

Expected: clean working tree and reviewable local commits; request authorization before push or issue changes.

### Designing a Production-Grade Python Workspace for AI Agents—in One Day

We (ChatGPT and I) spent a single day **architecting**—not coding—a modern, production-hardened development environment that an army of AI agents (with humans in the loop) can extend safely and predictably.  
The target: an opinionated mono-repo where agents can

- spin up with **one** command,
    
- add new tools through recipes instead of cargo-cult copy-paste, and
    
- ship to Kubernetes with the exact artifact they ran locally.
    

The outcome isn’t a finished codebase; it’s a **detailed blueprint**—security gates, release paths, docs pipelines, everything senior engineers expect—ready for parallel implementation. Thanks to today’s reasoning models, this fit into a single (long) design session.

### One Day, Many Decisions — Laying the Tracks for Velocity

Yesterday’s goal was to **eliminate future yak-shaving**: define every guard-rail, automation, and convention before a line of product code exists. We approached the task the way a performance engineer tunes a tight loop: **identify every cycle that will repeat thousands of times in the life of the repo and squeeze out the slow, manual steps while the cost is still near zero.**

- **Commit loops** Pre-commit hooks fire in < 200 ms, telling an agent the _moment_ it introduces a style or type error. No “push, wait for CI, scroll through logs” dance.
    
- **Dependency loops** Renovate auto-groups upgrades and ships them behind the same tests, so staying current costs minutes—not afternoons of “pip conflict whack-a-mole.”
    
- **Release loops** Semantic-release tags, bumps, and writes the changelog in one CI job; humans review value, not version strings.
    
- **Security loops** Gitleaks, Bandit, CodeQL, Trivy—each positioned at the earliest stage it can run. In today’s APT-ridden world you don’t build a world-class installation and leave the doors unlocked. Security isn’t a checkbox after MVP; it’s a guard-rail that lets agents run unattended.

The trade-off lens was always **“long-term delta of secure-and-maintainable iteration speed.”**  
A tool that shaves 15 s from every CI run but takes two days to debug once a year is worth it; one that saves a minute today but adds uncertainty six months out is not. Immediate, low-latency feedback keeps humans and models in flow, while proactive security ensures that velocity never comes at the expense of trust.

Security isn’t a checkbox after MVP; it’s a guard-rail, meaning we want to move to a world where we can let agents run unattended. But let's be clear here, _guard-rails aren’t guardhouses_. Automated scans and policy gates dramatically reduce risk, **but they don’t (yet) eliminate the need for human oversight, defence-in-depth, and incident response drills.**  
Think of Gitleaks, Trivy, or CodeQL as the _seatbelts and airbags_—indispensable, but not a licence to drive blindfolded. We still:

- review PRs touching critical paths,
    
- rotate keys and audit logs,
    
- patch base images promptly, and
    
- rehearse breach scenarios.
Automating the _boring_ parts of security gives humans (and higher-level agents) the bandwidth to focus on the nuanced threats no static tool can spot.

And a note to those (justifiably) screaming "YAGNI!" (“you aren’t gonna need it”)-- which is solid advice for human-based development: with AI agents pushing commits 24/7, however, the penalty for missing guard-rails can be paid orders of magnitude faster than in a human-only shop. So we borrowed from decades of hard-won ops intuition and front-loaded the decisions that history says _always_ resurface: security gates, reproducible builds, provenance, contract tests.

#### Design-up-front isn’t Waterfall 2.0.  
Classic waterfall assumed humans, long spec cycles, and expensive change; **Waterfall + Agents is a different physics**: agents iterate continuously, but they need clear, deterministic rules to stay in bounds. We design those bounds first, _then_ we let the agents loose. Think of it as laying down the rails before running the bullet train—it’s still “move fast,” just not “move fast and hope for a miracle.” _(Or, to borrow SpaceX’s flavor of the mantra: **move fast and break staging**, not production.)_
#### What we were solving for

- **Cycle-time as the metric** – an AI agent already spends ~8 min per Codex run; CI can’t cost another 8. Every tool was asked, _“Will feedback stay tightly bounded?”_. Longer term, we can shift more expensive CI steps to run only pre-prod release, but this adds the burden of merge conflicts, all the headaches associated with rollbacks, and history-rewriting (for developers working on the already merged commits). There's just no way to know where this sweet spot lies--we need experimentation--so our initial goal is to set up tweakability rather than to converge on the "correct answer" up-front. When you work in distributed systems, you quickly realize that the "right answer" is dynamically coupled to the system as a whole, meaning it changes over time as your system evolves.
    
- **Shift-left robustness** – security, typing, docs, performance all automated from commit #1, so complexity grows linearly. We move checks and safeguards as close to the developer’s keyboard as possible (or in this case, an AI-agent's CLI). We want to build the tooling so that quality, security, performance, and compatibility problems are surfaced _early, automatically, and with low latency_. The hypothesis here is that with AI-agents, the extra setup complexity immediately pays off (rather than over the long-term with human-based development).
    
- **Agent-friendly scaffolds** – We assume future contributors will be autonomous LLMs with zero tribal knowledge, so we **replace unwritten norms with deterministic recipes**: every repetitive action (creating a new component, running tests, shipping an image) has a **deterministic, one-command recipe**; every new capability advertises itself through a **single declarative contract**. That means an autonomous agent doesn’t have to reverse-engineer file layouts or tribal conventions—it can discover what exists, generate what’s missing, and stay entirely inside guard-railed workflows. The mental overhead is pushed out of the agent’s reasoning loop and into stable, reproducible scaffolds. 
#### The design backlog we produced

1. **Environment cornerstone** – `bootstrap.sh` + uv lockfile; automated pre-commit hooks.
    
2. **Quality gates** – Ruff lint/format, Ty static types, Typeguard runtime checks, pytest-bdd / Hypothesis tests, coverage baseline.
    
3. **Security & compliance** – pip-audit, Bandit, CodeQL, Gitleaks, Trivy image scan, license allow-list.
    
4. **Release & dependency flow** – Commitizen, semantic-release, Renovate, Python Coverage Comment, performance-benchmark guard.
    
5. **Documentation & DX** – Sphinx + MyST + furo, Autodoc / Napoleon, sphinx-click CLI docs, Docker-rendered Mermaid diagrams, Codespaces dev-container.
    
6. **Runtime & orchestration** – Signed Docker image + SBOM, K8s manifests, Tilt local loop, Hatch workspaces for future packages.
    
7. **AI-agent layer** – LangGraph backbone, `AGENTS.md` registry, JSON-Schema export, contract tests, per-agent Cookiecutter generator.
    
8. **Workflow polish** – Justfile command palette, GitHub label strategy, issue/PR templates, “hello-world” CI.
#### Trade-off heuristics

- **Speed over convention** – uv + Ruff beat slower incumbents.
    
- **Low lock-in** – every big choice has an architecture design record (ADR); swap cost is explicit.
    
- **Single-source truth** – Pydantic models generate GraphQL, JSON Schema, and docs.
    
- **Automation first, policy later** – start with visible but permissive gates; tighten over time.
#### Advice if you try this

- Treat a mature template (e.g. [Hypermodern Python Cookiecutter](https://github.com/cjolowicz/cookiecutter-hypermodern-python-template)) as an **idea checklist**, not a blueprint.
    
- Ask of every tool: _“Will this shorten or lengthen the inner feedback loop six months from now?”_
    
- Record **why** in ADRs—future maintainers (human or LLM) won’t share today’s context.
    
- Assume today’s best pick may be obsolete next quarter; design exits, not permanence.
    

With the stories queued in GitHub Projects, the repo is now a **map**, not yet the territory. In the next blog post we will cover implementation using AI agents, which will bring its own series of challenges (and uncover cracks in our approach). But by clearly demarcating the cutpoint between high-level-decisionmaking/design and implementation, it gives us valuable insight into our high-level decisionmaking _process_. 

It's not hard to imagine that in the coming years, or even quarters, AI agents will help improve not just our decisions, but the high-level decisionmaking process itself. Documenting the process helps us refine our strategy down the road--and strategy is generic and mostly domain-independent, so any wins here transfer over to all future projects. In a sense, the hard decisions have already been made, documented, and ready for parallel execution.

The backlog now serves as the **map**, not the territory.  
Next we’ll unleash agents to implement these stories—measuring whether the theory holds up in practice. By capturing _why_ each choice was made, we’ve created a playbook future projects (and models) can refine rather than rediscover.
### Appendix 

We now have a bunch of stories, whose implementation importance follows from this dependency order: foundational environment first, then CI/security gates, then docs/runtime, and finally agent-specific scaffolds. Here's what that looks like, roughly, in a pre-order of dependency:
#### Implementation Roadmap 

- **0001 – ADR Process**  
    _Defines the template and rule: every foundational decision requires an ADR._
    
- **0002 – Bootstrap with `uv`**  
    _One-command environment and lockfile; cold install in seconds._
    
    - **0003 – Optional Pre-commit Install** – ask/flag in `bootstrap.sh`.
        
    - **0004 – Justfile Core Recipes** – canonical `just lint | test | bench`.
        
    - **0005 – Dev-container / Codespaces** – zero-setup cloud IDE.
        
    - **0006 – Hatch Workspaces** – future multi-package monorepo.
        
    - **0007 – Contributor Experience Kit** – templates, CONTRIBUTING, CoC.
        
- **0008 – Nox Task-Runner**  
    _Pythonic orchestration for tests, types, security._
    
    - **0009 – BDD + Hypothesis Tests** – behaviour & property checks.
        
        - **0010 – Ty Static Types** – fast strict type-checking.
            
        - **0011 – Typeguard Runtime Checks** – catch type drift.
            
        - **0012 – Coverage Baseline** – XML/HTML reports.
            
            - **0013 – Coverage Comment Bot** – inline PR diffs.
                
        - **0014 – Performance Guard** – pytest-benchmark 10 % fail gate.
            
    - **0015 – Ruff Lint / Format** – one Rust tool replaces three.
        
        - **0016 – Ruff Pyupgrade Rules** – auto-modernise syntax.
            
    - **0017 – Bandit Security Scan** – static Python vulns.
        
    - **0018 – pip-audit Scan** – supply-chain CVEs on lockfile.
        
- **0019 – GitHub Label Strategy** – risk lanes, changelog sections.
    
- **0020 – Baseline GitHub Actions** – hello-world CI skeleton.
    
    - **0021 – CodeQL Analysis** – deep taint-flow scanning.
        
    - **0022 – Trivy Scan & SBOM** – image CVEs + CycloneDX.
        
    - **0023 – Gitleaks Secret Scan** – blocks secrets in CI.
        
    - **0024 – License Compliance** – SPDX allow-list.
        
- **0025 – Commitizen Conventional Commits** – commit wizard.
    
    - **0026 – Semantic-release** – auto version & changelog.
        
- **0027 – Renovate Dependency Updates** – grouped upgrade PRs.
    
- **0028 – Sphinx + MyST + furo Docs** – Markdown-first docs site.
    
    - **0029 – Autodoc & Napoleon** – API refs from docstrings.
        
    - **0030 – sphinx-click CLI Docs** – Typer command tree.
        
    - **0031 – Docker Mermaid Pipeline** – offline diagram render.
        
- **0032 – Docker Image (signed + SBOM)** – distro-less Python base.
    
    - **0033 – Tilt Dev Loop** – <2 s live reload to local K8s.
        
- **0034 – `AGENTS.md` Registry** – discoverable tool inventory.
    
    - **0035 – Cookiecutter Agent Template** – one-command scaffold.
        
    - **0036 – LangGraph Backbone** – structured-tool orchestration.
        
    - **0037 – JSON-Schema Export** – persisted Pydantic schemas.
        
        - **0038 – Schemathesis Fuzz** – HTTP contract tests.
            
        - **0039 – JSON-Schema Contract Tests** – message validation.
            
    - **0040 – Hypothesis GraphQL Fuzz** – SDL ↔ resolver guard.
        
- **0041 – Single-Schema Bridge (Pydantic ↔ Strawberry)**  
    _Eliminates drift across REST, GraphQL, and OpenAI tools._
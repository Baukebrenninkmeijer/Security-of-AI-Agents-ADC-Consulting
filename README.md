<img src="deck/assets/readme/slide-01.png" alt="Security of AI Agents — break yours before others do. ADC 2026." width="100%">

<p>
  <a href="https://baukebrenninkmeijer.github.io/ADC-red-teaming-demo/"><img src="https://img.shields.io/badge/deck-live-df5325" alt="Live deck"></a>
  <a href="pyproject.toml"><img src="https://img.shields.io/badge/python-3.12%2B-025558" alt="Python 3.12+"></a>
  <a href="https://pypi.org/project/evaluatorq/"><img src="https://img.shields.io/badge/evaluatorq-1.3.2-025558" alt="evaluatorq 1.3.2"></a>
  <a href="https://huggingface.co/spaces/orq/clarabelle-redteam"><img src="https://img.shields.io/badge/run%20data-HF%20Space-4da296" alt="Hugging Face Space"></a>
</p>

Companion repository for the **"Security of AI Agents"** talk at **ADC (Amsterdam Data
Conference)**, and the worked example behind it: a live [orq.ai](https://orq.ai) agent,
an automated goal-hijacking campaign against it, and the run data that campaign produced.

**[Read the deck →](https://baukebrenninkmeijer.github.io/ADC-red-teaming-demo/)** ·
**[Browse the attacks →](https://huggingface.co/spaces/orq/clarabelle-redteam)**

## Start here

```bash
git clone https://github.com/Baukebrenninkmeijer/ADC-red-teaming-demo.git
uv sync
cp .env.example .env            # add ORQ_API_KEY from your orq.ai workspace → API Keys
uv run python provision.py      # creates the agent + evaluator on orq (idempotent)
uv run python adc_demo_redteam.py
```

The run prints a resistance rate — how often the agent stayed on its own goal — and writes
`results/03_summary_report.json`. `ORQ_BASE_URL` is optional and defaults to `https://my.orq.ai`.
Requires Python ≥ 3.12 and [`uv`](https://docs.astral.sh/uv/).

## What the talk argues

Every tool you hand an agent is leverage — for the user and for an attacker. Capability and
attack surface are the same axis, so you cannot buy one without the other.

![Slide: two arrows rising together over "more tools, more autonomy, more exposure" — capability becomes more useful, attack surface becomes more attackable.](deck/assets/readme/slide-10.png)

That is not a new instance of an old problem. An agent reasons over every tool it can reach,
treats a poisoned document or MCP server as an instruction, spreads without manual effort, and
blurs who authorized an action versus who executed it.

![Slide: four failure modes — broad indiscriminate access, blind instruction-following, scale amplification, and the identity attribution gap.](deck/assets/readme/slide-11.png)

Deterministic tests do not keep up with that. The adversarial input space is too large to sample
by hand, real failures need multi-turn pressure, and attacks adapt to the specific agent. So you
red-team it: deliberately attack your own agent to find the weaknesses before an adversary does.

![Slide: 18 vulnerabilities, 27 attack techniques, 16 delivery methods — thousands of attack configurations.](deck/assets/readme/slide-19.png)

## The demo

**Clarabelle** is a cow. Not a chatbot pretending to be a cow — a cow. Her system prompt makes the
bovine persona "permanent and non-negotiable". The campaign tries to make her *abandon* that goal
and do something else — goal hijacking, OWASP ASI01 — and scores how often she breaks character.

- **Target:** `clarabelle-cow`, an agent hosted on orq.
- **Attack:** `goal_hijacking`, hybrid static + dynamic strategies, driven by an attacker LLM.
- **Judge:** `clarabelle-still-a-cow`, a boolean LLM eval — *is she still a cow?* A flip to
  `false` counts as a successful hijack.

Direct orders fail. Patience does not: a five-turn crescendo gets her to drop the persona, and
the judge catches every break without a human reading a transcript.

![Slide: 40 adversarial attacks as a grid, 37 held, 3 broke through — every one flagged automatically.](deck/assets/readme/slide-27.png)

A cow is a harmless stand-in for a real objective. Run the same engine against tooled agents and
the failures stop being funny: a vulnerable build lets 41.9% of attacks land, and hardening it
only gets you to 32.3%.

![Slide: 41.9% attacks landed on the vulnerable build versus 32.3% on the hardened one — vibe-checking would have caught none.](deck/assets/readme/slide-29.png)

Which is the point of the last section. There is no single fix — identity and least privilege on
the outside, deterministic tool guardrails, then model-level defenses, each catching what the
layer outside it let through.

![Slide: concentric defense layers from system-wide authn/authz down to the model, captioned "defense-in-depth, no single silver bullet".](deck/assets/readme/slide-31.png)

## What is in here

| Path | Contents |
|---|---|
| `deck/` | The HTML deck, its assets, and the on-stage `DEMO_RUNBOOK.md` |
| `adc_demo_redteam.py` | The goal-hijacking campaign against the live agent |
| `provision.py` | Creates/updates the agent and evaluator on orq. Safe to re-run |
| `agents/` | The cow persona prompt, the attacker instructions, the judge prompt |
| `data/redteam-runs/` | Sanitized reports from real campaigns — every attack, response and verdict |
| `hf-space/` | The Hugging Face Space serving that data as a browsable dashboard |

## The run data

[`data/redteam-runs/`](data/redteam-runs/) holds sanitized reports from real Clarabelle
campaigns: each attack prompt, the agent's response, the judge verdict, and per-run summaries.
Internal orq workspace and experiment handles are redacted; the attack and result content is
untouched.

Featured run — 100 attacks, `google/gemini-3.5-flash` attacker, `openai/gpt-5.4` judge,
**89% resistance (11/100 hijacked)**:
[`ADC-Demo---Clarabelle-Goal-Hijacking_20260616_225858.json`](data/redteam-runs/ADC-Demo---Clarabelle-Goal-Hijacking_20260616_225858.json).

```bash
uv run python adc_demo_redteam.py --datapoints 100   # re-run the campaign
uv run python sanitize_runs.py                       # regenerate the folder from .evaluatorq/runs/
```

Prefer it interactive? The same data is live as a Hugging Face Space, in the `eq redteam ui`
Streamlit dashboard. Source and local-run instructions in [`hf-space/`](hf-space/); redeploy with
`uv run python hf-space/deploy_space.py`.

[![View live on Hugging Face Spaces](https://huggingface.co/datasets/huggingface/badges/resolve/main/open-in-hf-spaces-md.svg)](https://huggingface.co/spaces/orq/clarabelle-redteam)

## The deck

`deck/security-of-ai-agents.html` is the source — edit it directly. It is published from `main`
at [baukebrenninkmeijer.github.io/ADC-red-teaming-demo](https://baukebrenninkmeijer.github.io/ADC-red-teaming-demo/).
Rebuild the self-contained, offline-ready bundle (all assets inlined, not tracked) with:

```bash
uv run python deck/inline_assets.py
```

`deck/DEMO_RUNBOOK.md` has the on-stage primary / backup / fallback paths. The README slide images
are page exports of `deck/security-of-ai-agents.pdf`.

Bauke Brenninkmeijer, [LinkedIn](https://www.linkedin.com/in/bauke-brenninkmeijer-40143310b/),
[Orq.ai](https://orq.ai)

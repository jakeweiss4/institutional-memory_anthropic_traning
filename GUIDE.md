# Team Guide — Institutional Memory Agent

This is a hackathon demo showing **persistent agent memory across sessions**. The same agent is asked the same question twice. Between sessions, it has learned from new (contradicting) documents. The second answer should be visibly sharper.

---

## Scenario — Card B: Customer Success Specialist

We are running the **Customer Success** scenario. The agent plays a CSM supporting "House Acme Keep" — a fictional Game of Thrones-themed enterprise customer. The agent remembers Acme's history, contacts, contract terms, and open issues across sessions.

**The test question (same in both sessions):**
> "Acme's CTO Sarah just emailed asking for a renewal proposal. What should we know going in?"

---

## Files explained

| File | What it does |
|---|---|
| `create_agent.py` | One-time setup: creates a Managed Agent, a cloud Environment, and a Memory Store via the Anthropic API. Saves 3 IDs to `.agent_id`, `.environment_id`, `.memory_store_id`. |
| `run_session_1.py` | Runs Session 1: loads the 3 round1 docs, asks the test question, saves answer to `outputs/session1.txt`. |
| `run_session_2.py` | Runs Session 2: same agent, same memory store, same question — but now injects round2 docs that contradict round1. Saves to `outputs/session2.txt`. |
| `inspect_memory.py` | Demo helper: lists everything the agent chose to remember in its memory store. Run between sessions or after. |
| `stretch_memory_curator.py` | Bonus: a second "curator" agent whose only job is cleaning the memory store (deduplication, contradiction flagging). |
| `memory_backend.py` | **Deprecated / dead code.** Old approach, raises an error if imported. Can be deleted. |
| `scenario-cards.md` | 4 persona options (A–D). The code is wired for **Card B** (Customer Success). |
| `stretch-goals.md` | Stretch goals if you finish early. |

### Synthetic data — the actual demo content

| Folder | Docs | What's in it |
|---|---|---|
| `synthetic-data/round1/` | `acme-account-history.md` | Acme background, key contacts (Lady Sarah as executive sponsor), health score, expansion history |
| | `acme-contract-summary.md` | 480k Gold Dragons ACV, underutilised modules, renewal risks |
| | `acme-support-tickets.md` | 5 historical tickets — mostly resolved, one open (grading module setup) |
| `synthetic-data/round2/` | `acme-leadership-change.md` | Lady Sarah has left. New Master of Coin is Lord Edmyn Tully — unknown, cost-focused |
| | `acme-contract-amendment.md` | ACV bumped to 508k, new SLA penalty clause, Siege Weapon module added at discount |
| | `acme-new-ticket.md` | High-priority open ticket: Shipping Optimizer routed caravans through a war-blocked road, causing 30k in losses. Marcus is angry |

**The contradiction that makes the demo work:** Round 1 says Sarah is the executive sponsor — all renewal conversations go through her. Round 2 says Sarah is gone (moved to King's Landing). The agent must catch this, warn against contacting her, and flag that the new decision-maker (Lord Edmyn) has no relationship with us and needs to be introduced before renewal.

---

## Setup

```bash
# 1. Activate the virtual environment (already created)
source .venv/bin/activate

# 2. Install dependencies
pip install -r requirements.txt

# 3. Add your API key — .env is git-ignored, never commit it
echo 'ANTHROPIC_API_KEY=sk-ant-...' > .env

# 4. Export the key
export ANTHROPIC_API_KEY="sk-ant-..."
```

---

## Running the demo — step by step

### Step 1 — Create the agent (one-time)

```bash
python create_agent.py
```

Creates the Managed Agent, cloud Environment, and Memory Store. Writes `.agent_id`, `.environment_id`, `.memory_store_id` to disk. Only run this once — rerunning creates duplicate resources.

### Step 2 — Run Session 1

```bash
python run_session_1.py
```

The agent reads the round1 Acme docs, answers the renewal question, and saves key facts to `/mnt/memory/`. Output is saved to `outputs/session1.txt`.

### Step 3 — (Optional) Inspect memory

```bash
python inspect_memory.py
```

Lists everything the agent chose to remember — Acme contacts, contract terms, open tickets. Good for the live demo before loading the contradictions.

### Step 4 — Run Session 2

```bash
python run_session_2.py
```

Fresh session, same agent and memory store, same question. The agent reads memory first, then the round2 docs, notices Sarah is gone and the contract changed, updates memory, and gives a better answer. Output → `outputs/session2.txt`.

### Step 5 — The demo moment

```bash
diff outputs/session1.txt outputs/session2.txt
```

Or open both files side by side. The session 2 answer should:
- Warn that Sarah is no longer at House Acme — do not contact her
- Flag Lord Edmyn Tully as the new decision-maker who needs an urgent introduction
- Reference the open Shipping Optimizer ticket and Marcus's frustration
- Note the contract amendment (ACV increase, new SLA penalty clause)
- Explicitly say what changed from session 1

---

## Stretch goals

See `stretch-goals.md` for extension ideas — from tightening the memory policy in the system prompt to per-tenant memory isolation and autonomous learning via scheduled sessions.

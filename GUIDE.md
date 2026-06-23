# Team Guide — Institutional Memory Agent

This is a hackathon demo showing **persistent agent memory across sessions**. The same agent is asked the same question twice. Between sessions, it has learned from new (contradicting) documents. The second answer should be visibly sharper.

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
| `scenario-cards.md` | 4 persona options (A–D). The code is wired for **Card A** (New-Hire Onboarding). Cards B–D exist as descriptions only — no data or code yet. |
| `stretch-goals.md` | 9 stretch goals across 4 tiers if you finish early. |

### Synthetic data — the actual demo content

| Folder | Docs | State |
|---|---|---|
| `synthetic-data/round1/` | `onboarding-handbook.md`, `team-directory.md`, `access-policy.md` | January 2026 — old process |
| `synthetic-data/round2/` | `policy-update-2026-05-15.md`, `team-directory-update.md` | May 2026 — contradicts round1 |

**The contradiction that makes the demo work:** Round1 says prod access requires 2-week tenure + SRE Slack ticket + pairing session (takes days). Round2 says that policy was replaced — now it's 3-day tenure + self-paced online course + IAM portal, just-in-time. Leadership also reshuffled (Anika is no longer Head of Engineering, Yuki is).

---

## Setup

```bash
# 1. Activate the virtual environment (already created)
source .venv/bin/activate

# 2. Install dependencies
pip install -r requirements.txt

# 3. Add your API key to .env (file already exists, fill in your key)
#    .env is git-ignored — never commit it
echo 'ANTHROPIC_API_KEY=sk-ant-...' > .env

# 4. Export the key — the scripts read from the environment, not .env directly
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

The agent reads the round1 docs, answers the test question, and saves key facts to `/mnt/memory/`. Output is saved to `outputs/session1.txt`.

### Step 3 — (Optional) Inspect memory

```bash
python inspect_memory.py
```

Lists everything the agent chose to remember. Good for the live demo — shows the agent's "working knowledge" before session 2 loads the contradictions.

### Step 4 — Run Session 2

```bash
python run_session_2.py
```

Fresh session, same agent and memory store, same question. The agent reads memory first, then the round2 docs, notices the contradictions, updates memory, and gives a better answer. Output → `outputs/session2.txt`.

### Step 5 — The demo moment

```bash
diff outputs/session1.txt outputs/session2.txt
```

Or open both files side by side. The session 2 answer should:
- Cite the new access policy (not the old Slack-ticket workflow)
- Catch the org changes (Yuki is now Head of Engineering, not Anika)
- Explicitly note what changed from session 1

---

## Scenario cards

The code runs **Card A — New-Hire Onboarding** out of the box. To switch to another card (B, C, or D) you would need to:

1. Replace the `TEST_QUESTION` string in both session files
2. Create new `synthetic-data/round1/` and `round2/` docs for your scenario

The doc-loading logic is already dynamic (reads all `*.md` files from the directory) so no structural code changes are needed.

---

## Stretch goals

See `stretch-goals.md` for 9 extension ideas in 4 tiers — from tightening the memory policy in the system prompt (Tier 1) to per-tenant memory isolation and autonomous learning via scheduled sessions (Tier 3–4).

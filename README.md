# Hacktoberfest_vision
# CommitWise

**AI that writes your commit story and shows the proof.**

CommitWise helps you explain a code change before you commit it. It collects real
evidence from git and your code, then uses Gemma 4 to turn that evidence into a
commit message, an impact summary, docs fixes and a description of UI changes.
Every AI claim cites the evidence behind it, and a plain-code check flags any
claim the evidence does not support.

Built for the Finite Loop OSS Fest (Hacktoberfest Hack Day) at NMAMIT, for the
**Best Use of Gemma 4** challenge.

<!-- TODO: add a demo GIF or screenshot here -->
<!-- TODO: add the demo video link here -->

## The problem

After changing several files, a developer still has to write an accurate commit
message, work out what the change might break, find a sensible reviewer and fix any
docs that are now wrong. People rush this, so commit messages are vague, docs go
stale and reviewers get changes they cannot easily judge. AI commit-message tools
make it faster, but they can describe changes the code never made, and nobody can
tell which claims are true.

## What it does

1. **Collects evidence (no AI):** changed files, added/removed/modified functions,
   who calls them, who last edited the changed lines (`git blame`), and which docs
   mention them.
2. **Writes the story (Gemma 4):** a Conventional Commits message, an impact
   summary with risk levels, and suggested doc edits. Gemma only sees the evidence.
3. **Checks the claims (no AI):** any claim that cites no evidence, cites an unknown
   evidence ID, or names a file or function that is not in the evidence is flagged.
4. **Reads screenshots (Gemma 4, multimodal):** upload before and after screenshots
   and Gemma describes what visibly changed, then links it to the changed UI files.
5. **Says what it does not know:** gaps (for example dynamic calls it cannot
   resolve) are listed instead of guessed.

## Role of AI

Gemma 4 does the parts that need language and vision:

- turns structured evidence into a commit message, impact explanation and doc edits
- reads before/after screenshots and describes the visible UI changes
- reports what it is unsure about

The project does not work without it: without a model there is no explanation or
screenshot analysis, only raw git facts. The AI never invents facts, because it is
only given the evidence and its output is verified by code in `commitwise/verify.py`,
not by another model.

## Model

- **Gemma 4**, accessed through the Gemini API
- Model ID used: `<MODEL_ID>` <!-- TODO: exact model id -->
- License / terms: <!-- TODO: link to the Gemma 4 license or terms of use -->

Open-weight does not mean every part is open source, so check the linked terms for
the exact model you use.

## How it works

```
git repo ──> evidence.py ──> evidence JSON (E1, E2, ...)
                                   │
                        gemma.py + prompts.py ──> Gemma 4 ──> claims + evidence ids
                                   │                              │
                                   └─────────> verify.py <────────┘
                                                   │
                                                   v
                                             app.py (UI)
```

Example evidence item and the claim that cites it:

```json
{"id": "E4", "symbol": "verify_token", "called_by": ["api/routes.py:5"]}
```
```json
{"claim": "verify_token() is called from api/routes.py", "evidence_ids": ["E4"], "risk": "medium"}
```

## Project structure

```
app.py                  Streamlit UI
commitwise/
  evidence.py           git diff, AST symbols, blame, callers, doc search
  prompts.py            prompts for commit/impact and screenshots
  gemma.py              Gemini API calls (text and images), JSON parsing
  verify.py             honesty check against the evidence
scripts/
  make_demo_repo.sh     builds a demo repo with two authors and pending changes
```

## Key dependencies

| Dependency | Use | License |
|---|---|---|
| Streamlit | web UI | Apache-2.0 |
| google-genai | Gemini API client | Apache-2.0 |
| git | diff and blame | GPL-2.0 (system tool) |

<!-- TODO: confirm each license before submitting -->

## How to run

Requirements: Python 3.10+, git, a Gemini API key.

```bash
git clone <your-repo-url>
cd commitwise
pip install -r requirements.txt

export GEMINI_API_KEY=your-key
export GEMMA_MODEL=<MODEL_ID>

bash scripts/make_demo_repo.sh demo_repo
streamlit run app.py
```

In the app, enter `demo_repo` as the repository path and press **Analyze my
changes**. Optionally upload before and after screenshots and list the changed UI
files.

To see the evidence engine on its own (no AI):

```bash
python -m commitwise.evidence demo_repo
```

## Demo walkthrough

1. Open a repo with uncommitted changes (the demo repo works).
2. Press **Analyze my changes** and read the commit message, impact and reviewers.
3. Tick **Demo: inject a false claim** and run again to watch the honesty check
   flag it.
4. Upload before and after screenshots to get the visual change description.

## How it differs from existing tools

Tools already exist for each piece: commit-message generators, reviewer suggesters,
documentation-drift detectors and PR intent checkers. CommitWise combines them in one
author-side workflow, runs before the commit instead of after the pull request,
shows the evidence behind every claim, and links screenshot changes to the code
files that likely caused them.

## Limitations

- Symbol analysis and caller lookup cover Python only, and are name-based
- The honesty check catches unsupported files, functions and citations, not every
  kind of wrong claim
- Reviewer suggestions come from blame history only, not from review activity
- Screenshot analysis depends on image quality and can miss subtle changes

## Roadmap

- More languages (JavaScript and TypeScript) for symbols and callers
- Automatic before/after screenshot capture
- A pre-commit hook and a GitHub Action
- A benchmark of reviewer suggestions against real past reviewers

## Team

<!-- TODO: names and GitHub handles of all team members -->

## License

MIT, see [LICENSE](LICENSE).

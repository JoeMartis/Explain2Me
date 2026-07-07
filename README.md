# Explain2Me — Explanation Quality Checker

A small, static web app that scores the **quality of problem explanations** — the worked rationales that accompany quiz/problem items. It's built for module teams to run against their own explanations before publishing.

The explanation-quality logic is adapted from **[Fixatron2000](https://github.com/JoeMartis/Fixatron2000)**, distilled down to the single concern of "is this explanation any good?"

## What it checks

Every explanation is scored two ways:

1. **Offline heuristics** (no key, runs in the browser):
   - **Present** — there is a real explanation
   - **Substantive length** — more than one throwaway sentence (configurable word target)
   - **Shows reasoning** — uses explanatory language ("because", "therefore", …) rather than bare assertion
   - **Not a restatement** — adds something beyond echoing the question/answer
   - **Not cut off** — doesn't end truncated mid-sentence
   - **Clean formatting** — no Word/AI copy-paste artifacts
   - LaTeX/MathJax is treated as normal, not an error.

2. **AI review** (optional, bring-your-own Anthropic API key): mirrors Fixatron's "explanation quality" check — Claude judges whether the explanation explains *why* the answer is correct, gives reasoning, and is substantive, returning **Sufficient / Insufficient** plus a one-line reason.

Both the heuristic rubric (weights + thresholds) and the AI grading prompt are **editable in the UI** and saved to your browser.

## Usage

- **Single explanation** — paste a question + explanation, get a score, a per-criterion checklist, and (if enabled) the AI verdict.
- **Batch / upload** — paste or upload a **CSV / TSV / JSON** file of many explanations; get a sortable results table, summary stats, and CSV/JSON export.
  - Recognised columns: `id`, `question`, `explanation` (only `explanation` is required). Use the in-app **Download CSV template** button for the exact shape.

### AI review setup

Open **⚙ Settings**, paste an Anthropic API key, pick a model (Haiku 4.5 is the fast/cheap default, matching the Fixatron), and tick **Enable AI**. The key is stored **only in your browser's localStorage** and is sent **directly to `api.anthropic.com`** — it never touches any intermediary server. Use **Forget stored key** to clear it.

> Note: browser calls use Anthropic's `anthropic-dangerous-direct-browser-access` header. This is fine for a trusted internal tool where each teammate uses their own key, but the key is exposed to anything running in that browser tab — don't embed a shared org key in the page.

## Hosting on GitHub Pages

This is a fully static site (no build step). To publish:

1. Push this branch to GitHub (already done if you're reading this in the repo).
2. In the repo, go to **Settings → Pages**.
3. Under **Build and deployment → Source**, choose **Deploy from a branch**.
4. Select this branch (`claude/explanation-quality-webapp-yx5ww6`) — or merge it into `main` first — and folder **/ (root)**. Save.
5. After a minute the app is live at `https://<owner>.github.io/<repo>/`. Share that link with your module teams.

No server, database, or secrets are required.

## Files

```
index.html          # markup
assets/styles.css    # theme-aware styling (light/dark)
assets/app.js        # heuristics, AI review, parsing, rendering
```

## Development

Just open `index.html` in a browser, or serve the folder:

```sh
python3 -m http.server 8000    # then visit http://localhost:8000
```

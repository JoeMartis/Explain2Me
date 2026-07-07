# Explainiac — Explanation Writing Trainer

A small, static web app that **trains module authors to write problem explanations that teach** — the worked rationales that accompany quiz/problem items. Authors practice on their real items: paste a draft, get instant rubric feedback and an AI verdict, revise, re-check, and compare against an example rewrite. Batch mode doubles as a QA sweep over a whole item set before publishing.

> Hosted from the `JoeMartis/Explain2Me` repository; the app itself is branded **Explainiac**.

The explanation-quality logic is adapted from **[Fixatron2000](https://github.com/JoeMartis/Fixatron2000)**, distilled down to the single concern of "is this explanation any good?"

## What it checks

Every explanation is scored two ways:

1. **Offline heuristics** (no key, runs in the browser):
   - **Substantive length** — an explanation should be **more than one sentence**, and meet a configurable word target
   - **Shows reasoning** — uses explanatory language ("because", "therefore", …) rather than bare assertion
   - **Not a restatement** — adds something beyond echoing the question/answer
   - **Addresses wrong options** — for multiple-choice items, generally explains why the incorrect answers are incorrect
   - **No positional labels** — never "Option 1" / "the first option" / "choice B"; options may be shuffled when displayed, so options must be referred to by content
   - **Not cut off** — doesn't end truncated mid-sentence
   - LaTeX/MathJax is treated as normal, not an error.

2. **AI review** (optional, bring-your-own Anthropic API key): the semantic judgment the offline checks can't make — Claude evaluates reasoning quality, conceptual grounding, **factual soundness** (when a keyed correct answer is provided it verifies the explanation supports *that* answer; otherwise it works the problem itself), and distractor coverage, returning **Sufficient / Insufficient** plus a one-line reason naming the failed criterion. Mechanical issues (length, positional labels, truncation) are left to the offline checks so the two layers don't double-report. It renders as its own section beneath the heuristic score.
   - **✨ Make it better** — a one-click button in the AI review section asks the model to write a strong replacement explanation for that question (explains the mechanism, addresses the wrong options, avoids positional labels). **Use & re-check** drops the suggestion into the input and re-grades it; **Copy** copies it to the clipboard.

Both the heuristic rubric (weights + thresholds) and the AI prompts (grading + "make it better") are **editable in the admin panel** and saved to your browser.

## Usage

- **Single explanation** — paste the **full question stem and all answer options** (required), the **correct answer** (recommended), plus the **explanation** (required); get a score, a per-criterion checklist, and (if enabled) the AI verdict. The options matter: without them the checker can't tell whether the explanation covers the *wrong* answers or leans on positional labels. The correct answer matters too: with it, the AI verifies the explanation supports the *keyed* answer instead of solving the problem itself.
- **Batch / upload** — paste or upload a **CSV / TSV / JSON** file of many explanations; get a sortable results table, summary stats, and CSV/JSON export.
  - Recognised columns: `id`, `question`, `options`, `answer`, `explanation`. Include `question` and `options` so the wrong-answer and positional-label checks are meaningful (rows missing them are still scored, with a heads-up), and `answer` (the keyed correct answer, by content) so the AI verifies against it. In JSON, `options` may be an array of choices and `answer`/`correct`/`correct_answer` are all recognised. Use the in-app **Download CSV template** button for the exact shape.

### AI review setup

Paste an Anthropic API key into the **🔑 API key** box on the main page. The key is stored **only in your browser's localStorage** and is sent **directly to `api.anthropic.com`** — it never touches any intermediary server. Use **Forget** to clear it. Without a key, the offline heuristics still run.

### Admin settings

The tuning knobs — model choice, AI on/off, rubric weights and word threshold, and the AI grading prompt — live in an admin panel that is hidden from everyday users. Open the app with **`?admin=1`** appended to the URL (e.g. `https://<owner>.github.io/<repo>/?admin=1`) and a **⚙ Admin settings** button appears in the header.

> This hides complexity; it is **not** access control. The app is fully client-side, so anyone who knows the parameter can open the panel — and every setting is stored per-browser, so an admin's changes only affect their own browser.

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

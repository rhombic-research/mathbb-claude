# Research Arenas

A **Research Arena** is a public page for an open problem whose progress can be
measured. Its owner states the problem and names the quantities that track
progress — the width of a zero-free region, an exponent, a constant known to lie
between two bounds — and registered MathBB users submit **progress reports**: a
summary, a PDF writeup, attachments and the values they claim. The arena charts
the record over time, ranks a leaderboard and lists every report. Its public page
is `/arenas/<slug>`; the owner runs it in the app at `/app/arena/<id>`.

## The tools

- `list_arenas` — the user's arenas and those they have reported to
  (`include_public` adds the rest of the directory). `get_arena` — one arena as
  JSON, with everything an edit needs. Read it before changing anything.
- `create_arena` — a new arena, always a **draft**: only the owner sees it.
- `update_arena` — title, address (`slug`), tagline, problem statement
  (`description`), rules, `status`, `review_mode`, `listed`, `require_writeup`.
  Send only what changes; `description` and `rules` are replaced whole.
- `set_arena_parameters` — the full parameter list (below).
- `upload_arena_image` — a one-time link and a `curl` command for an image from
  the user's machine (below).
- `list_arena_reports`, `get_arena_report` — read the reports (an owner sees all
  of them, others the listed ones and their own, pending ones included).
- `upload_arena_report_file`, `submit_arena_report`, `update_arena_report` —
  send a progress report as the user, and change it while it is pending
  ([Submitting a report](#submitting-a-report)).
- `set_arena_report_date` — MathBB administrators only, and refused for
  everyone else: dates a report by when its result first appeared, so a
  historical result recorded later sits at its own date on the chart. Don't
  offer it to other users.
- **Browser only**: deleting an arena (the **⋯** menu of its console in the
  app, or Settings → Delete arena), reviewing reports (accept, verify, dispute,
  reject: the console's Reports section), and withdrawing or deleting a report
  (its page). Tell the user rather than working around it.

## The fields

- `title`, `tagline` (one line), `slug` — the address, from the title by
  default (lowercase letters, digits, hyphens). Changing it later keeps the old
  address redirecting.
- `description` — the **problem statement**, in MathBB's markdown + LaTeX (the
  page syntax of the core guide; `$…$`, `$$…$$`, theorem blocks). State the
  problem precisely, what is known, and why the parameters measure progress.
- `rules` — how to submit and what counts: what a report must contain, how
  values are checked, how ties and rounding are treated.
- `review_mode` — `"review"` (default): a new report waits for the owner's
  review; `"open"`: it is listed at once and the owner can still dispute or
  reject it. `listed` (default true): shown in the public `/arenas` directory
  once open. `require_writeup` (default true): each report must attach a PDF.
- `status` — `draft` (only the owner sees it), `open` (accepting reports),
  `closed` (read-only). Opening needs at least one parameter. Open an arena only
  when the user asks: it becomes public.

## Parameters

Each parameter is one tracked number. Fields: `name` (plain text), `symbol`
(LaTeX for the quantity alone, without `$…$`, e.g. `\Lambda` — no `\le` or
`=`: the arena writes the relation), `unit`, `description` (markdown), `direction`,
`baseline_value` / `baseline_label` (the best known result before the arena and
its source), `target_value` / `target_label` (the conjectured value), `scale`
(`linear`, or `log` for values spanning orders of magnitude; then every value
must be positive), `decimals` (fixed display digits, 0–12, or null for
automatic), `is_primary` (exactly one: it ranks the leaderboard and is charted
first; default the first), and `key` (a short stable identifier, derived from
the name when blank). Up to 20; the list order is the display order.

`direction` says what counts as progress:

- `"max"` — higher is better (a zero-free region's width, a lower bound).
- `"min"` — lower is better (an exponent, an error term, an upper bound).
- `"both"` — **two-sided**: a constant known only to lie between two bounds
  (0 ≤ Λ ≤ 0.2 for the de Bruijn–Newman constant, 43 ≤ R(5,5) ≤ 46). Reports
  claim a new lower bound, a new upper bound or both, and each bound keeps its
  own records. `baseline_value` is then the known **lower** bound,
  `upper_baseline_value` / `upper_baseline_label` the known **upper** bound, and
  the optional target must lie between them. Prefer one two-sided parameter to
  two one-sided ones for the same constant: the arena then shows the interval,
  its gap and when the bounds cross.

`set_arena_parameters` replaces the whole list. Keep an existing parameter by
sending its `id` (from `get_arena`) and the fields you change: the fields you
leave out keep their values, and `null` (or `""`) clears one. A parameter left
out of the list is deleted; one without an `id` is new and needs a `name`.

**The lock.** Once any report has been submitted (`parameters_locked` in
`get_arena`), existing parameters can't be removed and can't change their
`direction` or `key` — that would rewrite what reports claimed. Their name,
symbol, unit, description, known values, target, scale, decimals, primary flag
and order stay editable, and new parameters can be added. So settle the
directions before the arena opens.

## A worked example

```json
create_arena({
  "title": "The de Bruijn–Newman constant",
  "tagline": "Pin down Λ between its known bounds",
  "description": "The **de Bruijn–Newman constant** $\\Lambda$ is the real number such that $H_t$ has only real zeros exactly when $t \\ge \\Lambda$. The Riemann hypothesis is equivalent to $\\Lambda \\le 0$, and Rodgers and Tao proved $\\Lambda \\ge 0$.\n\n![The heat flow of the zeros](heat_flow.png)",
  "rules": "Each report proves a new bound on $\\Lambda$ and attaches the proof as a PDF. Round a lower bound down and an upper bound up.",
  "review_mode": "review",
  "parameters": [{
    "name": "de Bruijn–Newman constant",
    "symbol": "\\Lambda",
    "direction": "both",
    "baseline_value": 0, "baseline_label": "Rodgers–Tao (2020)",
    "upper_baseline_value": 0.2, "upper_baseline_label": "Platt–Trudgian (2021)",
    "target_value": 0, "target_label": "The Riemann hypothesis",
    "decimals": 3
  }]
})
```

The result is the new arena as JSON, with its id, its parameters' ids and its
app and public URLs.

## Images

The problem statement and rules can show images uploaded to the arena:

1. `upload_arena_image` with the arena's id returns a `curl` command; run it
   with the local file (png, jpg, gif or webp, up to 10 MB). The link works once
   and expires in 15 minutes.
2. The reply names the stored file — the name made safe and unique, which may
   differ from the local one (`heat flow.png` becomes `heat_flow.png`).
3. Reference it by that name: `![The heat flow of the zeros](heat_flow.png)`,
   then `update_arena` with the new `description` or `rules`.

`get_arena` lists the arena's images with the markdown for each. Removing one
is done in the app (the arena's Settings → Images).

## Submitting a report

Anyone registered can report to an open arena, the owner included. Submit only
when the user asks, with their result: the report goes out under their name,
and the owner is emailed.

**What a good report contains.**

- `title`: the result in plain text, e.g. "Λ ≤ 0.19 by a refined barrier
  argument".
- `values`: the claimed values, keyed by parameter id (from `get_arena`). Give
  the exact form as well as the number whenever the result has one:
  `{"value": 0.19, "display": "\\tfrac{19}{100}"}`. A two-sided parameter takes
  `{"lower": …, "upper": …}`, either side optional; a one-sided one a number or
  `{"value", "display"}`. Claim only what the writeup proves, rounded the safe
  way (a lower bound down, an upper bound up), and check `standings` in
  `get_arena` to see whether it improves on the record.
- `summary`: markdown + LaTeX (the page syntax of the core guide). The result
  stated precisely, then the method in a few paragraphs, and what it builds on.
  It is what readers see first.
- The **writeup**: a PDF with the proof. Most arenas require one
  (`require_writeup` in `get_arena`).
- **Attachments**: what lets others check the work. Data (csv, json), code
  (py, sage, ipynb), Lean files, archives of them, plots. Up to 10, 25 MB each.
- Optionally `authors` (default: the user's name), `builds_on` (the id of a
  report in the same arena this one improves on), `external_url` (arXiv, a
  repository) and `tools_disclosure` ("Lean 4 + Mathlib; Claude Code").
  Follow the arena's `rules`.

**Two steps.**

1. For each file, `upload_arena_report_file` with the arena's id and `kind`
   (`"writeup"` for the PDF, `"attachment"` for the rest) returns a `curl`
   command: run it with the local path. The reply is JSON with a `file_token`
   and the filename the report will show. A link takes one file and expires in
   15 minutes; the uploaded file waits an hour.
2. `submit_arena_report` with the fields, `writeup_file` (a token) and
   `attachment_files` (a list of tokens). Everything is checked first; if the
   report is refused, fix what the message says and submit again with the same
   tokens. Once the submit succeeds, the tokens are spent. If you lost a
   submit's reply, check `list_arena_reports` before submitting again.

**Then.** In an arena under review (`review_mode` `"review"`, the default) the
report is **pending**: only the user and the owner see it, until the owner
accepts it. While it is pending, `update_arena_report` changes any field
(`values`, when given, replace all of them), replaces the writeup
(`replace_writeup_file`), adds attachments (`add_attachment_files`, tokens
from new uploads) and removes them (`remove_attachment_names`); these names
differ from `submit_arena_report`'s. Once the owner
has decided, it can't be changed. In an arena with `review_mode` `"open"`
the report is listed at once and can't be changed. Each user can submit at most
10 reports to an arena a day. Open the report's URL for the user to check.

## Showing the user

Tools return the arena's app URL (`/app/arena/<id>`, where the owner edits it)
and its public URL (`/arenas/<slug>`). After creating or substantially changing
an arena, open the app URL so the user can check it, and open it for reports
only when they say so.

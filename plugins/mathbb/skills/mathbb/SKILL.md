---
name: mathbb
description: How to work in MathBB notebooks through the MathBB tools — the page markdown dialect (math, theorem blocks, labels and cross-references, collapsible content, page builds, galleries, columns, Lean), figures, 3D models and widgets, and how to show the user a result. Use whenever you read, write or organize MathBB notebooks.
---

# Working in MathBB

MathBB is a web app for writing mathematics: notebooks of pages written in
markdown with LaTeX math, plus resources (figures, 3D models, widgets, text and
Lean files). The `mathbb` MCP server gives you its tools; you are signed in as
the user, with exactly their permissions.

## How to work

- **Find the notebook first.** `list_notebooks` lists everything the user can
  open (their own folder tree, then notebooks shared with them). `get_notebook`
  shows one notebook's pages and resources — call it before working in a
  notebook, and again if someone else may have changed it. Every notebook tool
  takes a `notebook_id`.
- **Name things, never number them.** Tool results carry ids (`[1234]`,
  `[folder id=7]`) because the tools need them — but users never see ids
  anywhere in MathBB. When you talk to the user, call notebooks, pages, folders
  and resources by their titles and filenames ("the page *Norms*"), never by id,
  and never ask the user for one.
- **Read before you edit, and edit small.** `read_page`, then `edit_page` with
  `old_string`/`new_string` for the part that changes. Rewrite a whole page only
  when it is short. The user may have the page open in the browser: your edits
  appear there live, so targeted edits are also kinder to what they see.
- **Show the user.** Tools report browser URLs. When you have made something the
  user will want to look at — a new notebook, a page you wrote, a figure — open
  it: `open <url>` on macOS, `xdg-open <url>` on Linux, `start <url>` on Windows.
  Don't open a tab for every small edit.
- **Files between the user's machine and MathBB**: `download_resource` and
  `upload_resource` return a short-lived link and a `curl` command — run it
  yourself (with the local path, for an upload). Use them when the user asks to
  download or add a file; to just read a text file, `read_resource_text` is
  enough.
- **What you can't do here**: delete a notebook, share one (public links,
  collaborators, course shares) or change the account. Those are done in the
  browser — tell the user rather than working around it. On a notebook the user
  can only view, tools that change it are refused. GitHub backup tools work only
  on notebooks the user owns.
- Claude Code asks before each MathBB tool call; the user can allow them for good
  with `/permissions`.

## Guides in this skill — read the one you need first

- [widgets.md](widgets.md) — before writing or changing a widget (`create_widget`).
- [computing.md](computing.md) — before running code (`execute_code`).
- [files-and-folders.md](files-and-folders.md) — text files and organizing
  pages and resources into folders.
- [github.md](github.md) — before using the GitHub backup tools.

## Page syntax

- Use $...$ for inline math and $$...$$ for display math. Do not emit
  \(...\) or \[...\] yourself — the renderer accepts them (e.g. in content
  pasted from elsewhere), but $-delimiters are the house style.
- For a NUMBERED display equation, write a bare equation environment
  (no $$ wrapper needed):
  \begin{equation}
  f(x) = x^2 \label{eq:square}
  \end{equation}
  Equation numbers are scoped per page ("(2.3)" = page 2's third numbered
  equation) and render live. Reference a labeled equation with
  \eqref{eq:square} from anywhere in the SAME NOTEBOOK — labels are
  notebook-wide, so referencing across pages works and is normal. Only add
  \label when the equation is actually referenced. Use plain $$...$$ for
  display math that doesn't need a number.
- For theorem-like statements, use a fenced div block:
  ::: {.theorem title="Euler's formula"}
  \label{thm:euler}
  For all real $x$, $e^{ix} = \cos x + i \sin x$.
  :::
  Supported types: theorem, lemma, proposition, corollary, definition,
  example, remark (all numbered from one shared counter, scoped per page —
  "Theorem 2.1, Lemma 2.2"), and proof (unnumbered, gets a QED box). The
  title="..." attribute is optional, as is the \label — add one only when
  the result is referenced, then cite it with \ref{thm:euler} → "2.1". These
  render as styled blocks live and become real amsthm environments in the
  LaTeX manuscript export. Do NOT use them in chat replies — only in pages.
- CROSS-REFERENCES: \label{key} is the ONE way to mark a link destination, and
  it attaches to whatever encloses it, exactly as in LaTeX — the equation, the
  theorem block, the heading it sits on, or failing all three, that spot in
  the text:
    ## Convergence \label{sec:conv}
    Some prose. \label{key-step} More prose.
  Labels are one namespace per NOTEBOOK, so a reference resolves from any page
  and never has to say which page its target is on. Link to one with:
    \ref{thm:euler}   the number — "2.1"
    \eqref{eq:square} the number in parentheses — "(2.3)"
    [text](#sec:conv) a text link to any label at all
    [](#sec:conv)     same, with the text filled in from the target
    [](page:Norms)    a page, by its CURRENT title (no label needed on it)
  Use markdown link text for prose ("as shown [in the last section](#sec:conv)")
  and \ref/\eqref inside a sentence that names the object ("by Theorem
  \ref{thm:euler}"). Never invent a label you did not define — an unresolved
  reference renders as a red (??). A page link must match a real page title
  exactly; check the notebook snapshot's page list before writing one.
- LEAN FORMALIZATION: a theorem-like div takes lean="file.lean", naming a .lean
  resource — ::: {.theorem lean="gauss_statement.lean"} — and the block
  then shows a "Lean" button that swaps the written text for the file. Put the
  STATEMENT file (body `sorry`, verdict `incomplete` — correct and unremarkable)
  on the theorem block and the PROOF file (verdict `proved`) on the proof block,
  and write both when both were asked for. Filename only, no path. Check every
  file with lean_check and report it in one word beside the name:
  `gauss_statement.lean` (elaborated), `gauss.lean` (validated). Say more only when
  a check failed or the formal statement really differs from what was asked — a
  hypothesis added or dropped, a weaker claim, a different object. Never call
  anything proved on any verdict but `proved`. Keep verdicts, status notes and
  `sorry` apologetics out of the file and off the page: at most one /-- … -/ line,
  for a modelling trap a reader could miss (ℕ division truncating, a coercion, a
  log base).
- COLLAPSIBLE CONTENT: `\collapsehere` in a list item or a theorem-like fenced
  div marks where a fold begins — the reader can then collapse everything after
  the marker, leaving the words before it visible, and expand it again with a
  "⋯" capsule. It changes nothing about the printed document: the LaTeX export,
  the ZIP download and every reader who hasn't clicked see the full text.
  * An example. \collapsehere The worked-through details, which fold away.
  ::: {.proof}
  Suppose not. \collapsehere Then the whole argument follows, at length.
  :::
  Use it where a reader scanning the page benefits from a summary line: long
  examples, routine or computational proofs, side remarks, case analyses. Put
  it after a short self-describing lead-in, so the collapsed state still says
  what is hidden ("Proof.", "Theorem 1.1 (Law of large numbers).", "An example
  of a non-measurable set."). Do NOT fold something short, do not fold the
  statement of a theorem the rest of the page depends on, and do not put a
  marker on every item — a page where everything is collapsed is worse than one
  where nothing is. It only works in list items and theorem-like divs (a marker
  in a header or an ordinary paragraph does nothing); the first marker in a
  container wins; and it is page syntax, never for chat replies.
- PAGE BUILDS (SLIDES): `\pausehere` splits a page into stages that are revealed
  one keypress at a time, the way Beamer's \pause does — in presentation mode,
  and in the reading view behind a public share link, where arrow keys step
  through the build before turning the page. Everything else shows the whole
  page at once (the editor preview, the LaTeX export, the ZIP download), so a
  build never withholds anything from a reader who can't step. Written as its
  own paragraph (a blank line either side) it gates
  everything after it; written at the head of a list item — `- \pausehere Second
  point` — it gates that item, bullet included. An optional argument picks the
  transition: [noanimation] (what a bare marker means today), [fade], [slide]
  (in from the right), [typewriter] (characters appearing in sequence). Space is
  reserved for what hasn't arrived, so nothing already on screen moves as the
  page builds.
  Use it only when the user is making something to PRESENT, and only where the
  reveal carries meaning: a claim before its proof, a list built point by point,
  an answer after the question has been asked, a punchline. Do NOT add builds to
  an ordinary reading page, and do not pause at every paragraph — a page that
  stops ten times is one nobody can talk through. Mid-paragraph, a marker only
  makes sense with [fade] or [noanimation]; [slide] there animates half a
  sentence in from the margin and looks broken. It is page syntax, never for
  chat replies.
- IMAGE GALLERIES: several images shown in the space of one, stepped with ‹ ›
  arrows and a row of dots. Same markdown as an ordinary image embed, with the
  filenames separated by `|`:
  ![alt_text;size=WIDTHxHEIGHT](first.png|second.png|third.png)
  The size is the size of ONE frame. Add `;auto=N` (seconds) to have it advance
  by itself: ![alt_text;size=600x400;auto=5](a.png|b.png|c.png). Only image
  files — never mix in a .pdf, .glb or .js, which are embeds of their own.
  Any image resource works as a frame, .svg included; note only that the LaTeX
  manuscript cannot embed .svg/.gif/.webp and prints a "not included in the
  PDF" note for such a frame, so prefer .png/.jpg frames for a notebook headed
  for a manuscript.
  Where there is nothing to click (the LaTeX manuscript, the PDF export) every
  frame prints as a grid, so a gallery never hides content from a reader.
  Use one when the images are variations on a single thing the reader compares
  or steps through — frames of an animation, stages of a construction, the same
  plot at several parameter values. Do NOT use one for images that belong to
  different parts of the discussion: those are separate figures and each needs
  its own place in the text.
- YOUTUBE VIDEOS: a video is embedded with the ordinary image syntax pointed at
  a YouTube URL instead of a filename:
  ![caption](https://youtu.be/VIDEO_ID)
  Any form of YouTube link works (watch?v=, youtu.be, /embed/, /shorts/), and a
  `?t=90` or `?t=1m30s` start time is kept. `;size=WIDTHxHEIGHT` sizes the
  player (default 560x315); give it a visible italicized caption line below, as
  you would any other embed. There is no tool for this and no file involved —
  only embed a link the user gave you or one you actually found on the web;
  never invent a video id. On paper (the LaTeX manuscript, the PDF export) the
  video prints as a placeholder naming it and its link, so a video must never
  be the only place some content lives.
- For multi-column page layout, use column fenced divs. Side-by-side blocks,
  where you write each column's content:
  ::: {.columns}
  ::: {.column width="38%"}
  ![Spiral flow](spiral.svg)
  :::
  ::: {.column}
  Text that sits beside the figure.
  :::
  :::
  width="..." is optional (a percentage like "38%" or a fraction like "0.38");
  columns without one share whatever is left, so two bare .column blocks split
  the row evenly. For a single stream of text flowed across balanced columns
  instead, use ::: {.multicol count="2"} ... ::: (count 2-6, default 2).
  Use columns where the layout earns it — a figure beside its discussion, two
  cases compared, a definition next to an example — not for ordinary prose.
  Keep each column's content short enough to read at half width, and avoid wide
  display equations inside a narrow column (they scroll rather than wrap).
  Columns collapse to a single column on narrow screens and in a narrow editor
  preview pane; that is expected, not a mistake to work around.
  Theorem blocks may be nested inside a .column. Two limits come from the LaTeX
  export: a .columns block cannot be nested inside another .columns block, and
  .multicol cannot be used inside a .column — both still render in the live
  preview but degrade to ordinary flowed content in the compiled PDF, so avoid
  them. Do NOT use column divs in chat replies — only in pages.
- Do NOT use \begin{align}, \begin{align*}, \begin{gather}, \begin{multline},
  or similar multi-line environments. For multi-line display math, use
  $$...$$ with \\ line breaks inside, and use \begin{aligned}...\end{aligned}
  within $$...$$ for alignment. (Multi-line displays cannot be numbered.)
- Math delimiters are ignored inside code fences and inline code spans, so
  show literal math syntax as code when explaining it.
- Use standard LaTeX commands supported by KaTeX.
- Use **bold** and *italic* for text formatting.

## Figures, 3D models, SVGs and widgets

- When asked to create a figure/plot/diagram, use generate_figure: you write the matplotlib script yourself, it runs in the code sandbox, and the rendered image comes back to you — look at it and fix what's wrong before moving on. The code is kept with the figure, so the user can read and reuse it; write it to be clear.
- To change an existing figure or 3D model, read its code with read_resource_text, then edit it with edit_resource_text (old_string/new_string on the figure's or model's filename) — that re-renders it in place, and every page embedding it updates. For a rewrite, call generate_figure / generate_3d_model with the same filename and replace=true. Never make a second copy.
- When asked to create a 3D model, use generate_3d_model: you write the Blender scene script yourself (no export, no render — the server does both), and a preview render comes back to you to check. Its code is kept with the model, like a figure's.
- When asked for an editable SVG, use insert_inline_svg.
- When asked for an SVG resource file, use create_svg_resource.
- If unsure whether to use a figure or SVG, prefer generate_figure.
- When asked for an interactive animation, simulation, demo, or game, use create_widget.

ACCESSIBILITY RULES (apply to every figure, SVG, 3D model, and widget you generate):
- Always provide a short alt_text (no more than 20 words) that concisely describes the asset for screen-reader users. This goes into the markdown image's alt attribute.
- Always provide a plain-text caption that names or describes the asset for readers.
- When you insert an asset into a page (via create_page / edit_page following generate_figure/generate_3d_model/create_svg_resource/create_widget, or via insert_inline_svg), the page markdown MUST include the caption as visible text alongside the asset — typically as an italicized line immediately below the image, e.g.:
  ![alt_text](filename.pdf)
  *caption goes here*
  For insert_inline_svg, use ![alt_text;svg-inline](<svg>...</svg>) and put the caption as a visible italicized line beside/below it in new_content.
- When an asset is generated but not inserted into a page, still supply a caption; do not embed it inside the asset file itself.

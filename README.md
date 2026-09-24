# MathBB for Claude Code

Work on your [MathBB](https://mathbb.app) notebooks from Claude Code, on your
own Claude plan: write and edit math-rich pages, render figures and 3D models,
build interactive widgets, run code and check Lean proofs in MathBB's
sandboxes, and organize your notebooks.

## Install

In Claude Code:

```
/plugin marketplace add rhombic-research/mathbb-claude
/plugin install mathbb@rhombic-research
```

Then run `/mcp`, choose **mathbb**, and sign in: your browser opens MathBB and
asks you to allow the connection. You need a MathBB account (guest accounts
can't connect).

## Use

Ask for what you want, e.g. *"In my Sendov notebook, add a page proving the
n = 3 case, with a figure of the roots and critical points"*. Claude finds the
notebook, writes the page, renders the figure, and can open the result in your
browser.

Claude Code asks before each MathBB tool call; allow them for good with
`/permissions`.

## Your data and the connection

The connection has exactly your MathBB permissions — a notebook you can only
view stays view-only. It can't delete notebooks, share them, or change your
account. It lasts until it goes unused for 30 days; revoke it any time in
MathBB under **Settings → Security → Connected apps**.

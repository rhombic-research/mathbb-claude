# MathBB for Claude Code

Connect [MathBB](https://mathbb.app) to Claude Code and work on your notebooks,
blog and Research Arenas from there, on your own Claude plan. Claude can write
and edit math-rich pages, render figures and 3D models, build interactive
widgets, run code and check Lean proofs in MathBB's sandboxes, organize your
notebooks, write and publish blog posts, create and edit Research Arenas, and
submit progress reports.

## Connect

In a terminal:

```
claude mcp add --transport http mathbb https://mathbb.app/mcp
```

Then run `/mcp` in Claude Code, choose **mathbb**, and sign in. Your browser
opens MathBB and asks you to allow the connection. You need a MathBB account
(guest accounts can't connect).

Step by step, and other MCP clients: <https://mathbb.app/connect>

## The plugin (optional)

The plugin sets up the same connection and adds the MathBB authoring skill,
which Claude Code loads when a task needs it. Without the plugin, Claude gets
the same guidance from the server's `read_mathbb_guide` tool.

In Claude Code:

```
/plugin marketplace add rhombic-research/mathbb-claude
/plugin install mathbb@rhombic-research
```

Then sign in with `/mcp` as above. Use the plugin or the command, not both.

## Use

Ask for what you want, e.g. *"In my Sendov notebook, add a page proving the
n = 3 case, with a figure of the roots and critical points"*. Claude finds the
notebook, writes the page, renders the figure, and can open the result in your
browser.

Or set up a Research Arena: *"Create an arena for the de Bruijn–Newman
constant, tracking its known lower and upper bounds, with the problem statement
from my notebook's first page"*. Claude creates it as a private draft and opens
it for you to check. You open it for reports when you're ready.

Or report progress: *"Submit my new upper bound to the de Bruijn–Newman arena,
with paper.pdf from this folder as the writeup"*.

It can also move files both ways: *"download the figures from my Sendov notebook
into this folder"*, or *"add ~/Desktop/diagram.png to the notebook and put it on
the second page"*.

Claude Code asks before each MathBB tool call. Allow them for good with
`/permissions`.

## Your data and the connection

The connection has exactly your MathBB permissions. A notebook you can only
view stays view-only. It can't delete notebooks, arenas or your blog, share
notebooks, moderate comments, review, withdraw or delete arena reports, or
change your account. It lasts
until it goes unused for 30 days. Revoke it any time in MathBB under
**Settings → Security → Connected apps**.

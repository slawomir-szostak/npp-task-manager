# npp-task-manager

A Notepad++ User Defined Language for managing tasks in `.tsk` files.

This is not a new programming language — it is a set of highlighting rules that turn
a plain text file into a readable task list that rolls over day by day.

## The `.tsk` format

```
== 2026-09-21
!!007 renew the signing certificate // expires on friday
~~004 write the import adapter // created ISSUE-1232
005 send a follow-up e-mail about ISSUE-1232
++003 tag release 1.4.0
--002 drop the legacy exporter

== 2026-09-19
...
```

The file reads top down: the most recent day sits at the very top.

### Day header

```
== 2026-09-21
```

The `==` prefix is **mandatory**. `==` alone is enough, and the date must be in
`YYYY-MM-DD` format. The header consumes the whole line, so the hyphens in the date
never collide with the `--` marker.

### Task line

```
[marker]NNN task description // optional remark
```

- `NNN` — task number, continuous, incremented as new tasks are added
- the marker is optional; its absence means an open task
- the marker sits in column 1, directly before the number, with no space
- `//` starts a remark that runs to the end of the line

### Markers

| marker | meaning | colour |
|---|---|---|
| *(none)* | open task | default, number in bold grey |
| `!!` | important / urgent | red, **bold** |
| `~~` | in progress | amber, **bold** |
| `??` | waiting for — blocked on someone else | blue |
| `##` | on hold — paused by you | slate, *italic* |
| `**` | new idea / maybe | purple, *italic* |
| `>>` | migrated — drops out of the daily rotation into the backlog | brown, *italic* |
| `++` | done | muted green, *italic* |
| `--` | won't do / cancelled | light grey, *italic* |
| `// …` | remark | green, *italic* |

The colour rule: **lightness equals currency**. Anything that needs action is dark and
high contrast. Anything closed (`++`, `--`) fades into the background. Only `!!` and `~~`
are bold — "this is on fire" and "this is where I am right now".

`??` versus `##`: with `??` somebody else is the blocker and needs a nudge; with `##`
you paused the task yourself and it needs nothing.

### Why the markers are doubled

Notepad++ UDL cannot anchor a rule to the start of a line — whole-line highlighting
triggers on the symbol **anywhere** in the text. A single `-` would fire the "won't do"
style inside words like `e-mail` or `follow-up`, a `?` inside a question, a `>` inside
an arrow such as `->`.

Doubling removes those collisions: `e-mail` has one hyphen, not two. It costs one extra
keystroke, and only when marking a task.

Note that inside an already marked line and inside a `//` remark the symbols are inert,
so `--002 send an e-mail` and `// ISSUE-1232` are completely safe. The risk applies only
to the description of an **unmarked** task.

### Rolling the day over

To start a new day, copy the whole previous block to the top of the file, change the date
in the header and delete the `++`, `--` and `>>` lines. Open and paused tasks carry over
with their numbers unchanged.

## Installation

There are two colour variants. **Install only one** — both register the `.tsk` extension
and will fight over it if installed together.

- `udl/tsk-light.udl.xml` — for the light Notepad++ theme
- `udl/tsk-dark.udl.xml` — for the dark one

### Option 1 — drop in the file (recommended, N++ 7.6+)

Copy the chosen file into:

```
%APPDATA%\Notepad++\userDefineLangs\
```

and restart Notepad++.

### Option 2 — import from the menu

`Language` → `User Defined Language` → `Define your language…` → `Import…`,
pick the XML file, restart Notepad++.

### Verifying

Open `examples/tasks.tsk`. If the highlighting did not kick in by itself, select it
manually: `Language` → `TSK Tasks (Light)` / `TSK Tasks (Dark)`.

Check three things:

1. The `== 2026-09-21` header has a background band and is bold.
2. The line `001 review the read-me` is entirely in the default colour — no hyphen has
   fired grey highlighting through to the end of the line.
3. In `~~004 … // created ISSUE-1232` the remark is green, not amber like the rest of
   the line.

If point 3 does not hold, it is a nesting setting: `Define your language…` →
`Operators & Delimiters` tab → for each of Delimiters 1–8 tick **Comment** in the
nesting section. In the XML files this corresponds to the `nesting="256"` attribute.
Without it the remark takes the colour of its line — still readable, just less pretty.

## Known limitations

- **No strikethrough.** Notepad++ styles support only bold, italic and underline, so
  `++` and `--` tasks are faded out by colour rather than struck through.
- **The dark variant forces a `#1E1E1E` background.** Under a substantially different
  theme the day header band may clash — override `bgColor` in the XML file.
- **Residual collisions.** Doubling still lets `C++`, `Notepad++`, `--save` and `-->`
  through if they land in the description of an **unmarked** task. The symptom is
  immediately visible (the tail of the line changes colour), so rewording fixes it.
- **No folding of day blocks.** UDL would need an explicit block-end marker.

## Configuring your own tags

The `Keywords1` list in the XML file is empty and meant for your own keywords — names,
project names, fixed labels. Enter them space separated:

```xml
<Keywords name="Keywords1">alice bob infra billing</Keywords>
```

To make them visible inside marked lines as well, add `1024` to the delimiter nesting
value (`nesting="1280"` instead of `256`).

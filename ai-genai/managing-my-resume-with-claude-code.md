# Resume as Code: How I Manage My Resume with Claude Code

I used to have multiple copies of my resume. One general version, one very detailed, one condensed, and a PDF and word doc whose origins I could no longer explain. Every time I updated a job description, I updated it in one file and skipped the others. None of them were current. All of them were "the latest version" at some point.

The fix turned out to be the same one we apply to every other document-drift problem in software: put it in a git repo, split it into single-purpose files, and generate everything else. Once I did that, something better happened. The repo became trivially easy for an AI agent to work on. Now when I want to retarget my resume for a role, I open Claude Code in the repo and describe the job. It edits a handful of small Markdown files, rebuilds, and I review the diff.

This post walks through the structure. It works with Claude Code, but nothing about it is Claude-specific. Any agent that can read files and run a shell script can maintain it.

## The Core Idea: Content as Partials, Layout as Data

Every section of the resume is a tiny Markdown file. A resume variant is nothing more than an ordered list of those files. The whole repo looks like this:

```text
resume/
├── build.sh              # assembles and renders every variant
├── CLAUDE.md             # agent instructions (style rules, build docs)
├── style.css             # print stylesheet for the PDF
├── src/
│   ├── partials/         # shared blocks: header.md, skills.md,
│   │                     #   role-acme.md, role-globex.md, role-initech.md,
│   │                     #   writing.md, education.md ...
│   ├── headline/         # per-variant title + tagline
│   ├── summary/          # per-variant professional summary
│   └── variants/         # recipes: <name>.txt = ordered list of partials
└── dist/                 # generated output, gitignored, one folder per variant
```

The `partials/` folder holds everything that should be written once: your contact header, your skills list, one file per job you've held. The `headline/` and `summary/` folders hold the two pieces that actually change per target: the title line under your name and the professional summary paragraph. Those are the sections where tailoring pays off, so they get one file per variant.

**Single source of truth:** when I update a bullet point in `role-acme.md`, every variant picks it up on the next build. There is no second copy to forget.

## Variants Are Just Text Files

A variant is a recipe: a plain-text file listing partials in the order they should appear. Here's what an `engineering-manager` variant might look like:

```text
# Engineering manager variant — leadership summary, writing elevated.
partials/header.md
headline/engineering-manager.md
summary/engineering-manager.md
partials/skills.md
partials/role-acme-head-manager.md   # same role, management framing
partials/role-acme-body.md
partials/role-globex.md
partials/role-initech.md
partials/writing.md
partials/education.md
```

That's the entire layout system. Reordering lines reorders sections. Deleting a line drops a section. Adding a variant means adding three small files: a headline, a summary, and a recipe. There is no templating engine, no YAML schema, no config format to learn.

The recipe mechanism enables a useful trick: splitting a role into a head and a body. The head file holds the job title line and framing sentence; the body holds the bullets. A role can have two head files, one framing it as hands-on engineering and one framing it as leadership. Each recipe picks the framing that fits its audience, and both share the same body, so the accomplishments never drift apart.

**Layout is data, not code.** That's the property that makes everything downstream easy, for both me and the agent.

## The Build: pandoc + Headless Chrome

`build.sh` is about sixty lines of bash. For each variant it does four things:

1. Reads the recipe and concatenates the listed partials into `dist/<variant>/Your_Name_Resume.md`, skipping blank lines and `#` comments, and failing loudly if a partial is missing.
2. Runs pandoc to turn that Markdown into a standalone HTML file with the stylesheet embedded.
3. Prints the HTML to PDF with headless Chrome.
4. Runs pandoc again to produce a DOCX for the recruiters who ask for one.

The three render commands, in full:

```bash
pandoc "$md" -f markdown --standalone --embed-resources --css style.css \
  --metadata pagetitle="John Doe Resume" -o "$out/$NAME.html"

"$CHROME" --headless --disable-gpu --no-pdf-header-footer \
  --print-to-pdf="$out/$NAME.pdf" "file://$(pwd)/$out/$NAME.html"

pandoc "$md" -f markdown -o "$out/$NAME.docx"
```

`$CHROME` points at the Chrome binary (on macOS, `/Applications/Google Chrome.app/Contents/MacOS/Google Chrome`; adjust for your platform). Running `./build.sh` builds every variant; `./build.sh engineering-manager` rebuilds just one.

Two things I like about this toolchain. First, there's no LaTeX installation. Pandoc and a browser you already have cover Markdown, HTML, PDF, and DOCX. Second, the intermediate files earn their keep: the assembled `.md` is exactly what I paste into web application forms that want plain text, and the HTML is what Chrome prints, so what I preview is what I ship.

The PDF layout lives entirely in `style.css`: `@page` margins for Letter paper, the name styled as an H1, uppercase section headers with a bottom rule, and `break-inside: avoid` on role blocks so a job never splits across a page boundary. All variants share the one stylesheet. Presentation and content order are fully separated.

The script also lints as it builds. It warns about orphans, source files that no recipe references (unused partials rot silently, and someday you edit one and wonder why nothing changed), and it prints each PDF's page count, flagging any over a threshold. The page check matters more than it looks: editing a shared partial affects every variant, and the failure mode is one of them quietly spilling onto an extra page.

## Where the AI Agent Comes In

Here's where the structure pays for itself. "Tailor my resume for this job posting" used to mean manual editing and a new orphaned file. Now it decomposes into exactly the operations an agent is good at:

- Write a new `headline/<variant>.md` that mirrors the posting's language.
- Write a new `summary/<variant>.md` that leads with the most relevant experience.
- Write a recipe that reorders sections to put that experience up top.
- Run `./build.sh <variant>` and confirm it renders.

I paste a job posting into Claude Code and ask for a variant targeting it. The agent reads the existing partials, writes the three new files, builds, and I review a diff that is maybe forty lines long. Because the shared partials didn't change, I know the other variants are untouched. Because everything is generated, nothing drifts.

Two more conventions close the loop on this workflow. The posting itself gets saved to `postings/<variant>.md`, preserving the input alongside the output; a future session can re-tune the variant with full context instead of guessing why the headline reads the way it does. And every resume that actually goes out gets a row in `applications.md`: date, employer, role, variant, and commit SHA, with significant applications tagged in git. Every variant renders to the same filename and `dist/` is gitignored, so without that log there's no record of which resume went to which employer. With it, when a recruiter calls back months later, `git checkout <commit> && ./build.sh <variant>` rebuilds the exact document they're holding.

Small files matter more than you'd think here. An agent editing a 400-line `resume.md` produces a diff you have to read suspiciously. An agent writing a 6-line summary file produces a diff you can review in ten seconds. The structure isn't just DRY for my benefit; it keeps every AI change small, isolated, and easy to reject.

## CLAUDE.md: Version-Controlled Preferences

The repo has a `CLAUDE.md` that every Claude Code session reads on startup. Half of it documents the build. The other half is style rules, and my favorite is this one:

```markdown
- **Do not use long (em) dashes (`—`) in resume content or any generated
  document.** Use commas, colons, parentheses, or separate sentences instead.
  Em dashes read as "AI-generated" and the user does not want them.
- En dashes (`–`) are acceptable **only** for numeric and date ranges
  (e.g. `2018 – Present`). Do not use them as sentence punctuation.
```

That rule exists because I caught the agent doing it, corrected it, and then codified the correction. That's the workflow: **every time you correct the agent, write the correction into CLAUDE.md so it sticks.** The file becomes a version-controlled record of your preferences, and every future session starts already knowing them.

The git history turns into something useful too. Commits like "Soften the age signal in the summary" or "Reframe consulting role for the portfolio variant" are a log of every positioning decision I've made about my own career. When I wonder why a bullet reads the way it does, the answer is one `git log` away.

## Why This Works So Well

- **Single source of truth.** Each role, skill list, and section exists once. Editing a partial updates every variant on the next build.
- **Variants are cheap.** Three small files and a build command. I stopped rationing tailored resumes because tailoring stopped being expensive.
- **Every change is a diff.** Whether I made the edit or the agent did, review is `git diff`, and rollback is `git revert`.
- **Multi-format from one source.** Markdown in; PDF, DOCX, HTML, and paste-ready text out. No format is hand-maintained.
- **The structure is legible to an agent.** The filesystem is the database and the recipe is the layout. There's no schema to explain, so instructions like "move Writing above the older roles in the portfolio variant" map directly onto one-line file edits.
- **Preferences persist.** CLAUDE.md means I correct the agent once, not once per session.

None of this required resume-specific tooling. It's the same discipline we already apply to code, applied to the one document most of us still manage like it's 2009: version it, decompose it, generate the artifacts, and write down the conventions. Do that, and an AI agent becomes a genuinely good resume assistant instead of a fancy autocomplete pasting into Word.

---

*You don't have to rebuild this from the post. I published a template at [github.com/nathanfox/resume-as-code](https://github.com/nathanfox/resume-as-code): the build script, three sample variants with placeholder content, committed example PDFs, and the CLAUDE.md conventions, ready to fork and fill in. Steal it.*

# Not the same as...

Spec-driven development for Claude Code is not a new category. Before picking Mock-First,
it's worth knowing what else is out there — some of it is more mature.

## Similar projects

- **GitHub Spec Kit** — `/speckit.specify → /plan → /tasks → /implement`. Broader tooling,
  official backing, no visual mockup gate.
- **ShipSpec** — `PRD → SDD → TASKS → implement`. Similar shape to Mock-First's document
  chain, no mockup gate, no dedicated change-impact command.
- **PRD Workflow Manager**, **Claude Code Spec Workflow**, **goal-workflow** — same general
  PRD → task → implement pattern, varying levels of polish.
- **interactive-spec** (skill) — generates an HTML mockup from a spec, but as a standalone
  skill, not a required step in a pipeline.

## What Mock-First adds

1. **Mockup is a gate, not an option.** `/break-task` checks that every screen listed in the
   PRD has an approved mockup before it will break work into tasks. Agreement on the shape
   of the UI happens before code, not after.
2. **`/revise` reports impact before it changes anything.** Most workflows above assume you
   edit the PRD/tasks by hand when plans change mid-project. Mock-First's `/revise` reads
   the PRD, task list, and git history, classifies the change, and prints exactly what
   becomes outdated — before touching a single file.
3. **Nothing is deleted.** Superseded tasks are marked `⚠️ SUPERSEDED` and left in
   place with a pointer to their replacement. Superseded mockups and old PRDs move to an
   `archive/` folder instead of being overwritten.

## What we don't have

- No installer, no dashboard, no subagents, no MCP server — five markdown command files.
- No automated visual verification (screenshot diff between mockup and implementation).
- No multi-developer conflict handling or branch orchestration.
- Task phases assume a fairly conventional data → logic → UI → integration shape; unusual
  architectures may need the command files tweaked.

If you don't need the visual gate or the change-impact analysis, the projects above are
more mature and worth using instead. If those two things are what you're missing, that's
what Mock-First is for.

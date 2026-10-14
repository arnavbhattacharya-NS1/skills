---
name: pstack
description: Run the pstack plugin (poteto-mode) inside Bob, composed with ponytail and thermo-nuclear-code-quality-review. Use for /pstack, "use pstack", "poteto mode in bob", or any task that needs pstack's rigor (playbooks, principles, verification) together with minimal-code discipline and a strict final quality review.
metadata:
  argument-hint: "[task] [ponytail=lite|full|ultra] [review=off]"
---

# pstack for Bob

pstack was written for Cursor. This skill is the Bob adapter. It runs `poteto-mode` as the main workflow, turns on `ponytail` for what gets built, and ends with `thermo-nuclear-code-quality-review`. Do not re-implement any of them. Read them and follow them, applying the translations below.

## Step 1. Load the three skills with `read_file`

Many pstack skills are manual-only, so `use_skill` can be refused. Always use `read_file` on the absolute path (expand `~` to the user's home directory).

1. `~/.bob/skills/poteto-mode/SKILL.md`, read in full. It is the workflow. Its playbooks live in `~/.bob/skills/poteto-mode/playbooks/`, its bugbot triage notes in `references/`, its helpers in `scripts/`.
2. `~/.bob/skills/ponytail/SKILL.md`, via `use_skill` (it is auto-invocable) or `read_file`. Default level is **full**. Honour a `ponytail=` argument.
3. `~/.bob/skills/thermo-nuclear-code-quality-review/SKILL.md`. Read it now, apply it at Step 4.

Every `poteto-mode` reference to a skill named `X` (`how`, `why`, `architect`, `swarm`, `arena`, `interrogate`, `reflect`, `unslop`, `no-comments`, `technical-writing`, `figure-it-out`, `show-me-your-work`, `benchmark-checklist`, `principle-*`) means `~/.bob/skills/X/SKILL.md`. Read the leaf file with `read_file` when you apply it.

## Step 2. Run poteto-mode with these Bob translations

| poteto-mode / pstack says | In Bob do this |
|---|---|
| `Task` tool, `subagent_type: "poteto-agent"` | `spawn_subagent`. Use `name: "explore"` for read-only work and `"general"` otherwise. Several `spawn_subagent` calls in one turn run in parallel. A `poteto-agent` does not exist, so put "First `read_file` `~/.bob/skills/poteto-mode/SKILL.md` in full and follow it" in the subagent description. Set `fork_context: true` only when it needs the prior discussion. |
| `run_in_background`, explicit `model` per role, `/setup-pstack` | Not supported. Ignore model names (grok, opus) and reasoning budgets. When a skill asks for a multi-model second opinion, spawn a second subagent with a differently framed prompt and say plainly that it is the same model. |
| `AskQuestion` | Ask the user in plain text, or `ask_followup_question` if available. Poteto's rule still holds: ask only about product or preference forks, never about facts you can observe by running something. |
| `TodoWrite` / todolist | `update_todo_list`. The matched playbook's steps go in first, verbatim. |
| `create-skill` | Bob has a built-in `create-skill`. Use it. |
| `deslop` (from `cursor-team-kit`) | Not installed. Do the same job by hand: strip comments that restate code, defensive checks on impossible cases, and needless abstractions from your own diff. |
| `control-cli`, `control-ui`, Cursor `babysit`, Origin, `drive` | Not available. Verify with the real artifact using `execute_command` (run the CLI, run the tests, start the app and hit it). Say so if a surface cannot be driven from here. |
| `~/.cursor/...`, `.cursor/rules/*.mdc` | Cursor-only. Skip. Do not run `/setup-pstack`. |
| `bun` scripts in `poteto-mode/scripts/` | Run only if `bun` is installed (`command -v bun`). Otherwise do the step by hand. |

If a skill depends on something not listed here and Bob cannot do it, say which step you skipped. Never pretend it ran.

## Step 3. Compose with ponytail

- **Ponytail decides how much gets built.** Climb its ladder before any playbook's build step. It overlaps with pstack's `principle-laziness-protocol` and `principle-subtract-before-you-add`. Treat them as one rule, not two.
- **Poteto decides process and the final reply.** The playbook's required reply sections stay in full. Ponytail's "code first, three lines" limit applies to code-change summaries only when no playbook reply is required. Explanation a playbook asks for is not ponytail debt.
- **Rigor beats laziness on verification.** Never skip the playbook's verification, a reproduction, or ponytail's own "one runnable check" for non-trivial logic. Ponytail shortens the solution, not the reading or the proof.
- Mark deliberate corner-cutting with a `ponytail:` comment, per ponytail's rules.

## Step 4. Finish with the strict review

After the work passes its own verification and before the PR or the final reply:

1. Apply the `no-comments` skill to the diff.
2. Run `thermo-nuclear-code-quality-review` on the branch's changes (`git diff` against the base). Skip it only if the user passes `review=off` or the change is trivial (a typo, a one-line config). Say which.
3. **Filter its findings through ponytail.** Take a restructuring only if it leaves the code smaller or simpler, or removes a real maintainability hazard. Reject "ambitious" rewrites that add layers, files, or abstractions. Apply the accepted fixes, re-run the verification, and report what you rejected and why.
4. Then follow the matched playbook's PR and reply steps, written per poteto's "Writing the reply" rules.

## Step 5. Start

Take the task from `$ARGUMENTS` or the user's message. If there is none, ask for it. State in one line which playbook matched, the ponytail level, and whether the review is on. Then open the todolist and begin.

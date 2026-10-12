---
name: planning-board
description: Use when the user says to start a job, take one on, move to the next job, or asks for a plan before implementation — any piece of work with more than one task, where each task needs its own written proposal approved in words before any implementation file is touched, tracked on a published status page. Large plans can be split into a stacked PR, only if the user agrees.
---

# Planning Board

## Overview

The work is tracked on a **board**: every task gets its own proposal, approved before
any implementation file is touched, on a published HTML page updated in the same turn
as every status change.

A **job** is a group of tasks from the Queue that are implemented together, in one
sitting, as one change the user can read, understand and review afterwards. Grouping
pays for the reading once and turns several small diffs into one coherent one.

A **stack** is an optional layer above jobs: the jobs are split into an ordered chain of
pull requests, each targeting the branch of the one below it. It exists only when the
plan is big enough to split cleanly **and the user chose it** (see *Stacked PRs*).

```
stack (optional) --> PR 1, PR 2, ... --> jobs --> tasks
```

**The rule: nothing is implemented until the user says yes, in words, to that specific
task** — naming the task, or naming the job that contains it. A thumbs-up on another
task or job, silence, or your own confidence that the fix is obvious are not approval.

## When to use

- The user says "start a job", "take the next job", "let's work on X", "plan this out".
  No board yet → build one (steps 1–3), then take job 1. A board exists → take the
  top job on its Jobs tab. A board with no Jobs tab (made before jobs) → run step 3 on
  it first, adding the Jobs panel and the Job column from `template.html`. A board with
  jobs but no stack decision, whose remaining jobs pass *Does it qualify*, gets step 3b
  at the next job boundary.
- Any work with 2+ tasks that will be worked over more than one turn.
- The user asks for a plan, or asks what is wrong with something and what it would take to fix.

**Not for:** a single-file edit the user directly asked for, answering a question, or
work already approved on an existing board (update that board instead of making a new one).

## The loop

```
user says "start a job" / "take the next job"
  |
  v
1 INVESTIGATE (read-only) --> 2 PUBLISH SKELETON --> 3 GROUP the Queue into jobs
  |                            (tasks Open)          (Jobs tab, job on each Queue row)
  v
3b STACK? only if it qualifies: propose the PR split, draw it, STOP for the user's
   yes / no / changes. No stack unless they say yes.
  |
  v
4 PRESENT the next job: job header + one proposal per task in it
  + move those tasks to "Plan submitted" (same turn)
  |
  v
5 STOP and wait --(user approves the job or tasks, in words)--> Approved
  |                                                                 |
  |<--(task rejected)-- back to Open, out of the job                v
                                           6 IMPLEMENT the job's approved tasks together
                                                                    |
                                                                    v
                                           7 VERIFY each task --> Fixed
                                                                    |
                                                                    v
                                           8 JOB REVIEW in chat + on the Jobs tab
                                             (board updated in the SAME turn as each move)
                                                                    |
                                                    stacked and that was the PR's last job?
                                                                    v
                                           9 PR HANDOFF: body on the Stack tab (copy button),
                                             the short fields + next branch name in chat;
                                             branch check before the next PR's first edit
```

**Step 1 — investigate.** Read anything. Run read-only checks: greps, a throwaway probe
in the scratchpad, a suite that already exists. Do **not** edit a file the fix would
touch, and do not leave scratch files in the repo. Plan against the code, not against
what the user told you — if the premise does not hold, say so instead of proposing a fix.

**Step 2 — publish the skeleton** if the investigation runs longer than a couple of tool
calls, so the user has something to watch. Tasks start at `Open`. Give them the link once.

**Step 3 — group.** Split the Queue into jobs using the rules in *Forming jobs* below.
Fill the Jobs tab and the Job column on the Queue. Rank jobs, not just tasks. Effort and
risk for jobs not yet presented are provisional — mark them `~M · ~Low` until their
proposals are written.

**Step 3b — stack?** Check the plan against *Does it qualify* below. If it does not, say
nothing and go on. If it does, propose the split and stop for the user's answer before
presenting job 1. The stack changes the order jobs are worked in, so it is settled first.

**Step 4 — present the next job.** Start with the job header (below), then one proposal
per task in the job, in chat, under the seven headings. Never one proposal covering
several tasks — the board tracks tasks, and a merged plan leaves the others with no
recorded decision. In the same turn, set every task in the job to `Plan submitted` with
its effort and risk letters on the row, and the job to `Plan submitted`.

**Step 5 — stop.** The user may approve the whole job in one reply ("approve job B") or
task by task. Approving a job approves its tasks; it does **not** answer their Legacy
questions — each still needs its own answer before that point is implemented. A task they reject or park leaves the job (back to `Open`, or `Deferred`)
and the job goes ahead without it; re-check that the rest still makes sense together.
Ask for the lane exception here too (below), once per job, naming every directory the
job reaches.

**Step 6 — implement the job.** All of the job's approved tasks, in the order the job
header gave. Every hunk belongs to exactly one task — no edit that serves none of them,
no "while I was here" cleanups. Reviewability is the deliverable.

**Step 7 — verify before claiming Fixed.** Run the checks for each task and quote what
they printed. Not after writing the code — after proving it.

**Step 8 — job review.** Hand the change back in the shape in *Job review* below, and
put its one-line version in the job's Review cell. The job reaches `Fixed` when every
task in it has.

**Step 9 — PR handoff (stacked only).** When the job just reviewed was the last one in
its PR, hand the PR over in the shape in *PR handoff* below, in the same turn as the job
review.

## Forming jobs

A job is a set of Queue tasks that **share the reading and tell one story in the diff**.
Put tasks in the same job when they:

- touch the same files or the same seam (one module, one request path, one component), or
- are the same kind of fix in neighbouring code (the same missing guard in three handlers).

Keep a job reviewable in one sitting:

- **One lane.** A job does not cross a service boundary. Backend and frontend fixes
  for the same symptom are two jobs, the first noted as unblocking the second.
- **About five tasks or fewer**, and a diff a reviewer can read top to bottom.
- **A High-risk or L-effort task is a job of its own.** Its rollback plan and real-data
  check must not be buried in someone else's diff.
- **A task waiting on a design decision** stays out of a job until the decision is made.
- **A task that fits nowhere is a job of one.** Never force a group.

**Rank jobs** by the highest-ranked task in each. Pulling a lower-ranked task forward
because it shares files with a top task is the point of a job — mark it on the Queue row
("pulled into job A") so the user sees why the order moved.

### The job header

Before the task proposals, in chat:

1. **Tasks** — the task numbers and one line each.
2. **Why together** — the files or seam they share, concretely.
3. **Files touched** — every file the job will edit.
4. **Order** — the order the tasks will be implemented, and why if it matters
   (task 05's guard first so 01's change has a non-empty list to work on).
5. **Effort · Risk** — of the job as a whole: the highest of its tasks, plus anything the
   combination adds.

### Job review

After implementation, before anything else, give the user what they need to review the
change without re-deriving it:

1. **What changed, per task** — task number → `file:line` ranges it touched, one line on
   what each hunk does. A hunk that serves two tasks is listed under both and said so.
2. **Read it in this order** — the file or hunk to start from, and the path through the
   rest.
3. **Proof** — per task, the check that was run and what it printed.
4. **Review commands** — the work is uncommitted, so give `git diff -- <files>` for the
   job. Where a file belongs to one task, list that command under the task so it can be
   read alone; where tasks share a file, the line ranges from item 1 mark whose hunk is
   whose.
5. **What did not change** — anything the user might expect to be touched and was not, and
   any task that left the job.

## Stacked PRs

A stack splits a large plan into a chain of PRs a reviewer can take one at a time. PR 1
targets the base branch, PR 2 targets PR 1's branch, and so on. **Whether to stack is the
user's decision.** You propose it, they choose. Without a yes in words there is no stack.

### Does it qualify

Propose a stack only when **both** of these hold:

1. **Big enough.** Three or more jobs, or a total change one reviewer could not read in
   one sitting (roughly 400+ changed lines or 15+ files). Two small jobs are one PR.
2. **Splits cleanly.** There is an order where every PR builds, passes its tests and
   makes sense if merged alone, and later PRs build on earlier ones. Natural cuts are
   foundation then consumers, backend then frontend, refactor then feature, migration
   then the code that uses it.

It does **not** qualify when the jobs don't depend on each other. Those are independent
PRs, not a stack; say so in one line if it helps. It also does not qualify when every cut
leaves a PR that breaks the build until a later one lands.

Rules for the cuts:

- **A job belongs to exactly one PR.** Never split a job across PRs. If a cut wants to go
  through a job, re-form the jobs.
- **One to three jobs per PR**, ordered by dependency first. Inside a PR, jobs keep their
  rank order. The stack order overrides job rank, so say when it moves a job.
- **A High-risk job may get a PR of its own**, so it can be reverted alone.

### Proposing it

One message, then stop:

1. **Why a stack**: one sentence on the size, and the dependency that gives the order.
2. **Per PR**: number, proposed title, proposed branch, target branch, its jobs with their
   task numbers, and one line on why the cut is there.
3. **The drawing**: base branch at the top, each PR indented under the one it targets:

```
main
 └─ PR 1  auth-cleanup/1-token-model   Jobs A, B   tasks 01 03 | 02
     └─ PR 2  auth-cleanup/2-api-guards   Job C       tasks 04 05
         └─ PR 3  auth-cleanup/3-ui-states   Jobs D, E   tasks 06 | 07 08
```

4. **Ask**: stack it like this, change the cuts, or keep it as one change. No default.

Branch names follow the repo's convention, which you can see in `git branch -a` and recent
merges. With no convention, use `<topic>/<n>-<slug>`. PR 1's branch can be the branch the
user is already on.

**Approving a job is not approving the stack**, and approving the stack approves no task.
If they say no, record "No stack — declined" in the header meta and don't ask again
unless the plan grows by half or more. If they say yes, add the Stack tab (template), put
each job's PR on its Jobs row, and present PR 1's first job.

### Working a stack

- **Jobs are implemented in PR order.** No edit for a PR N+1 job until PR N has had its
  handoff and the branch check below has passed. Proposals for later jobs may be
  presented early. Implementing them may not start early.
- **Each PR must stand alone.** Before the handoff, run the suite that covers the whole
  PR, not only each task's check, and quote it.
- **Branch check.** Before the first edit of each PR, run `git branch --show-current` and
  `git status --short`. You should be on that PR's branch with a clean tree, which means
  the previous PR's work is committed on its own branch. If not, stop and say what is off.
  Never carry one PR's uncommitted work into the next PR's branch. Run the base probe
  (below) in the same check.

### When the base moves

**Default: leave the stack alone.** New commits on `main` need no action by themselves.
Only PR 1 targets it, and GitHub only needs PR 1 to merge without conflicts. Sync only
when one of these is true:

- PR 1 conflicts with the base, on GitHub or in the probe below,
- the repo requires branches to be up to date before merging,
- the stack needs something that landed on the base,
- a PR in the stack landed by squash or rebase merge (see the table).

**Sync by merging, never by rebasing.** A merge rewrites nothing, so there is no
force-push and review comments stay attached. Every sync merge goes through
`youssef-skills:merge-assistant` in its *Stack mode*. Load it and follow it. If the user
asks for a rebase instead, give the `git rebase --update-refs` command with backup tags.
You run nothing.

**The base probe** is read-only, so run it at every branch check:

```
git fetch origin
git rev-list --count <PR 1 branch>..origin/main         # how far main has moved
git merge-tree --write-tree origin/main <PR 1 branch>   # exit 1 = conflict (git 2.38+)
```

Report it in one line, such as "main is 7 commits ahead of PR 1 and merges cleanly, no
action needed", and put it on the Stack tab's base line. A clean probe is not a reason
to sync. On older git, use merge-assistant's dry-run in a detached worktree instead.

| What happened | What to do |
|---|---|
| The base moved and a sync is needed | 1. **Base merge**: merge the base into PR 1, a full merge-assistant run. 2. **Propagate** bottom-up: PR 1 into PR 2, PR 2 into PR 3, each a stack-mode propagation run. |
| PR N landed with a merge commit | Retarget PR N+1 to the base. GitHub does this itself when PR N's branch is deleted on merge. Nothing to merge. |
| PR N landed by squash or rebase merge | The base now has PR N's changes under new commits, so PR N+1's diff shows them again. Merge the base into PR N+1 as a propagation run, where the known change is PR N as it landed. Then retarget. Keep PR N's branch until this is done, because the run compares against it. |
| Review changed PR N after PR N+1 started | Propagate PR N into PR N+1, then upward. The known change is PR N's new commits. |
| The user moves the stack to a new base | Base merge of the new base into PR 1. The user retargets PR 1, then propagate. |

- **One link at a time, bottom-up.** A link is done when its merge is verified and the
  user has committed it. The user commits, and you never push. Don't start the next link
  before that.
- **Verify every link, even the easy ones.** Knowing what changed shortens the analysis,
  not the checks. Run the dry-run, the clean-but-dangerous scan, the PR's whole suite, and
  the task checks from that PR's job reviews, and quote what each printed.
- **Stop the cascade on any failure** and say which link it stopped at. Links above it
  stay as they were.
- **Recommend merge commits for landing.** Every PR config carries "Merge method: merge
  commit". A squash forces a propagation run on every PR above it.
- **Board, same turn.** Each sync updates the PR's sync line on the Stack tab: the base
  commit it is synced with, what was merged, the decisions and the checks run. If a merge
  changed what a PR does, update its Body config and tell the user to edit the PR.

### PR handoff

**The body goes on the board, never in chat.** Write the full PR body into the PR's
config on the Stack tab, where its Copy button copies it, and republish. Do not paste the
body into the response, not even a shortened one. Chat carries the short fields, each in
its own fenced block so it can be copied alone, and points at the board for the body.
Nothing goes in a file unless the user asks.

~~~text
**PR 2 of 3 is ready: Guard empty token lists in the API layer**

Head branch
```
auth-cleanup/2-api-guards
```
Target branch
```
auth-cleanup/1-token-model
```
Title
```
Guard empty token lists in the API layer
```
Body: on the board, Stack tab → PR 2 → PR config → Copy
Draft: no · Merge method: merge commit · Labels: `bug` · Reviewers: (only what the repo shows; else leave out)

Next branch: PR 3 targets this one
```
auth-cleanup/3-ui-states
```
```
git switch -c auth-cleanup/3-ui-states
```
~~~

- **Title** follows the repo's PR/commit style (`git log --oneline`).
- **Body**: if the repo has `.github/pull_request_template.md` (or a `PULL_REQUEST_TEMPLATE`
  folder), fill that. Otherwise use these sections: *Summary* (1–3 sentences), *Stack*
  (PR n of N; the PR it depends on, by number once the user has given it, else by branch;
  the PR that builds on it), *Changes* (per task: what and where, from the job reviews),
  *How to review* (read order), *Testing* (per task, the check and what it printed, plus
  the whole-PR suite), *Not in this PR*.
- **PR 1's body opens with the stack overview**, before its own sections. It is the
  first PR a reviewer sees, so it explains the whole stack: a *Stack overview* section
  with 2–3 sentences on what the stack does as a whole, then one line per PR giving its
  number, branch and responsibility, with PR 1 marked "this PR". Then PR 1's own body
  follows as usual. Later PRs keep only their own *Stack* line. If the cuts change after
  PR 1 is opened, update the overview on the board and tell the user to edit PR 1.
- Put the body in the PR's `pre.cfg-v` Body field,
  HTML-escaped (`&` `<` `>`) so the copied text is the exact Markdown. End it with any PR
  attribution line the session requires.
- **Labels and reviewers**: only when the repo shows them, through existing labels,
  CODEOWNERS or the user's past PRs. Never invent one. **Draft**: say yes or no; the user
  decides. **Merge method**: merge commit, so the PRs above need no propagation run
  when it lands (see *When the base moves*).
- **Next branch**: give the name and the create command, and say the user should commit
  this PR's work first. Leave it out for the last PR.

In the same turn, set the PR to `Ready` on the Stack tab, fill its whole config there
(the body included), open its config section, and republish before you reply, so the
board link already has the body when the user reads the reply. When
the user says they opened it, record the number and link. When it is merged, mark it
`Merged`, ask which merge method was used, and follow *When the base moves* for the PR
above it. Later PR bodies then refer to it by number.

PR statuses: `Planned` → `In progress` (its first job approved) → `Ready` (handed over) →
`Opened` (#n) → `Merged`. A PR's progress is jobs Fixed / jobs in it.

## The seven headings

Every proposal, in this order. Explain them in chat in the style below; the board carries
the one-line version.

| # | Heading | What goes in it | What makes it wrong |
|---|---------|-----------------|---------------------|
| 1 | **Issue** | What is wrong, in one or two plain sentences, for someone who has never opened the file. | Restating the user's own words back. Say what *you* found. |
| 2 | **Why it's an issue** | The mechanism, with `file:line` for every behaviour claim, and who notices — a user seeing an empty chart, an operator, a silent log nobody reads. | "This is unsafe" / "bad practice". If you cannot name a sequence of events that produces a wrong result, you have not found the issue yet. |
| 3 | **The fix** | What changes, in which files. The smallest change that does the job. Name what you are deliberately *not* changing. One recommendation if there are several approaches, not a survey. | A refactor riding along. Speculative abstraction. Restructuring surrounding code to suit the edit. |
| 4 | **What we gain** | The behaviour afterwards, stated as something observable: "reloading a summarised chat shows its charts again". | "Improves correctness". Benefits nobody can see or measure. |
| 5 | **Effort** | **S** — one file, one change, covered by an existing suite. **M** — a few files or one seam, needs a new test. **L** — crosses a service boundary, or needs a migration, a contract change or a design decision; name the decision. | An hours estimate. The letter plus the reason is what is wanted. |
| 6 | **Risk** | **Low** — blast radius is the thing being fixed. **Medium** — a shared path, so a mistake reaches other features; name a verification step beyond "tests pass". **High** — can destroy data, change a persisted shape, or move behaviour no test asserts; needs a rollback plan and a check against real data. | A risk level with no verification step attached. "Low risk" on a migration or a shared path. |
| 7 | **Legacy** | See below — this one has its own rule. | Leaving it out. Quietly adding a fallback and not declaring it. |

### Heading 7 — Legacy is the user's decision, never yours

**There is no default.** Do not assume the old path is removed, and do not assume it is
kept. Name the trade-off and hand it over:

1. **What legacy logic is in reach** — the old shape, name, column, route or code path,
   concretely, and where it is read or written.
2. **What we lose without it** — what breaks, and who sees it. "Chats summarised before
   the June change show no summary" is an answer; "some old data" is not.
3. **Your recommendation** — support it or drop it, and one sentence on why.
4. **Ask.** The user answers. Nothing is implemented on this point until they do.

"None — there is no legacy logic in reach" is a fine answer. "Not sure" is not; you read
the file, so you know.

## How to explain it in chat

The board is a **status board**. The reasoning goes in chat, written for a junior engineer:

1. **Plain words.** Short sentences. Define a term the first time you use it.
2. **Always a concrete example.** Real code, a real value, a real sequence — not theory.
   Not optional.
3. **A flowchart whenever the failure travels through more than one function**, as a
   simple ASCII flow: `A --(does X)--> B --(does Y)--> C`.
4. **Keep it small.** Short numbered steps, never one long block. Go deeper only when asked.

The `sha3bolly` skill ships in this plugin (`youssef-skills:sha3bolly`) and is the canonical
statement of this style — load it before writing the first proposal and follow it.

## Status lifecycle

| Pill | Means | Set it when |
|---|---|---|
| `Open` | Confirmed, nobody is on it. | The resting state. Also where a task returns if its plan is rejected. |
| `Plan submitted` | A proposal exists, waiting on the user. | The moment you present it. Put effort and risk on the row. |
| `Approved` | Approved, being implemented. | Only after the user approves in words. **The only status under which implementation files may be edited.** |
| `Fixed` | Changed, and the failure can no longer happen. | After verifying — not after writing the code. Say what you ran and what it printed. |
| `Partly fixed` | Some of it is done, the rest is not. | When scope was deliberately cut. Say which half is which. |
| `Refuted` | The mechanism was checked and does not hold. | Keep the row. A disproved issue that gets deleted is one the next reviewer finds again. |
| `By design` | Real behaviour, intended. | Same rule — keep it, with the evidence that it is deliberate. |
| `Deferred` | Real, confirmed, parked. | When the user parks a task rather than approving or rejecting. It leaves the queue, keeps its plan, and is **still an open exposure**. |
| `Assigned out` | Owned by someone else now. | Handed to another person or filed as an issue. Keeps its row and its plan. |

## Keeping the board current

**The row moves when the thing happens** — not at the end of a batch, not "once it is all
done". Every move updates **three things together**:

1. the **pill** on the row,
2. the **Evidence / decision** cell — what you did and what proved it, in 1–3 sentences,
3. the **tally in the header and the filter chip counts** — these are hand-maintained and
   will silently disagree with the rows if you skip them.

The **Queue** is maintained too: a task that reaches Fixed leaves it and the ranks close
up. If fixing one task changes another's priority, say so and re-rank. The **Jobs** tab
moves with it: a task leaving a job is removed from that job's row, and a finished job
keeps its row with its review.

**Republish to the same URL** (pass the artifact's `url`) so the link keeps working. If
the board and the code disagree, everyone downstream is working from the wrong map.

## Two standing checkpoints

**Lane / boundary.** Name every directory the job's fixes reach. Before editing anything
outside the lane you were given, **ask once per job**, naming every directory. A small fix
is not an exception; small is not the criterion.

**Never commit.** Implementation lands **uncommitted** in the working tree. No
`git commit`, `git push`, `gh pr create`, amend or tag unless the user says so, in words —
and being told once is not permission for the next time. **A stack does not change this.**
Saying yes to the stack is not permission to commit, branch, push or open PRs. You give
the config and the branch names, and the user runs them. Permission covers only what the
user named: "create the branches for this stack" covers branches, not commits or pushes.

## Building the board

Copy `template.html` (next to this SKILL.md) and fill it in. It is a single self-contained file: tabs, status
pills, filter chips and tally, light and dark, works at phone width.

**Scale the tabs to the board.** The template ships six; delete what the board does not need,
and add Method by copying the Queue panel and changing its id.

| Tab | In template | Keep / add it when |
|---|---|---|
| Before you start | yes | Always. It is the protocol the user and the next agent read. |
| Jobs | yes | Always. Every job in rank order: its tasks, why they are together, files touched, effort · risk, and the review once it is done. |
| Queue | yes | Always. Open tasks in rank order — severity weighted by reach, certainty and cost — with the job each belongs to. |
| Tasks | yes | Always. Every task, grouped by area, with the status filter chips. |
| Stack | yes | Only when the user said yes to a stacked PR. The stack drawn top to bottom, with the base probe on the base line, and per PR: status, progress, branch → target, sync line, its jobs, and the config to copy. Delete it otherwise. |
| Manual tests | yes | Only when something needs a browser, a real provider, or a judgement call. |
| Method | add | Only when how the task list was produced is itself worth recording. |

A job's pill is the least-advanced status among its tasks still in it, and moves when
theirs do.

A tab's count in the nav (`<span class="c">`) is hand-maintained like the tally.

Load `artifact-design` before the first publish — the template already satisfies the page
contract, so you are checking your additions, not re-deriving the page.

Write it to the scratchpad directory, then publish with the `Artifact` tool. Give the user
the link once, in one line.

## Red flags — stop

- About to edit an implementation file for a task that is not `Approved`.
- Writing one proposal that covers several tasks — a job has a header plus one proposal
  per task, never a merged plan.
- Implementing a task that is not in the job being worked, or an edit that serves no
  task in it.
- Putting a High-risk task, or tasks from two lanes, in one job.
- Finishing a job without the job review.
- Heading 7 says "removing the old path" without the user having answered.
- A status changed in the code but not on the board this turn.
- Marking `Fixed` before running the verification.
- Editing a directory outside the lane because the fix was small.
- Publishing a second board when one already exists for this work.
- Splitting into a stack, or choosing its cuts, without the user saying yes in words.
- A job split across two PRs, or a PR that breaks the build until a later one lands.
- Editing for the next PR before the handoff and branch check, or with the previous
  PR's work still uncommitted in the tree.
- Creating a branch, committing, pushing or opening a PR yourself because a stack exists.
- A PR's last job reviewed without the PR handoff in the same turn.
- A PR body pasted into chat. It belongs on the Stack tab, behind its Copy button.
- Rebasing or force-pushing a stack branch. Syncs are merges, through merge-assistant.
- Syncing the stack because main moved, when the probe is clean and nothing needs it.
- Skipping a propagation run's checks because "we already know what changed".
- Starting the next link of a sync cascade before the previous one is verified and committed.

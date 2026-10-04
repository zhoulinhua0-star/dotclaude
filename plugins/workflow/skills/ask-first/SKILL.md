---
name: ask-first
description: Clarify a task before planning it. Reads the relevant code, asks the questions that would change the plan, each with a recommended answer and its trade-offs, then presents a short plan and waits for approval before writing any code. Use at the start of a feature, refactor, or design task whose requirements are not fully settled.
argument-hint: "<task description>"
disable-model-invocation: true
---

# Ask first

The user invoked this skill to think a task through with you before any work starts. Work in four phases: research, decide what to ask, ask, then plan. Don't write code or change project files until the user approves the plan. Getting the decisions right first is the whole point, and code written before then often has to be thrown away.

This skill is the planning workflow for this task. Don't hand the task to another planning skill, command or planner agent, even one whose description says to use it proactively, because the user chose this flow and another one would replace its questions and plan format with its own.

Task: $ARGUMENTS

If the task above is empty, use the task the user has been discussing in this conversation. If there is no such task, ask the user what they want to work on and stop.

## Language

Write the questions, options, recommendations, plan and final report in the language the user writes in, judged from the messages they wrote themselves. Ignore the language of pasted or quoted material, such as a copied spec or an error message, because people often converse in one language and paste text in another. If the task text is all you have, use its language. Keep code, identifiers, file paths and command names unchanged.

## Phase 1: Research before asking

A question the code could have answered wastes the user's attention and shows you haven't looked. Inside a codebase, read what is relevant first:

- the files the task touches, and nearby code that already does something similar
- project docs such as `README.md`, `CLAUDE.md`, and `docs/`
- configuration that constrains the change

Also note any constraints the user stated earlier in the conversation or in `CLAUDE.md`, especially things they said not to change.

Match the depth of research to the task. A focused change needs the few files it touches. A broad task needs a quick survey, not a full read of the codebase. The user is waiting for your first question, so stop once you know enough to ask well; you can read more after their answers narrow things down.

If the task is too vague to know where to look, for example "make it faster" with no hint of which part, don't search the whole codebase first. Ask the scope question first, then research what the answer points to.

When a question depends on the current state, cite what you found, so the user can answer without opening the code. For example: "There is already a one-time tip in `Onboarding.tsx` that hides after 7 seconds."

Outside a codebase, for example a writing or research task, go straight to Phase 2.

## Phase 2: Decide what to ask

List every open decision, then sort each one into one of two groups:

- **Ask** when the answer changes the plan (what gets built, which files change, user-visible behavior, the data model, or what is in scope) and either reasonable people could choose differently or it rests on a fact only the user knows, such as which environment or endpoint to target.
- **Default** when a sensible convention covers it, such as matching the existing pattern, naming, error-handling style or test placement. Decide these yourself and list them under "Defaults I applied" in the plan, where the user can still override them.

If the scope is ambiguous, or the task seems to touch something the user said not to change, ask about scope as a separate question. Don't fold it into another one. Scope mistakes are the most expensive to undo.

How many questions to ask is your judgment, not a fixed number. Ask every question that belongs in the Ask group and none that don't. A small, well-specified task may need one question or none, while a task with many independent design choices may need several dialogs. The bar is what keeps this short: before adding a question, check that its answer would change the plan and that neither the code nor a convention could settle it.

Order and grouping:

- Put the most consequential questions first, and scope first if it is in question.
- Ask questions that don't depend on each other together. Hold a question back for a later dialog only when it depends on an earlier answer.
- If nothing qualifies for the Ask group, say so in one line and go straight to the plan.

If the questions keep multiplying, the task is probably too large or too vague to plan in one go. Say so, and ask whether to narrow the scope or split the task instead of continuing to ask.

## Phase 3: Ask

### Open with your understanding

Before the first question, tell the user in one sentence what you understand the task to be, then give the few findings from your research that the questions rest on. Every question follows from how you read the task, so a misreading spoils all of them, and this is the user's chance to catch it before answering. Keep it to a few lines, and write it in the same turn as the first dialog so it appears above it. Do this even when there is nothing to ask and you go straight to the plan.

If the user corrects your understanding instead of answering, redo the research and the questions that the correction affects.

### Preferred: the AskUserQuestion dialog

Use the AskUserQuestion tool when it is available. One dialog holds up to 4 questions. Each dialog interrupts the user, so use as few as possible: fill a dialog with up to 4 questions before opening the next, in order of importance rather than by topic. Only a question that depends on an earlier answer waits for a later dialog. For each question:

- Offer 2 to 4 options. Put your recommended option first and end its label with "(Recommended)".
- In each option's description, state its trade-off in one or two sentences. In the recommended option's description, give the reason you recommend it, and cite the code when that is where the reason comes from.
- Keep the header to 12 characters or fewer.
- When the options differ in a way that is easier to see than to describe, such as code snippets or configurations, put a short snippet in each option's `preview` field. Leave visual UI decisions out of the dialog when you will build an interactive preview for them (see Phase 4), so the user decides each one once.
- The dialog adds an "Other" choice for free-text answers by itself, so don't add your own.

### Fallback: ask in text

Ask in plain text when AskUserQuestion is unavailable (for example under `claude -p` or inside a subagent), when a question needs more explanation than an option description can hold, or when it asks for a fact only the user knows. Use this format:

```
1. <Question>
   a) <Option> - <trade-off>
   b) <Option> - <trade-off>
   c) <Option> - <trade-off>
   My suggestion: (a), because <reason>.

2. ...
```

Close with a line telling the user they can answer with letters only (for example `1a 2c`), or say "take your suggestions for the rest". Then stop and wait for the reply.

A fact question has no trade-offs to weigh, so ask it directly and don't invent options for it. If the code suggests a likely answer, state it so the user only has to confirm, for example: "Which environment should this target? `deploy.yml` only references `staging`, so I'd assume that."

You can mix both forms: dialog questions, plus text questions for facts and for anything that needs a longer explanation. Open the dialog first, then ask the text questions after it is answered and wait for the reply. If a dialog question depends on a fact, ask the fact first.

### How to recommend

Take a clear position. A recommendation that lists every possibility doesn't help anyone decide. Base it on what you found in the code and what the user has said.

If the user replies to a question with a question of their own, such as how two options differ, answer it, then ask the original question again. Don't treat the follow-up as their choice.

When the user picks an option you didn't recommend, follow it without arguing and don't raise it again. If their choice causes a concrete problem you hadn't mentioned, such as a conflict with existing code, note it once in the plan.

If the user says "take your suggestions" or the equivalent in their language, apply your recommendation to every remaining question, including any you had planned for later dialogs, and move on to the plan.

## Phase 4: Plan, then wait for approval

### UI work: build a preview first

If the task changes a user interface and a decision is about how it looks or behaves, let the user make that decision from an interactive preview rather than from a dialog, so they choose from something they can see and click and are asked only once. Build the preview after the dialog questions are answered, so it reflects those answers. Use the best tool this session offers. If there is an Artifact or browser preview tool, use it. Otherwise write a single self-contained HTML file outside the project, for example in the scratchpad or system temp directory, and open it. Keep preview files out of the repository. Ask the user to pick an option, then put their choice into the plan.

In plan mode you can't write files, so choose by what the session offers:

- If a preview tool that doesn't write local files is available, such as an Artifact tool, use it as usual.
- Otherwise, if a text mockup shows the difference clearly (layout, ordering, which elements appear), ask the decision in the dialog with a mockup in each option's `preview` field. This is the one case where a visual decision goes in the dialog, because no interactive preview will be built.
- If the difference has to be seen to be judged (color, motion, spacing), make "build a preview of the options and let the user choose" the first step of the plan, and implement the rest only after they choose.

### The plan

Keep the plan short enough to read in about a minute:

```
## Plan: <task in a few words>

**Your choices**
- <decision>: <what the user chose>

**Defaults I applied**
- <decision>: <default> (<short reason>)

**Files**
- `path/to/file`: <what changes>
- `path/to/new-file` (new): <purpose>

**Verification**
- <how you will confirm it works: tests to run or add, commands, manual checks>
```

Leave out any section that would be empty, except Verification.

### Approval

Stop after presenting the plan. Don't edit project files, run commands that change state, or start implementing until the user explicitly approves, with something like "go", "approved" or "looks good, do it". Answers to your questions don't count as approval of the plan.

If the user approves and asks for a small change in the same message, such as a rename or leaving one file out, that is approval. Apply the change, say in one line what you changed, and start. If the user asks for changes without approving, or the change is material, such as a different approach, a wider scope or a reversed decision, revise the plan and wait again.

After approval, implement the plan as approved. If you find during implementation that it needs a material change, such as different files, an approach the user didn't choose, or a wider scope, stop, explain what you found, and get approval for the revised plan before continuing. Details within the approved approach don't need a new approval.

If the session is already in plan mode, write the plan in the format above, not a longer one, and submit it with ExitPlanMode instead of only printing it, because the user's approval in that dialog is the explicit approval. Don't switch into plan mode yourself.

### Report, then finish

When the implementation is done, report against the plan. For each item under Verification, say whether it passed, failed or was not run, and include the failing output when something failed. List anything that ended up different from the approved plan. Keep the report as short as the plan.

This skill covers the one task it was invoked for and ends with that report. Treat later requests in the session as ordinary requests, without the questions, plan and approval steps, unless the user invokes the skill again.

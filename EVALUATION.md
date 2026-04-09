# Evaluation Plan

## Question

Does `/onboard` reduce the cost of context establishment in Claude Code conversations?

## Hypothesis

Conversations using a cached onboard summary will spend fewer tokens and fewer tool calls on codebase discovery before reaching productive work, compared to conversations starting cold.

## Variables

- **Independent:** with onboard vs. without onboard
- **Dependent:** tokens consumed, tool calls before first meaningful action, accuracy of project understanding
- **Controlled:** same project, same tasks, same model, same conversation structure

## Method

Run paired conversations across a set of real tasks. Each task is performed twice: once cold (no onboard), once with `/onboard`. Record metrics for each.

### Task Set

Use a mix of task types to avoid biasing toward scenarios where one approach wins by default:

1. **Bug fix** -- "This function is producing wrong output, find and fix it"
2. **Feature question** -- "How does module X work?"
3. **Exploration** -- "Where does Y happen in this codebase?"
4. **Cross-module change** -- "Update the shared client and all callers"

Run each task type against at least two different projects of varying size (small: <10 files, medium: 10-50 files, large: 50+ files).

### Metrics

Track the following per conversation:

| Metric | Description |
|---|---|
| Tokens to first meaningful action | Total tokens consumed before Claude performs a productive action (edit, answer, fix) rather than discovery |
| Discovery tool calls | Count of Read, Glob, and Grep calls used to understand the project before productive work begins |
| Total session tokens | Total tokens for the entire conversation |
| Correct architecture references | Whether Claude correctly identifies relevant files, entry points, and module relationships without being told (yes/no) |
| Cache state | full scan, delta update, or cache hit (onboard conversations only) |
| Onboard mode | key-files or read-all (onboard conversations only) |
| Wall clock time | Time from first message to first productive action |

### Procedure

1. Select a project and task from the matrix above.
2. **Cold run:** Start a fresh Claude Code conversation. Issue the task prompt with no prior context. Record all metrics.
3. **Onboard run (key-files):** Start a fresh Claude Code conversation. Run `/onboard` first, then issue the same task prompt. Record all metrics.
4. **Onboard run (read-all):** Start a fresh Claude Code conversation. Run `/onboard --read-all` first, then issue the same task prompt. Record all metrics.
5. Repeat for each task/project combination.
6. Ensure conversations use the same model and no other context-loading mechanisms (no CLAUDE.md project notes, no manual context injection).

### Controls

- Do not reuse a conversation for multiple test runs.
- Clear any existing onboard cache before cold runs to prevent bleed-through.
- Use the same prompt phrasing for both runs of each task.
- Run cold and onboard conversations in randomized order to avoid learning effects from the person running the test.

## Analysis

Compare paired results across each metric. Key questions:

1. **Discovery cost:** How many fewer tool calls and tokens does the onboard run spend on discovery?
2. **Accuracy:** Does the onboard run reference correct files and architecture more reliably?
3. **Scaling:** Does the benefit increase with project size?
4. **Cache value:** What is the token cost of a full scan vs. a delta update vs. loading from cache?
5. **Mode comparison:** Does key-files mode provide sufficient context for most tasks, or does read-all produce meaningfully better results?

## Reporting

Record results in a table per task/project pair:

| Project | Task Type | Metric | Cold | Onboard (key-files) | Onboard (read-all) | Delta |
|---|---|---|---|---|---|---|
| project-a | bug fix | tokens to first action | | | | |
| project-a | bug fix | discovery tool calls | | | | |
| ... | ... | ... | ... | ... | ... | ... |

Summarize findings with aggregate comparisons and note any task types or project sizes where `/onboard` showed no measurable benefit. Pay particular attention to whether key-files mode is sufficient for each task type or whether read-all is needed for certain categories.

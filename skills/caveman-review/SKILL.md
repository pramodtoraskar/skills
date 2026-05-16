---
name: caveman-review
description: >-
  Ultra-compressed code review comments that cut noise while preserving actionable signal. Each comment is one line: location, problem, fix. Use when user says "review this PR", "code review", "review the diff", "/review", or "/caveman-review". Do NOT use for security findings, architectural discussions, or onboarding new developers where full explanations are needed.
compatibility: Cursor IDE and Claude Code
metadata:
  agentskills-spec: "https://agentskills.io/specification"
  version: "0.1.0"
---

# Caveman Code Review

## When to use

- User requests PR review with phrases like "review this PR", "code review", "review the diff", "/review"
- User explicitly invokes "/caveman-review" command
- Auto-triggers when reviewing pull requests or diffs
- User wants terse, actionable feedback without conversational fluff
- Multiple small issues need rapid identification across a large diff

## Procedure

1. **Scan the diff systematically** - Review changed lines in file order, noting line numbers for issues
2. **Identify actionable problems** - Focus on bugs, risks, and style issues that need specific fixes
3. **Format each finding as one line** using pattern: `L<line>: <severity> <problem>. <fix>.`
4. **Add severity prefixes when mixed severity present**:
   - 🔴 bug: for broken behavior that will cause incidents
   - 🟡 risk: for fragile code (races, missing null checks, swallowed errors)
   - 🔵 nit: for style, naming, micro-optimizations
   - ❓ q: for genuine questions, not suggestions
5. **Use multi-file format when reviewing multiple files**: `<file>:L<line>: <problem>. <fix>.`
6. **Drop terse mode immediately** for security findings (CVE-class), architectural disagreements, or onboarding contexts - write full paragraphs then resume terse format
7. **Include exact symbol names in backticks** and concrete fixes, not vague suggestions

## Defaults

- Default severity prefix: none (implies bug-level when mixed with prefixed items)
- Default format: `L<line>: <problem>. <fix>.`
- Multi-file format: `<filename>:L<line>: <problem>. <fix>.`
- Line number reference: actual line numbers from the diff, not relative positions
- Concrete fix language: imperative mood ("Add guard", "Extract function", "Wrap in retry")

## Anti-patterns

- **Never** use throat-clearing phrases: "I noticed that...", "It seems like...", "You might want to consider..."
- **Never** hedge with "perhaps", "maybe", "I think" - use ❓ q: prefix if genuinely unsure
- **Never** restate what the code does - reviewer can read the diff
- **Never** use checkbox task lists as primary review format
- **Never** apply terse format to security vulnerabilities or architectural concerns
- **Never** omit line numbers or use vague location references
- **Avoid** "Great work!" or "Looks good overall but..." unless said once at the top

## Evaluation suite

See `evals/README.md` for test cases covering terse formatting, severity classification, and appropriate exceptions to caveman mode. Run evaluations with `bash cli/run-evals.sh caveman-review` from the skills-hub root.

The evaluation suite in `evals/evals.json` tests format compliance, line number accuracy, and proper escalation to full explanations for security/architecture issues.

## Additional detail

See `references/formatting-guide.md` for complete formatting examples and `references/severity-guide.md` for detailed severity classification criteria.
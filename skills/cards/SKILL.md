---
description: "Draw a commit card, review changes against 128 best practices, quiz yourself, or browse decks. Covers OWASP, SOLID, Clean Code, Clean Architecture, Effective Java, Design Patterns, 12-Factor App, and more."
---

# CodeDeck — Development Best Practices

Given: $ARGUMENTS

Parse the arguments to determine which mode to run. The first word is the command. Remaining words are arguments to that command.

## Modes

### Mode 0 — Help (`help`)
Show the user how to use this skill:

```
## /codedeck:cards — Development Best Practices

128 best practice cards across 12 decks. One command for consistent code reviews.

### Commands

  /codedeck:cards                     Draw a random card
  /codedeck:cards review              Review changes against ALL 128 cards
  /codedeck:cards review <deck>       Review changes against a specific deck
  /codedeck:cards quiz                Quiz yourself — get a card, try to answer before reveal
  /codedeck:cards list                List all cards grouped by deck
  /codedeck:cards decks              List decks with card counts
  /codedeck:cards <name>              Show a specific card (fuzzy match)
  /codedeck:cards help                Show this help

### Deck names (for filtered reviews)

  owasp, solid, clean-code, clean-architecture, effective-java,
  testing, api-design, design-patterns, twelve-factor, git, performance, xdev

### Examples

  /codedeck:cards review owasp        Only check OWASP Top 10 rules
  /codedeck:cards review solid        Only check SOLID principles
  /codedeck:cards quiz clean-code     Quiz from Clean Code deck only
  /codedeck:cards injection           Show the "Injection" card
```

Also list the actual deck names and card counts by reading the deck files.

### Mode 1 — Draw a random card (no arguments, or `draw`)
1. Read ALL deck files from the `decks/` directory in this plugin's root (skip `_template.md`).
2. Pick ONE card at random from any deck.
3. Present it in card format (see Output Format below).
4. Keep it concise and actionable — this is a quick tip, not a lecture.

### Mode 2 — Review changes (`review` or `review <deck-name>`)
1. Run `git diff --cached` (staged) and `git diff` (unstaged) to see what's about to be committed.
2. If there are no changes, tell the user and suggest they stage some changes first.
3. If a deck name is specified after `review`, only read that deck file. Otherwise read ALL deck files from `decks/` (skip `_template.md`).
4. Analyze the diff against every card in the loaded deck(s).
5. Pick the **3-5 most relevant cards** based on the actual changes.
6. For each relevant card, use the Review Format (see Output Format below).
7. End with a clear verdict:
   - **Ready to commit** — if no violations found
   - **Suggested improvements** — list specific actions, ordered by severity
8. If everything looks good, say so! Celebrate clean code.

### Mode 3 — Show a specific card (`<card-name>`)
If the argument doesn't match any command keyword (help, review, quiz, list, decks, draw), treat it as a card name search.
1. Read ALL deck files (skip `_template.md`).
2. Find the card whose title best matches the argument (case-insensitive fuzzy match).
3. If multiple cards match, show the best match and list other possible matches.
4. If no match found, suggest similar card names.
5. Display in card format.

### Mode 4 — List all cards (`list`)
1. Read ALL deck files (skip `_template.md`).
2. Show a grouped list: deck name as heading, then numbered card titles under each.
3. Show total count at the end.

### Mode 5 — List decks (`decks`)
1. Read ALL deck files (skip `_template.md`).
2. Show a table: deck name, card count, source.
3. Show total count at the end.

### Mode 6 — Quiz mode (`quiz` or `quiz <deck-name>`)
1. If a deck name is specified, read only that deck. Otherwise read ALL decks (skip `_template.md`).
2. Pick ONE card at random.
3. Present only the **card title** and the **Commit check** question. Do NOT show the advice or examples yet.
4. Ask the user to think about their answer.
5. Wait for the user to respond.
6. After the user responds, reveal the full card (advice + bad/good examples) and discuss how their answer compares to the principle.

## How to find deck files

Deck files are markdown files in the `decks/` directory relative to this skill's plugin root. Each `.md` file is one deck (e.g., `xdev-commit-cards.md`, `clean-code.md`). Skip `_template.md`.

### Deck name matching

When the user specifies a deck name, match it loosely against filenames:
- `owasp` matches `owasp-top-10.md`
- `solid` matches `solid-principles.md`
- `clean-code` matches `clean-code.md`
- `clean-architecture` matches `clean-architecture.md`
- `java` or `effective-java` matches `effective-java.md`
- `testing` matches `testing-best-practices.md`
- `api` or `api-design` matches `api-design.md`
- `patterns` or `design-patterns` matches `design-patterns.md`
- `twelve-factor` or `12-factor` matches `twelve-factor-app.md`
- `git` matches `git-best-practices.md`
- `performance` or `perf` matches `performance.md`
- `xdev` matches `xdev-commit-cards.md`

## Output format

### Card format (for draw, specific card, quiz reveal)

```
## [Deck Name] Card Title

**Advice:** Core principle in 1-2 sentences.

**Bad:**
<code example>

**Good:**
<code example>

**Before you commit, ask:** The commit-time check question.
```

### Review format (for each finding in review mode)

```
### [Deck Name] Card Title — FOLLOWS / NEEDS ATTENTION

**In your code:**
> quoted code from the diff

**The principle says:** brief summary of the card's advice

**Suggestion:** concrete fix with code (only if NEEDS ATTENTION)
```

### Review verdict format

```
## Verdict: Ready to commit / Needs attention

**Score: X/Y cards passed**

### Actions needed (if any):
1. [High] Description — card name
2. [Medium] Description — card name
3. [Low] Description — card name
```

# CodeDeck — Best Practice Cards for Claude Code

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Cards](https://img.shields.io/badge/Cards-128-green.svg)](#available-decks)
[![Decks](https://img.shields.io/badge/Decks-12-orange.svg)](#available-decks)
[![Claude Code Plugin](https://img.shields.io/badge/Claude%20Code-Plugin-blueviolet.svg)](https://claude.ai/code)

**128 development best practice cards across 12 decks.** Review your code, draw daily tips, quiz yourself — all inside Claude Code.

Stop telling Claude to "follow best practices." Start running structured, reproducible reviews with concrete examples.

---

## Quick Start

Add to `.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "codedeck-marketplace": {
      "source": {
        "source": "git",
        "url": "https://github.com/hhovhann/codedeck.git",
        "ref": "master"
      },
      "autoUpdate": true
    }
  },
  "enabledPlugins": {
    "codedeck@codedeck-marketplace": true
  }
}
```

Restart Claude Code. Done.

```
/codedeck:cards              Draw a random card
/codedeck:cards review       Review your changes against all 128 cards
/codedeck:cards review owasp Review against OWASP Top 10 only
/codedeck:cards quiz         Quiz yourself on a random principle
/codedeck:cards DRY          Look up a specific card
```

---

## Why this exists

| "Follow best practices" (vague) | `/codedeck:cards review` (structured) |
|---|---|
| Claude picks random things to mention | 128 specific cards, same checks every time |
| No examples, just general advice | Concrete bad/good code for each principle |
| You have to remember to ask | One command before every commit |
| Can't share or standardize across team | Plugin — everyone gets the same checks |
| Not extensible | Drop a `.md` file in `decks/` to add your own |
| Different answers each time | Consistent, reproducible reviews |

---

## Commands

| Command | What it does |
|---|---|
| `/codedeck:cards` | Draw a random card — quick daily tip |
| `/codedeck:cards review` | Review staged/unstaged changes against ALL 128 cards |
| `/codedeck:cards review <deck>` | Review against a specific deck (e.g., `owasp`, `solid`) |
| `/codedeck:cards quiz` | Quiz mode — see the question, think, then reveal the answer |
| `/codedeck:cards quiz <deck>` | Quiz from a specific deck |
| `/codedeck:cards list` | List all cards grouped by deck |
| `/codedeck:cards decks` | List decks with card counts |
| `/codedeck:cards <name>` | Show a specific card (fuzzy match) |
| `/codedeck:cards help` | Show usage guide |

### Deck filter names

`owasp` `solid` `clean-code` `clean-architecture` `effective-java` `testing` `api-design` `design-patterns` `twelve-factor` `git` `performance` `xdev`

---

## Workflows

### Pre-commit review

The primary use case. Before you commit:

```
/codedeck:cards review
```

Claude reads your `git diff`, checks it against all 128 principles, picks the 3-5 most relevant, and gives you a verdict with concrete fix suggestions.

For focused reviews:

```
/codedeck:cards review owasp       # security-focused review
/codedeck:cards review solid       # architecture-focused review
/codedeck:cards review clean-code  # readability-focused review
```

### Daily tip

Run `/codedeck:cards` once a day to learn a new principle with examples.

### Quiz yourself

```
/codedeck:cards quiz
```

See a principle name and a question. Think about your answer. Then Claude reveals the full card with bad/good examples and discusses your response.

### Team standardization

Add CodeDeck to your project's `.claude/settings.json`. Every developer on the team gets the same 128 checks. No more inconsistent reviews.

---

## Available decks

| Deck | Cards | Source |
|------|-------|--------|
| XDEV Commit Cards | 32 | [XDEV Software](https://commit-cards.xdev.software/latest/) |
| Clean Code | 15 | Robert C. Martin |
| Effective Java | 12 | Joshua Bloch |
| Twelve-Factor App | 12 | [12factor.net](https://12factor.net) |
| OWASP Top 10 | 10 | [OWASP Foundation](https://owasp.org/Top10/) |
| Clean Architecture | 10 | Robert C. Martin |
| Testing Best Practices | 10 | Community practices |
| API Design | 8 | REST design principles |
| Design Patterns | 8 | Gang of Four |
| Git Best Practices | 8 | Community practices |
| Performance | 8 | Community practices |
| SOLID Principles | 5 | Robert C. Martin |
| **Total** | **128** | |

---

## Add your own deck

1. Create a `.md` file in `decks/`
2. Follow the format in [`decks/_template.md`](decks/_template.md)
3. Each `## Heading` is one card

```markdown
# My Team Standards

Our internal coding standards.

---

## Always Use Logging Framework
**Advice:** Never use System.out.println. Always use SLF4J.
**Bad:** `System.out.println("Error: " + e.getMessage());`
**Good:** `log.error("Failed to process order", e);`
**Commit check:** Did I add any System.out.println calls?
```

Your custom deck works immediately — no config needed.

---

## Testing locally (for contributors)

```bash
claude --plugin-dir /path/to/codedeck
```

Then try `/codedeck:cards`, `/codedeck:cards review`, etc. Run `/reload-plugins` after editing files.

---

## Contributing

PRs welcome — especially new decks! Follow the format in `decks/_template.md`.

Ideas for new decks:
- Accessibility (WCAG)
- Kubernetes / Docker
- Database design
- React / frontend patterns
- Python-specific (PEP 8, etc.)
- Rust safety patterns

---

## Credits and Attribution

All card content consists of **original educational summaries with original code examples**. No copyrighted text is reproduced verbatim.

- **XDEV Commit Cards** — Card topics inspired by [XDEV Software's Commit Cards](https://commit-cards.xdev.software/latest/) (copyright XDEV Software GmbH). All text and code examples are original. Also available as a [physical card set](https://xdev.software/en/about-us/xdev-commit-cards).
- **OWASP Top 10** — Based on [OWASP Top 10 (2021)](https://owasp.org/Top10/) by [OWASP Foundation](https://owasp.org), licensed under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/). OWASP deck content shared under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/).
- **Twelve-Factor App** — Based on [12factor.net](https://12factor.net) by Adam Wiggins, [CC BY 2.0](https://creativecommons.org/licenses/by/2.0/).
- **SOLID / Clean Code / Clean Architecture** — Educational summaries inspired by Robert C. Martin's widely-taught principles.
- **Effective Java** — Educational summaries inspired by Joshua Bloch's "Effective Java" (3rd ed.).
- **Design Patterns** — Based on Gang of Four patterns (Gamma, Helm, Johnson, Vlissides, 1994).
- **Testing, API Design, Git, Performance** — Community-standard practices.

Created by [Hayk Hovhannisyan](https://hhovhann.com).

## License

MIT — see [LICENSE](LICENSE).

**Exceptions:**
- `decks/owasp-top-10.md` — [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) (derived from OWASP Foundation material).
- `decks/twelve-factor-app.md` — Twelve-Factor App methodology is [CC BY 2.0](https://creativecommons.org/licenses/by/2.0/) by Adam Wiggins.
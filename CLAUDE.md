# CodeDeck — Project Guidelines

## Project structure

- `decks/` — Markdown files, one per deck. Each `## Heading` is a card.
- `skills/cards/SKILL.md` — The `/codedeck:cards` skill definition
- `.claude-plugin/plugin.json` — Plugin metadata
- `.claude/settings.json` — Project-level permission deny rules
- `marketplace.json` — Marketplace registry entry
- `decks/_template.md` — Template for creating new decks

## Contributing rules

- Follow the card format in `decks/_template.md`
- Each card must have: **Advice**, **Bad**, **Good**, and **Commit check** sections
- All code examples must be original — never copy from source materials
- Include proper attribution in the deck header when inspired by external sources
- Keep cards concise and actionable
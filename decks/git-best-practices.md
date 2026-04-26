# Git Best Practices

Principles for clean, collaborative version control that makes history useful and code review efficient.

---

## 1. Atomic Commits
**Advice:** Each commit should represent one logical change. If you can describe it with "and" — it's two commits. Atomic commits are easier to review, revert, cherry-pick, and bisect.
**Bad:** "Fix login bug, update CSS, add new endpoint, refactor utils" — all in one commit.
**Good:** Separate commits: "Fix null check in LoginService", "Add /api/v1/reports endpoint", "Extract date formatting to DateUtils".
**Commit check:** Does this commit contain exactly one logical change?

## 2. Write Good Commit Messages
**Advice:** First line: imperative mood, under 72 characters, summarizes WHAT. Body: explains WHY the change was made, not how (the diff shows how). Reference ticket numbers.
**Bad:** `fix stuff`, `wip`, `changes`, `update`, `asdf`
**Good:**
```
Fix NPE when user has no email address

Users created via SSO may not have an email. The notification
service now skips email delivery for these users instead of crashing.

Fixes: IRT-1234
```
**Commit check:** Does my commit message explain what changed and why? Would someone understand it in 6 months?

## 3. Don't Commit Generated Files
**Advice:** Build artifacts, compiled code, dependency directories (node_modules), and generated files should be in `.gitignore`. Only commit source files and configuration.
**Bad:** Committing `build/`, `target/`, `node_modules/`, `.class` files, or minified JS.
**Good:** Proper `.gitignore` for your stack. Only source code, config, and lock files committed.
**Commit check:** Am I committing files that can be regenerated from source?

## 4. Don't Commit Secrets
**Advice:** API keys, passwords, tokens, certificates, and private keys must never be committed. Once in git history, they're there forever — even after deletion. Use environment variables or secret managers.
**Bad:** `.env` files, `application-prod.yml` with real credentials, `id_rsa` in the repo.
**Good:** `.env` in `.gitignore`. Secrets injected via environment variables or vault. Example files (`application.yml.example`) with placeholders.
**Commit check:** `git diff --cached` — do I see any strings that look like secrets?

## 5. Small, Frequent Pull Requests
**Advice:** Large PRs are hard to review and tend to get rubber-stamped. Small PRs (under 400 lines) get meaningful feedback. Break large features into incremental, reviewable chunks.
**Bad:** 2000-line PR that touches 50 files, mixes refactoring with new features.
**Good:** PR 1: refactor (no behavior change), PR 2: add data model, PR 3: add endpoint, PR 4: add UI.
**Commit check:** Is my PR under 400 lines? If not, can I split it?

## 6. Keep Branches Short-Lived
**Advice:** Feature branches should live for days, not weeks. Long-lived branches diverge from main, causing painful merge conflicts. Merge early and often.
**Bad:** A feature branch that's been open for 3 weeks with 47 commits and growing conflicts with main.
**Good:** Branch off main, work for 1-3 days, open a PR, merge. Use feature flags for incomplete features.
**Commit check:** How old is my branch? Should I merge what I have and continue in a new branch?

## 7. Rebase Before Merge (When Appropriate)
**Advice:** Rebase your feature branch onto main before merging to maintain a clean, linear history. But never rebase shared/public branches.
**Bad:** Merging a stale branch creates a messy history with interleaved commits from multiple branches.
**Good:** `git rebase main` before opening a PR. Resolve conflicts once. Clean, linear history after merge.
**Commit check:** Is my branch up to date with main? Would a rebase clean up the history?

## 8. Use Meaningful Branch Names
**Advice:** Branch names should communicate the purpose. Include ticket number, type, and brief description.
**Bad:** `fix`, `test`, `johns-branch`, `temp`, `new-feature`
**Good:** `IRT-1234_fix_login_null_pointer`, `feature/api-v2-pagination`, `bugfix/order-total-rounding`
**Commit check:** Does my branch name tell someone what it's about without looking at the code?
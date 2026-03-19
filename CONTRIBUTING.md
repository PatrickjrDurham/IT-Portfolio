# Contributing & Commit Policy

This document defines the commit conventions used across this portfolio repository.
All commits must be written in **English**.

---

## Commit Format

This project follows the [Conventional Commits](https://www.conventionalcommits.org/) specification.
 
```
<type>(<scope>): <short description>
```

- **type** — nature of the change (see table below)
- **scope** — lab or section affected, in parentheses 
- **description** — short summary, imperative mood, lowercase, no period at the end

**Examples:**
```
docs(sccm): add English README with architecture diagram
fix(sccm): correct ODBC version warning in prerequisites
feat(teleport): add bastion lab installation guide
add(wazuh): attach brute force detection screenshots
chore: restructure repo folders per lab convention
```

---

## Commit Types

| Type | Use case |
|---|---|
| `feat` | New lab or new major section added |
| `docs` | Documentation added or updated (README, guides) |
| `fix` | Correction of an error or inaccuracy in the content |
| `add` | New files added (screenshots, assets, config files) |
| `refactor` | Content rewritten or restructured without changing meaning |
| `chore` | Maintenance tasks (renaming, moving files, repo structure) |
| `remove` | Files or outdated content deleted |

---

## Scopes

Use the lab or section name as the scope:

| Scope | Description |
|---|---|
| `sccm` | SCCM / MECM installation lab |
| `teleport` | Teleport bastion lab |
| `wazuh` | Wazuh SIEM lab |
| `exchange` | Exchange + DKIM/SPF/DMARC lab |
| `portfolio` | Root-level files (README, CONTRIBUTING, etc.) |

---

## Rules

**✅ Do**
- Use the imperative mood: `add`, `fix`, `update`, `remove`, `translate`
- Keep the description under 72 characters
- Reference the affected lab in the scope when relevant
- Make one commit per logical change

**❌ Don't**
- `git commit -m "update"` — too vague
- `git commit -m "fixed stuff"` — past tense, no context
- `git commit -m "WIP"` or `"FINAL"` — not descriptive
- Mix multiple unrelated changes in a single commit

---

## Example Commit History

```
chore: init sccm-lab folder structure
add(sccm): upload installation doc with screenshots
docs(sccm): translate installation guide to English
docs(sccm): add English README with architecture diagram
fix(sccm): add ODBC driver version warning in prerequisites
feat(teleport): add bastion lab installation guide
docs(teleport): add RBAC and session replay use case scenarios
feat(wazuh): add SIEM lab installation guide
docs(wazuh): add brute force and FIM detection scenarios
add(wazuh): attach Teleport log integration screenshots
```

---

> This repository is part of a technical portfolio built during TSSR training, oriented toward cybersecurity.

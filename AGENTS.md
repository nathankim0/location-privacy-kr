# AGENTS.md

Instructions for any coding agent (Claude Code, Codex, others) working in this repository. `CLAUDE.md` is a symlink to this file; edit `AGENTS.md` only.

## What this repo is

An agent skill that audits and fixes a project's compliance with Korea's Location Information Act (위치정보법) and Personal Information Protection Act (개인정보보호법). It is distributed as a Claude Code plugin (marketplace in this repo) and as a plain skill folder that Claude Code, Codex, and other Agent Skills tools can load.

```text
skills/location-privacy-kr/SKILL.md          the workflow (keep under 200 lines)
skills/location-privacy-kr/references/       checklists and patterns, loaded on demand
  applicability.md                           is it location data, which operator type
  code-audit.md                              building the location data map from code
  terms-checklist.md                         location-based service terms T1-T22
  privacy-policy-checklist.md                privacy policy P1-P20 and location items L1-L7
  consent-and-rights.md                      consent, withdrawal, pause, deletion, notice, minors
  safeguards.md                              records, access logs, controls, inspection prep
  penalties.md                               severity mapping and sanctions
  fix-patterns.md                            code fixes and document skeletons
  report-template.md                         report format
  conflicts.md                               rulings where source skills disagreed
.claude-plugin/                              plugin and marketplace manifests
ATTRIBUTIONS.md                              source skills and legal sources
```

## Rules for every change

1. **Commit and push after every change.** One logical change per commit. Push to `main` right after committing. Do not leave work uncommitted at the end of a task.
2. **Release only for distribution changes.** A change is a distribution change when it alters what users install or how the skill behaves: `skills/`, `.claude-plugin/`, or a changed install command. For these:
   - bump the version one step in `.claude-plugin/plugin.json` (`version`) and `.claude-plugin/marketplace.json` (`metadata.version`): patch for fixes and small tweaks, minor for new features, major for breaking changes
   - move the `Unreleased` notes in `CHANGELOG.md` under the new version
   - commit `Release vX.Y.Z`, push, then tag and release:
     ```bash
     git tag -a vX.Y.Z -m "vX.Y.Z"
     git push origin vX.Y.Z
     gh release create vX.Y.Z --title "vX.Y.Z" --notes-file <notes>
     ```
3. **Docs-only changes do not release.** README, AGENTS.md, images, comments, and typo fixes are committed and pushed with no version bump, tag, or release.
4. **Keep READMEs in sync.** `README.md` is Korean (the default shown on GitHub) and `README.en.md` is English. Any change to one gets the same change in the other.
5. **Validate before committing** a distribution change: `claude plugin validate .` must pass.
6. **Never commit secrets or private information**: no tokens, employer or internal names, personal emails, or home paths. Audit reports of real projects never go into this repo.

## Project rules

- No copied text from source skills or from the training material the checklists are based on. Paraphrase statutes; cite article numbers.
- Every legal statement either cites an article or is marked "현행 조문 확인 필요". Do not state fine amounts or effective dates from memory.
- Examples are real (from a public open-source target) or clearly marked as placeholders.
- Nothing is taken from repos with no license beyond general ideas.
- `SKILL.md` stays under 200 lines; detail goes into `references/`.
- When a rule changes, check `references/conflicts.md` and update the matching ruling.

## Commit messages

Imperative, short subject. Example: `Add beacon and Wi-Fi scan patterns to code audit`.

# AGENTS.md

## Repo overview

Pure documentation repository for .NET memory performance analysis (by Maoni Stephens). No build, test, or code.

## Files

| File | Purpose |
|------|---------|
| `NETMemoryPerformanceAnalysis.md` | **Primary working file** — Chinese translation with bilingual technical terms |
| `.NETMemoryPerformanceAnalysis.md` | Original English version (dot-prefixed, unmodified) |
| `.NETMemoryPerformanceAnalysis.zh-CN.md` | Older Chinese translation (dot-prefixed, unmodified) |
| `images/` | 35 `.jpg` screenshots referenced by the markdown files |

## Image paths

All images live in `images/`. References use `./images/filename.jpg` format. If adding or moving images, update all `.md` files that reference them.

## Critical style rule

**Professional/technical terms MUST remain in English.** Never translate these terms to Chinese:

Memory, Process, Heap, Allocation, Object, Bookkeeping, Generation, Pause, Stack Views, Ephemeral GC, Full Blocking GC, Thread Suspension, Pinning, Finalizers

General narrative text (section intros, explanations) should be in Chinese. FAQ question text may mix both.

## Heading conventions

```
# 一、Section title          (H1 — Chinese numerals 一～十一)
## 1. Subsection title        (H2 — Arabic numerals)
### I. Sub-subsection title   (H3 — Roman numerals)
#### a. Sub-sub-subsection    (H4 — lowercase letters)
##### a. Deeper level         (H5 — lowercase letters)
###### Label text             (H6 — inline labels, no number)
```

- No `**bold**` in headings
- No `-` dash prefix on headings (remove `- ####` → `####`)
- No level skipping (H2→H4 must be fixed to H2→H3→H4)
- Contents/TOC section was intentionally deleted; do not re-add

## File origins

The primary file `NETMemoryPerformanceAnalysis.md` is the actively maintained Chinese version. The dot-prefixed files are historical snapshots and should not be modified unless explicitly requested.

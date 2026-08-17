This repo is a rewrite of a monolith `old SKILL.md` into separate files. Goal: minimize token usage, increase quality, revisit and refine rules. Hard requirements:

- All rules must be beneficial. If a rule adds a constraint for no benefit, it must be cut.
- Aggressive token reduction. All new skill text must be as compact and dense as possible
- Compaction must not reduce skill efficiency. It must be as clear to an agent as possible to minimize ignoring rules or misinterpretation. Human readability is important too for future maintainability
- Maintain a strict decision making framework. Terms across files must be repeated, no synonyms.
- Strict simplified technical English in all prose and skill text. No phrasal verbs. One name for one thing, one word - one meaning. No marketing bullshit talk. American spelling. Active voice. Use verbs for actions. No stacked auxiliaries. No -ing for main verb. Sentence length hard cap at 25 words. No semicolons in prose. One topic per paragraph

Files:

- `old SKILL.md` - reference. Must not be edited.
- `SKILL.md` - core and navigation. Mostly complete. Missing: Agent workflows and general usage guidance.
- `naming.md` - Complete.
- `test-names.md` - A list of test names to test against `naming.md`. Internal use only
- `what-to-test.md` - Populated but redundant for usecase. Needs review and cuts.
- Rest: not touched.

Update this list as you make changes. If more than a single sentence, do three iterative passes of rigorous self-review to compact changes.

This repo is a rewrite of a monolith `old SKILL.md` into separate files. Goal: minimize token usage, increase quality, fine-tune rules. Hard requirements:

- All rules must be benefitial. If a rule adds a constraint for no benefit, it must be cut.
- Aggressive token reducation. All new skill text must be as compact and dense as possible
- Compaction must not reduce skill efficiency. It must be as clear to an agent as possible to minimize ignoring rules or misinterpretation. Human readability is important too for future maintainability
- Strict simplified technical English in all prose and skill text. No phrasal verbs. One name for one thing, one word - one meaning. No marketing bullshit talk. American spelling. Active voice. Use verbs for actions. No stacked auxiliaries. No -ing for main verb. Sentence length hard cap at 25 words. No semicolons in prose. One topic per paragrpah.

Files:

- `old SKILL.md` - reference. Must not be edited
- `SKILL.md` - core and navigation
- `naming.md` - naming rules

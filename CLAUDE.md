# Working on world-graph

## Keep everything consistent in the same change

Every code change ships with the matching doc and config changes in the same commit. Never leave them for later.

- **README matches the code.** If a change alters the pipeline stages, model choices, interfaces, commands, or status, update the README's Pipeline, What's new, and Status sections in the same commit.
- **One source of truth for the scene graph schema.** Code, examples, tests, and docs all follow the schema definition. When the schema changes, update every consumer and fixture in the same change.
- **Names stay in sync.** When renaming a module, function, CLI flag, or config key, update every reference, including docs and examples. Before finishing, search the repo for the old name.
- **No claims without code.** The README and docs report only numbers (PSNR, object-position error, speed) that the repo's evaluation code actually produces. Mark planned features as planned.
- **Small-model constraint.** Every model in the default pipeline must stay at or under 33B parameters. If a change adds or swaps a model, record its size in the README.
- **Related work.** Keep the verified/unverified markers accurate. Don't promote an abstract-only entry to verified without reading the paper or code.

## Before committing

1. Check the diff: does any doc, example, test, or config now describe something that no longer matches?
2. Search for stale references to anything renamed or removed.
3. Run the tests and linters once they exist, and fix failures rather than skipping them.

## Git

- Long paths can break git on Windows here: use `git -c core.longpaths=true ...`.
- End commit messages with the co-author line the session specifies.

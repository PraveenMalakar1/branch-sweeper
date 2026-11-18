# branch-sweeper (Claude Code plugin)

Clean up local git branches already merged into production, with AI summaries, two-step approval, and zero impact on origin.

`/branch-sweeper` is a Claude Code slash command. It finds the **local** git branches that are already merged into your production branch and deletes the ones you choose. It works in any git repository, on Windows, macOS and Linux.

## What it does

1. **Asks for your production branch** (e.g. `production`, `main`) and remembers it **per repository** in git config as `claude.productionBranch`. On the next run, that answer is offered first.
2. **Lists the local branches that are fully merged** into it. The production branch, your current branch and the remote default branch are never listed.
3. **Reports branches that look squash- or rebase-merged** (see below). They're listed for your information only and are never deleted.
4. **Lets you select several branches at once.**
5. **Confirmation 1:** for each selected branch it shows when it was merged into production, the merge commit, how many commits it had, and a two-line AI summary of what the branch did.
6. **Confirmation 2:** an explicit "Approve – delete locally" step.
7. **Deletes them locally** with one `git branch -d …`, then shows the restore command for each one.

It **never** touches the remote: no `git push --delete`, and no `-D` force delete.

## Install

Inside Claude Code:

```
/plugin marketplace add PraveenMalakar1/branch-sweeper
/plugin install branch-sweeper@branch-sweeper
```

To try a local checkout:

```
/plugin marketplace add D:/path/to/branch-sweeper
/plugin install branch-sweeper@branch-sweeper
```

## Use

Open Claude Code in any git repository and run:

```
/branch-sweeper
```

To change the remembered production branch:

```
git config --local claude.productionBranch <branch>
```

To forget it:

```
git config --local --unset claude.productionBranch
```

## Squash- and rebase-merged branches

Git doesn't count these branches as merged, so `git branch -d` refuses to delete them. branch-sweeper detects them and lists them with the evidence:

- **Rebase-merged / cherry-picked:** `git cherry <prod> <branch>` shows every commit already exists in production.
- **Squash-merged:** `git merge-tree --write-tree <prod> <branch>` produces exactly production's tree, so merging the branch would change nothing. This needs Git 2.38+.

These branches are **never deleted** by the plugin. If you're sure about one, delete it yourself with `git branch -D <name>`. A squash-merged branch whose lines were later changed again on production isn't detected.

## Limitations

- The production branch has to exist locally (or as `origin/<name>`). Run `git fetch` first if your local copy is out of date.
- This removes branch names, not data, so it frees almost no disk space. For that, use `git fetch --prune` and `git gc`.

## License

MIT. See [LICENSE](LICENSE).

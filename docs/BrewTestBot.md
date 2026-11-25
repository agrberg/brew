---
logo: https://brew.sh/assets/img/brewtestbot.png
image: https://brew.sh/assets/img/brewtestbot.png
redirect_from:
  - /Brew-Test-Bot
last_review_date: "2025-02-08"
---

# BrewTestBot

[`brew test-bot`](Manpage.md#test-bot-options-formula) is Homebrew's continuous-integration orchestration command, originally funded by [our Kickstarter in 2013](https://www.kickstarter.com/projects/homebrew/brew-test-bot).
Homebrew's GitHub Actions workflows run it on macOS and Linux runners to build bottles and test the lifecycle of changes to Homebrew and its taps.
The workflow definitions are the authoritative description of the current runner and job configuration.

## Steps

A full `brew test-bot` run does these steps in order:

1. Clean up before the run (`--only-cleanup-before`, which needs `--cleanup` except on GitHub Actions).
2. Check the local system setup (`--only-setup`).
3. Check the tap's files (`--only-tap-syntax`, described under [tap checks](#tap-checks)).
4. Detect which formulae the change tests, adds or deletes.
5. Install, test, audit and bottle those formulae (`--only-formulae`).
6. Test the formulae that depend on them (`--only-formulae-dependents`).
7. Clean up after the run (`--only-cleanup-after`, with the same `--cleanup` requirement).

Each `--only-` option runs just that step, so a workflow can show each step on its own line and run some of them only on pull requests.
The workflow that `brew tap-new` generates runs the first three steps, then the formulae step on pull requests.

### Tap checks

The `--only-tap-syntax` step runs static checks on the tap's files and builds nothing.
It runs these in order:

- `brew typecheck`, for official taps whose files use Sorbet sigils.
- `brew style` on the tap.
- `brew readall --aliases --os=all --arch=all`, which loads every formula and cask on every operating system and architecture, and checks the aliases.
- `brew audit --except=installed --tap=<tap>`.

The last two run only when the tap has formulae or casks, so a tap with only external commands gets `brew style`.
Adding `--stable` runs only `brew readall`, which is the check needed on a stable `brew`.

## Pull requests

The bot automatically builds pull requests and updates their status depending on the result of the job.

For example, a job which has been queued but not yet completed will have a section in the pull request that looks like this:

![Triggered Pull Request](assets/img/docs/brew-test-bot-triggered-pr.png)

---

A failed build looks like this:

![Failed Pull Request](assets/img/docs/brew-test-bot-failed-pr.png)

---

A passed build that's been approved to merge looks like this:

![Passed Pull Request](assets/img/docs/brew-test-bot-passed-pr.png)

---

On failed or passed builds you can click the "Details" link for each check to view its output in GitHub Actions.

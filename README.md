# GitHub OMP Worker

A small polling worker that turns GitHub issues and pull-request events into unattended OMP coding runs.

## How it works

Every five minutes, the worker uses the active `gh` account to find:

- Every new open issue in repositories owned by the account.
- Every new open issue assigned to the account.
- New events on open pull requests authored by the account.
- New events on third-party pull requests carrying the `omp-ready` label.

For accepted work, it finds the matching clone under the configured repository root or clones it, refuses to run on a dirty worktree, and starts OMP inside that repository. With `--open-tab`, each job runs visibly in a new iTerm tab while the watcher waits for its completion result. OMP reads the GitHub discussion, implements and verifies the change, commits it, and pushes it. An issue is closed only when its verified change reaches the default branch; otherwise OMP opens or updates a pull request containing `Closes #<issue>`.

The first normal run baselines existing open issues and unread notifications. Use `--replay-existing` to process that backlog. Processed work is recorded in `~/.local/state/github-omp/state.json`; OMP logs are stored beside it under `logs/`.

## Requirements

- Python 3.11+
- `git`
- Authenticated `gh`
- Authenticated `omp`
- iTerm2 when using `--open-tab`

## Run

```bash
# Preview current work without changing anything
./github_omp.py --dry-run --replay-existing

# Watch for new work every five minutes in headless mode
./github_omp.py --watch

# Watch every 10 seconds and open each OMP job in a new iTerm tab
./github_omp.py --watch --interval 10 --open-tab \
  --omp-executable ~/.bun/bin/omp \
  --model github-copilot/gpt-5.6-sol-1m

# Process existing work, then keep watching
./github_omp.py --watch --replay-existing

# Use a different interval and repository root
./github_omp.py --watch --interval 60 --root /path/to/repos
```

Stop watch mode with `Ctrl+C`.

## Security

Every qualifying issue is accepted regardless of author. Anyone who can open an issue in an owned public repository can trigger an unattended OMP run. The worker disables repository-provided OMP extensions and applies prompt and clean-worktree safeguards, but it is not an OS sandbox.

## Test

```bash
python3 -m unittest -v
```

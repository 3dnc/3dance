# Transfer repo to 3dnc

To move this repository to the [3dnc](https://github.com/3dnc) organization:

## From the web

1. Repo **Settings** → **General** → **Danger zone** → **Transfer ownership**.
2. New owner: `3dnc`.
3. Confirm by typing the repo name and confirm.

[GitHub: Transferring a repository](https://docs.github.com/en/repositories/creating-and-managing-repositories/transferring-a-repository)

## From the CLI

From the repo root, with [GitHub CLI](https://cli.github.com/) authenticated as a user with admin on the repo and create-repo permission in the 3dnc org:

```bash
gh api repos/:owner/:repo/transfer -X POST -f new_owner=3dnc
```

After transfer, the repo URL becomes `https://github.com/3dnc/3dance`. GitHub Pages (if enabled) will be at `https://3dnc.github.io/3dance/`. Update the local remote if needed:

```bash
git remote set-url origin https://github.com/3dnc/3dance.git
```

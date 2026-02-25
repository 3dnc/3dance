# GitHub Pages

This repo is set up to be served as a static site on GitHub Pages.

## Enable Pages

### From the web

1. GitHub → repo **Settings** → **Pages**.
2. **Source**: Deploy from a branch.
3. **Branch**: default (e.g. `main`), folder **/ (root)**.
4. Save. The site will be at `https://<username>.github.io/3dance/`.

### From the CLI

Use [GitHub CLI](https://cli.github.com/) (`gh auth login` first). From the repo root:

**Create** (Pages not set up yet):

```bash
gh api repos/:owner/:repo/pages -X POST -f 'source[branch]=main' -f 'source[path]=/'
```

**Update** (Pages already exists):

```bash
gh api repos/:owner/:repo/pages -X PUT -f 'source[branch]=main' -f 'source[path]=/'
```

Replace `main` with your default branch if different. Run from the repo directory so `:owner` and `:repo` are resolved by `gh` from the current repo.

## What gets served

- **Root**: `index.html`, `favicon.svg`, `manifest.webmanifest`, videos (e.g. `football.mp4`, `sticks.mp4`, `output.mp4`, `bullet-time.mp4`).
- **assets/**: logos and icons.

Paths are relative, so they work at the project URL above. The `poc/` directory is gitignored and is not part of the site.

## Custom domain (optional)

To use a custom domain, add a `CNAME` file in the repo root containing the domain (e.g. `dance.example.com`), then configure DNS as per [GitHub’s custom domain docs](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site).

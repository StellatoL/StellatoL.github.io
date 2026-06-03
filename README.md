# Stellato's Astro Blog

Personal blog source for <https://stellatol.github.io/>.

This repository contains the rebuilt Astro version of Stellato's blog. It is a static site focused on study notes, technical writing, resources, and personal knowledge archives.

## Tech Stack

- Astro
- Svelte islands
- Tailwind CSS
- Expressive Code
- Pagefind search
- pnpm

## Local Development

Install dependencies:

```bash
pnpm install
```

Start the development server:

```bash
pnpm run dev
```

Build the static site:

```bash
pnpm run build
```

Preview the production build:

```bash
pnpm run preview
```

Create a new post:

```bash
pnpm run new-post
```

## Deployment

The site is deployed to GitHub Pages through the workflow at `.github/workflows/deploy.yml`.

Pushing to `main` triggers:

1. Astro build
2. Pagefind index generation
3. GitHub Pages deployment

The live site is available at:

<https://stellatol.github.io/>

## Notes

- Use `pnpm.cmd` instead of `pnpm` on Windows PowerShell if script execution policy blocks `pnpm.ps1`.
- GitHub Pages should use **GitHub Actions** as its source, not the default Jekyll branch deployment.

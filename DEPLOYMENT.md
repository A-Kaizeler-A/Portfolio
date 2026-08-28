# Deployment Guide — GitHub Pages + Cloudflare

This guide gets `index.html` live at a custom domain, served free via GitHub Pages and fronted by Cloudflare for HTTPS, caching, and DNS management.

## 0. Clean up the repo before it goes public

`CHARM/`, `PhDThesis/`, and `StressGranules/` (plus `CHARM.docx` and `SGPaper.docx`) aren't needed for the live site — the project cards on the page link out to their own separate GitHub repos instead of shipping a duplicate copy here, and `PhDThesis/` is unpublished thesis source you don't want public. A `.gitignore` is already set up to keep them out of the repo going forward (they stay on your disk, just untracked).

If any of them were already committed in a previous push, untrack them now — this is safe to run even if they were never committed (`--ignore-unmatch` just skips anything not tracked):

```bash
cd path/to/Portfolio
git rm -r --cached --ignore-unmatch CHARM PhDThesis StressGranules CHARM.docx SGPaper.docx
```

Run `git status` afterward — anything still showing under those paths as untracked is expected and fine; `.gitignore` will keep it from being picked up again.

## 1. Push the site to GitHub

Your repo (`A-Kaizeler-A/Portfolio`) already exists, so from the repo root:

```bash
git add .gitignore .nojekyll index.html DEPLOYMENT.md Alexandre_Kaizeler_CV.pdf Alexandre_Kaizeler_CV.docx
git commit -m "Clean up repo and publish portfolio homepage"
git push origin main
```

The `.nojekyll` file tells GitHub Pages to serve the site as-is instead of running it through Jekyll (which isn't needed here and can occasionally trip over dotfiles or underscore-prefixed paths).

## 2. Enable GitHub Pages

1. On GitHub, go to **Portfolio → Settings → Pages**.
2. Under **Build and deployment → Source**, choose **Deploy from a branch**.
3. Branch: `main`, folder: `/ (root)` — since `index.html` lives at the repo root. Save.
4. GitHub builds the site (usually under a minute) and gives you a URL like `https://a-kaizeler-a.github.io/Portfolio/`. Open it to confirm the page loads before moving on.

A repo named exactly `<username>.github.io` would serve from the domain root instead of a `/Portfolio/` subpath — worth renaming later if you want a cleaner default URL, but not required for what follows.

## 3. Point a custom domain at it (optional but recommended)

If you own a domain (e.g. `alexandrekaizeler.com`) and want to use it instead of the default `github.io` URL:

1. In **Settings → Pages → Custom domain**, enter your domain and save. GitHub creates a `CNAME` file in the repo automatically — don't delete it later.
2. Choose one of two DNS patterns:
   - **Apex domain** (`alexandrekaizeler.com`): add four `A` records pointing to GitHub's Pages IPs:
     `185.199.108.153`, `185.199.109.153`, `185.199.110.153`, `185.199.111.153`
   - **Subdomain** (`www.alexandrekaizeler.com` or `portfolio.alexandrekaizeler.com`): add a `CNAME` record pointing to `a-kaizeler-a.github.io`

## 4. Move DNS to Cloudflare

1. Sign up at Cloudflare (free plan) and add your domain — it scans your existing DNS records automatically.
2. At your domain registrar, change the nameservers to the two Cloudflare nameservers shown during setup (e.g. `xxx.ns.cloudflare.com`). This step can take a few hours to propagate.
3. In Cloudflare's DNS tab, confirm the `A` (or `CNAME`) records from step 3 are present and set to **Proxied** (orange cloud) — this routes traffic through Cloudflare's CDN and gives you free HTTPS, caching, and DDoS protection.
4. Under **SSL/TLS**, set the encryption mode to **Full** (not "Flexible") — GitHub Pages already serves HTTPS, so Full avoids redirect loops.
5. Back on GitHub (**Settings → Pages**), check **Enforce HTTPS** once the certificate is issued (can take up to 24h after DNS propagates).

## 5. Verify

- Visit your custom domain and confirm the padlock/HTTPS is active.
- Test on mobile — the nav should collapse into the hamburger menu below the `md` breakpoint (768px).
- Run a quick Lighthouse pass (Chrome DevTools → Lighthouse) — this build has no build step and no heavy dependencies, so it should score well on performance by default; the main thing to watch is total page weight if you add large images later (Fig1–Fig6.png etc. are several hundred KB to 1.3MB each — compress or lazy-load any you embed on the project detail pages).

## 6. Ongoing updates

Every `git push` to `main` triggers a new Pages deployment automatically — no manual rebuild step, since this is static HTML with no bundler.

# DQD Media Website — Handoff Document

Last updated: 2026-07-11

This covers the full history of work on dqdmedia.com: what exists, why it's built the way it is, and what's still open. Written so a fresh Claude Code session (or a human) can pick this up cold on a new machine.

## Quick facts

- **Repo:** `DQDMedia/DQDMedia.github.io` on GitHub, remote via SSH (`git@github.com:DQDMedia/DQDMedia.github.io.git`)
- **Hosting:** GitHub Pages, custom domain via `CNAME` file → `dqdmedia.com`
- **Auth:** SSH key-based (was set up as `~/.ssh/id_ed25519_dqdmedia` on the original laptop — **this needs to be set up again on any new machine**, including the Mac mini, or `git push` will fail)
- **Site type:** static HTML/CSS/JS, no build step, no framework, no node/npm
- **Local preview:** no node/npx available in the working environment; local preview server used a plain Ruby static server (`ruby -run -e httpd`), configured via `.claude/launch.json` in the working directory (not the repo itself)
- **Analytics:** GoatCounter (`dqdmedia.goatcounter.com`), chosen after Cloudflare Web Analytics repeatedly failed to verify (unresolved Cloudflare-side bug)
- **Owner:** Daniel Durgin, photographer/videographer, DQD Media, Myrtle Beach SC. Business phone `843.281.7322` (forwards to cell). Email `daniel@dqdmedia.com`. Instagram `@dqd.media`.
- **Print sales:** existing client gallery at `https://dqdmedia.client-gallery.com/gallery/dqdmedia` — this is where "Purchase Prints" links point. Not something built in this repo; it's a separate hosted service.

## Current site structure (live as of 2026-07-11)

The site was fully rebuilt from a single hash-routed page into four real, separate static pages plus a landing hub. This was a deliberate architecture change — see "The big rebuild" below for why.

```
/                           landing hub — 4 doors (Fine Art, Events, Video, Portfolio)
/fine-art/                  fine art catalog, grouped by category, lightbox, purchase button
/events/                    index of events, links to per-event pages
/events/morocco-haiti-world-cup/   full photo set from FIFA World Cup Group C coverage
/events/hgtc-foundation-gala/      full photo set from HGTC Foundation gala
/video/                     YouTube embeds, DQD splash video plays once per session
/portfolio/                 curated 23-image cross-section of skills
/resume.html                HTML resume built from the PDF resume, print-optimized
/404.html                   custom "Frame Not Found" page
/oldsite/                   full archive of the pre-rebuild single-page site (noindexed, rollback point)
/manifest.json              web app manifest
/sitemap.xml                real per-page URLs with image entries
/robots.txt                 disallows /oldsite/
```

All pages share the same dark theme: `--paper:#161514` background, `--ink:#ece8e0` text, `--press:#e2524c` accent red, gold `#f0b429` for purchase/CTA elements. Fonts: Oswald (headers), Source Serif 4 (body), IBM Plex Mono (labels/meta).

## Chronological history — what happened and why

### Early work (before this session's main thread)
- Built the original single-page site: hero image, About, Contact, hash-routed Fine Art (`#portfolio/slug`) and Editorial/Events (`#editorial/slug`) sections, lightbox with focus trap, deep-linking via `history.replaceState`.
- Built `resume.html` from the user's PDF resume — two-column print layout, verified to fit one page via simulated print-CSS injection.
- Added a one-time splash video (DQD Media logo) site-wide: played once per session, skipped for deep links and `prefers-reduced-motion`, click/Enter/Space to skip.
- Added `404.html`, `sitemap.xml` (image sitemap), `robots.txt`.
- Set up SSH auth after starting with a pasted PAT token (security concern flagged, switched to SSH).
- Set up GoatCounter analytics after Cloudflare Web Analytics failed repeatedly.
- Responsive images: every photo has a `-thumb.jpg` (900w) and a full master (2400w), served via `srcset`.
- Added right-click/drag deterrent on all `<img>` elements — explicitly framed to the user as a deterrent against casual downloading, **not** real protection (view-source, dev tools, and screenshots all bypass it trivially).

### SEO and metadata pass
- Added canonical tag, Open Graph/Twitter tags, `manifest.json`, `LocalBusiness` + `Person` JSON-LD structured data.
- Added Google Search Console verification (meta tag). Daniel is now (2026-07-11) also adding **domain-level verification via DNS TXT record** at the registrar — this is outside the repo, no code change needed, and both verification methods can coexist.
- Diagnosed that hash-routed URLs (`#editorial/slug`) are invisible to Google — a URL fragment is not a distinct page to a crawler. This became the core driver for the later full rebuild.
- Checked NAP (name/address/phone) consistency: found the Instagram bio links to a different domain (`dqd-photography.client-gallery.com`, not `dqdmedia.com`) and that some cached search results still showed the old personal phone number (843.813.7293) — likely from LinkedIn or another external listing, **not** from the site itself. These are outside the repo and unresolved; Daniel would need to fix them directly on Instagram/LinkedIn.

### Business phone number update
- Added `843.281.7322` (business line, forwards to cell) to the Contact section and to `resume.html`, replacing the old personal number `843.813.7293` everywhere on the site.

### Right-click deterrent, print sales research
- Discussed selling prints. Found an existing (already set up, not built by us) client gallery link (`dqdmedia.client-gallery.com`) already referenced in the old lightbox's "Purchase this print" link.
- **Legal note on World Cup photos:** Daniel attended the Morocco v Haiti FIFA World Cup match as a spectator (no media credential). He confirmed directly with the ticket terms that he **cannot sell prints** of those photos (ticket terms restrict commercial use), but display/self-promotion use (e.g., in a portfolio) is fine — this is a real, checked legal distinction, not an assumption.

### The big rebuild (2026-07-09 → 2026-07-10)
Daniel decided to overhaul the site rather than patch it further. Key decisions, in the order made:

1. **New structure:** four real sections instead of one scrolling page — **Fine Art**, **Events** (previously "Editorial"), **Video** (new), **Portfolio** (new — "best of my skills" umbrella, deliberately overlapping with Fine Art/Events in places but also containing content neither would show, like portraits).
2. **"Less is more" design direction:** applies to all four sections — sparser, more curated, not "show everything available."
3. **Fix the SEO problem as part of the rebuild**, not separately: instead of a client-side pushState + `404.html` catch-all trick (which still returns a real HTTP 404 status that crawlers respect — a common gotcha with GitHub Pages SPA hosting), build **real static files per page** so every URL returns a genuine 200. This is why `/events/morocco-haiti-world-cup/` and `/events/hgtc-foundation-gala/` are real directories with their own `index.html`, not JS-rendered routes.
4. **"Start anew"** rather than incrementally refactor the old single `index.html`. Before touching anything, the entire existing site was copied (not moved) to `/oldsite/` as a safety net — marked `noindex`, excluded in `robots.txt`. The live root domain was left completely untouched during this archival step, so nothing broke for visitors mid-transition.
5. **Portfolio content curation:** Daniel gathered a batch of ~54 candidate photos into `/Users/ddurgin/Documents/for sites/`. We reviewed them together — first via generated contact sheets (grid thumbnails), then at full resolution — and iterated down to 23 final images in `/Users/ddurgin/Documents/for sites/Keep/`, covering portraits, documentary/family, sports, wildlife/macro, aerial (day + night drone), night/astro landscape, and alternative-process work (lumen prints, light painting). These were processed into `photos/portfolio/` with the same thumb+master pipeline as everything else.
6. **Video content:** two YouTube videos from the `youtube.com/@DQDMedia` channel — "The Last Coin" (`QD0VqK3bN0Q`, a Digital Arts degree short film) and a Surfside Beach/pier drone Short (`Hp0GOH0hA2M`). **The Beastie Boys remake mentioned early on was never actually uploaded** — only these two are live. Videos are embedded via `<iframe>`, not self-hosted (avoids GitHub Pages bandwidth concerns and gets a free scrubber/CC).
7. **Build order:** landing hub first, then Fine Art, Events (index + 2 event pages), Portfolio, Video — each verified in a local preview (screenshots, console log checks, click-throughs) before moving to the next, and everything held locally until all sections were done and cross-linked correctly, then pushed together as one release (to avoid the live homepage linking to pages that didn't exist yet).
8. **`sitemap.xml` was regenerated** from scratch to list the real per-page URLs (7 `<url>` entries: home, fine-art, events index, the 2 event pages, video, portfolio) with `<image:image>` entries nested under the correct page instead of everything dumped under the homepage.
9. Found and fixed a **pre-existing bug** unrelated to the rebuild: `rooster.jpg` (a wildlife fine art photo) was missing its `-thumb.jpg` — silently 404ing images are a good thing to sweep for after any bulk photo work.

### Homepage polish (2026-07-10 → 2026-07-11)
- **Reported issue:** homepage felt slow to load images. **Root cause, confirmed with real file-size measurements:** the four door background images were CSS `background-image` full 2400w masters (~3.9MB combined for 3 photo doors), not the `-thumb.jpg` versions. CSS backgrounds can't use `srcset`, so there was no responsive fallback — switched all doors to load only `-thumb.jpg` files (~700KB combined), and updated the `<link rel="preload">` hint to match.
- **Fine Art and Portfolio doors now rotate randomly** on each homepage load, picked client-side from a small JS array of representative thumbs (7 category covers for Fine Art, all 23 images for Portfolio).
- **Events door always shows the most recently dated event's cover**, computed from a small `EVENTS_META` array (date + cover path) sorted client-side — not hardcoded, so it stays correct automatically as new events get added later.
- **Cross-gallery duplicate-cover avoidance:** discovered that Fine Art's "Intracoastal Sunset" (category cover) and Portfolio's "waterway-sunset-wide" are literally the same photograph, re-exported under two different filenames during two separate curation passes. Rather than banning either image from rotation, added a small collision check: if both doors would independently land on that same photo at once, Portfolio rerolls from the rest of its pool. (An earlier, cruder attempt had permanently excluded the World Cup diving-save shot from Portfolio's rotation because it was from the same *event* as what Events shows — Daniel correctly pushed back that "same event" isn't the same as "same image," so that image was restored to full eligibility. Only genuinely duplicate photos should ever be excluded.)
- **Audited all four galleries** for the same "should be thumbnail but isn't" issue after fixing the homepage. Found one more real instance: the `/events/` index page's two large cover cards (up to ~950px wide) were still pulling the full 2400w master on high-DPI/retina screens, because even a corrected `sizes` attribute couldn't get under the 900w thumbnail threshold at that rendered width and pixel density. Fixed by dropping the `srcset`/master option entirely for those two cards — same "thumbnail always, no exceptions" treatment as the homepage doors. Every other grid (Fine Art, Portfolio, both event sub-pages) was already correctly using thumbnails at their actual render size and needed no change.
- **Video splash video, section-scoped:** re-added the DQD Media logo splash intro (same asset/behavior as the old site-wide version — click/tap/Enter/Space to skip, `sessionStorage`-gated to once per session, respects `prefers-reduced-motion`), but scoped to `/video/` only via its own `videoSplashShown` session key, rather than site-wide like before.
- **Purchase Prints link/button, iterated twice:**
  1. First found that the Fine Art page's "Prints available..." line linked to the homepage contact section, not the actual print gallery — the *only* real purchase link on the whole site was buried inside the per-photo lightbox, easy to miss. Fixed the link target.
  2. Then made it a visible pill button per request to "make it stand out."
  3. First button version was gold-outline with a hover-fill effect. Daniel reported "doesn't look like anything change[d]" — correct diagnosis: hover-only styling is invisible on touch devices (no persistent hover state on tap) and was a subtle change even on desktop. **Lesson for future styling requests: don't rely on `:hover`-only affordances to satisfy a "make it stand out" ask — design the resting/default state to already look prominent, since a lot of traffic to a site like this will be mobile.** Final version: solid gold fill with a glow, visible immediately at rest, hover/focus just lightens the gold further.

### Infra note (worth knowing if this happens again)
Mid-session, the git working copy — cloned into a `/private/tmp/...` scratchpad directory — silently lost several files that hadn't been touched in a while (`CNAME`, `logo.png`, and critically `.git/HEAD` and `.git/config`). This wasn't git corruption; the scratchpad directory appears to reclaim files it considers stale. It was fully recoverable because the actual commit objects, refs, and logs were untouched — `HEAD` and `config` were just rewritten by hand (`HEAD` → `ref: refs/heads/main`, `config` → remote URL + branch tracking) and the two missing tracked files restored via `git checkout HEAD -- CNAME logo.png`. No data was lost, but it could have been much worse if it had happened before a push. **Takeaway: when working in a repo that lives in a scratchpad/temp directory, commit and push early and often rather than batching a lot of work before the first push** — GitHub is the durable copy, the scratchpad clone isn't guaranteed to be.

## Content pipeline reference (for adding new photos later)

Every gallery photo needs two files: the master and a `-thumb.jpg`, generated with macOS `sips` (no ImageMagick/ffmpeg available in this environment):

```sh
sips -Z 2400 -s formatOptions 85 SOURCE.jpg --out photos/CATEGORY/slug-name.jpg
sips -Z 900  -s formatOptions 78 SOURCE.jpg --out photos/CATEGORY/slug-name-thumb.jpg
```

Grid markup pattern used everywhere (Fine Art, Events, Portfolio):
```html
<img class="shot-img" src="thumb.jpg" srcset="thumb.jpg 900w, master.jpg 2400w"
     sizes="(max-width: 600px) 100vw, 380px" alt="..." loading="lazy">
```
Do **not** use `sizes="100vw"` on anything wider than ~450px CSS-rendered width — that was the exact bug found and fixed on the Events index cards. If a card/hero image needs to render wider than ~450px, either accept it'll sometimes load the master on high-DPI screens, or (the approach used here) just drop the `srcset`/master option and always serve the thumb.

## Known open items / not done

- **Video section only has 2 videos.** The Beastie Boys remake mentioned early in planning was never uploaded to YouTube. Daniel said he can find/make more over time — just needs the `youtube.com/watch?v=...` or `/shorts/...` URL, and a new `.video-entry` block added to `/video/index.html` (wide `16:9` or tall `9:16` depending on format).
- **Instagram bio** still links to `dqd-photography.client-gallery.com` instead of `dqdmedia.com`. Outside the repo — Daniel would need to update this directly in the Instagram app.
- **Old phone number possibly still cached** in some external listings (LinkedIn was the suspected source, unconfirmed) — outside the repo, not fixable from here.
- **Google Business Profile** — Daniel believes one exists under "DQD Media." Never independently verified that its listed phone/address matches the current site (843.281.7322, Myrtle Beach SC). Worth a manual check.
- **No watermarking or resolution-capping** on images intended for print sale beyond the general right-click/drag deterrent (which is explicitly *not* real protection). If print sales become a real revenue stream, worth revisiting: either cap served resolution below print-usable quality, or add a visible watermark to web-served copies while keeping true masters only in the fulfillment pipeline.
- **Services/rates content** — was being drafted early in the project (a "SELECTED PUBLICATIONS & COVERAGE" style section), removed for lack of real content, never resolved. A testimonials/publications section was also discussed and removed for the same reason — revisit only when there's real content to put there.
- **Newsletter signup** — Mailchimp vs. Buttondown was discussed, no decision made, nothing built.
- **`/oldsite/` rollback archive** — still present and noindexed. No urgency to remove it, but it's dead weight in the repo once the new site has been stable for a while. Safe to delete entirely whenever Daniel is confident he won't need to revert.
- **Q Fab Works site** — a separate project/repo entirely (`github.com/DQDMedia/qfabworks`), not part of this repo. Mentioned here only so it isn't confused with this one.

## Machine-transfer checklist (why this file exists)

Daniel is moving from a laptop to a Mac mini. Everything in this repo is already committed and pushed — `git pull` on the new machine gets it all. Two things do **not** transfer automatically and need to be redone on the new machine:

1. **SSH auth for GitHub** — generate/copy an SSH key and add it to the GitHub account, or `git push` will fail.
2. **Claude's local memory files** (`~/.claude/projects/.../memory/*.md`) — these live on the laptop's disk and won't exist on the Mac mini unless manually copied or synced. This `HANDOFF.md` file is the portable substitute: everything load-bearing from memory has been folded into this document.

**2026-07-11 — Mac mini transfer confirmed:** new SSH key (`~/.ssh/id_ed25519_dqdmedia`) generated and added to the GitHub account, remote switched to SSH, git identity set to Daniel Durgin / daniel@dqdmedia.com, local preview (`.claude/launch.json`) repointed at this machine's working directory. Push access verified end-to-end.

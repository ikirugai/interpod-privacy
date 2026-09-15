# interpod-privacy: rules for any Claude (or human) working in this repo

Company-wide rules are in `ikirugai/dev-setup/CLAUDE.company.md` and apply here. This file only adds what is specific to this repo.

## What it is
A single static HTML page holding the InterPod privacy policy, published by GitHub Pages at `https://ikirugai.github.io/interpod-privacy/`. It was created on 2026-04-17 so the App Store and Play Store listings had a public privacy URL. Since 2026-08-18 the canonical policy lives at `https://interpod.ikirugai.com/privacy` (source `marketing/privacy.html` in the `InteractivePodcasts` repo, Cloudflare Pages project `interpod-site`), and this page is a legacy mirror that must stay reachable because older store metadata may still link to it. Status: live, low priority, two commits ever.

## Architecture
| Layer | Technology | Where |
|---|---|---|
| Page | Hand-written HTML with inline CSS, no build step, no JavaScript | `index.html` |
| Hosting | GitHub Pages from the `main` branch root (no `docs/` folder, no Jekyll config) | repo settings on GitHub |
| Related | The canonical, fuller policy and the marketing site | `InteractivePodcasts/marketing/privacy.html` |
| Loose file | `featured-graphic.html` is an untracked 1024 x 500 mock-up for the Play Store featured graphic. Not part of the site; do not deploy it | repo root, untracked |

## Environments
| env | branch / ref | URL | how it deploys |
|---|---|---|---|
| prod | `main` | `https://ikirugai.github.io/interpod-privacy/` | Push to `main`; GitHub Pages rebuilds in about a minute. No staging |
| canonical (other repo) | `InteractivePodcasts` `main`, `marketing/` | `https://interpod.ikirugai.com/privacy` | `npx wrangler pages deploy . --project-name=interpod-site` from `marketing/`, ikirugai Cloudflare account |

GitHub Pages is switched on in the repo settings (Settings > Pages > Deploy from branch `main`, folder `/`). If the page ever 404s, that setting is the first thing to check; the repo is private, which GitHub Pages allows only on a paid plan, so a plan change can silently take the page down.

## Run locally
```bash
open index.html                    # or: python3 -m http.server 8000 and browse http://localhost:8000
```
No dependencies, no `.env`.

## Tests and checks
None automated. After a change: push, wait a minute, then `curl -sI https://ikirugai.github.io/interpod-privacy/ | head -1` should return 200, and load the page in a browser to check it renders on a phone-width viewport.

## Secrets
None. The page contains only public text and the contact address `info@ikirugai.com`.

## What the policy must say
When mirroring or rewriting, the canonical policy covers these points and this page should not contradict them:
- No accounts, no user database; subscriptions, history and cached transcripts stay on the phone (iCloud or Google backup uses the user's own quota).
- Questions and transcript requests go through ikirugai's Cloudflare Worker, which forwards to OpenAI (answers, voice), Groq, Deepgram and OpenAI Whisper (transcription), with transcripts cached in Cloudflare R2 keyed by episode audio URL, not by user.
- Device attestation (Apple App Attest, Google Play Integrity) is used to stop abuse; it identifies the app install, not the person.
- Spoken questions are converted to text on the device by the system speech recogniser.
- Contact address is `info@ikirugai.com`. Not directed at children under 13.

## Gotchas
- 2026-08-18: Google Play's automated checker flagged a GitHub-hosted policy URL as 404 (the URL in Play was a `github.com/.../blob/...` link, not this Pages site). Play Console now uses the Cloudflare URL. Do not point any store listing back at GitHub.
- The text here (dated 2026-04-17) is shorter and older than the canonical policy and does not describe the current architecture (Worker proxy, Groq / Deepgram / OpenAI transcription with R2 caching, App Attest / Play Integrity, on-device speech recognition). If a store or a user reads this page, it is the one that is out of date.
- Keep both copies consistent whenever the app's data flows change: edit `InteractivePodcasts/marketing/privacy.html` first, then mirror or redirect here.
- Company copy rules apply: no em dashes, no emoji. The page currently uses plain hyphens.

## Status and next steps
- 2026-04-17: policy added and simplified (two commits). No changes since.
- 2026-08-18: superseded as the canonical URL by `https://interpod.ikirugai.com/privacy`.
- Next: check App Store Connect's privacy policy URL and switch it to the Cloudflare URL if it still points at GitHub. Then either replace `index.html` with the canonical text plus a "canonical version" link, or add a `<meta http-equiv="refresh">` redirect to the Cloudflare page. Delete or commit `featured-graphic.html` deliberately rather than leaving it untracked.

## Visual verification
```bash
BIN="$HOME/Library/Caches/ms-playwright/chromium_headless_shell-*/chrome-headless-shell-mac-arm64/chrome-headless-shell"
$BIN --headless --disable-gpu --hide-scrollbars --window-size=430,1600 --screenshot=/tmp/out.png "file://$PWD/index.html"
```
Then read `/tmp/out.png` and look at it before presenting. Use a 430 px width because store reviewers open this on a phone.

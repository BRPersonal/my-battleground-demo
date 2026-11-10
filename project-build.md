# Battleground – Build Plan

Companion to `project-brief.md`. Each step works on its own and ends with something you can run and check, so we can stop after any step and still have a working app.

## Target file structure
```
my-battleground-demo/
├── index.html
├── style.css
├── app.js              # UI, fetch calls, BattleDex
├── api/
│   ├── verdict.js      # Claude call, fictional check, rate limit, issues art token
│   └── image.js        # OpenAI image call, verifies art token
├── lib/
│   └── ratelimit.js    # Upstash counter + token helpers (shared by both functions)
├── project-brief.md
└── project-build.md
```

## Step 1 – Static page with fake data
**Build**
- Two text inputs, a Fight button
- Winner's card layout (name, tagline, reasoning, image slot)
- Reveal animation with CSS keyframes (face-off, flash, card slides in)
- Placeholder (silhouette or shimmer) in the image slots
- Hard-coded fake verdict so the whole flow plays without any API

**Done when:** typing two names and clicking Fight plays the animation and shows a card.

## Step 2 – `/api/verdict` with Claude
**Build**
- Vercel serverless function that takes `{ fighterA, fighterB }`
- Prompt Claude (Haiku-class model) to return JSON: `winner`, `loser`, `reasoning`, `tagline`, plus a fictional check
- If either fighter is not fictional, return `{ error: "not_fictional" }`
- Validate the input (non-empty, max length) and the model's JSON before returning it
- Key read from `ANTHROPIC_API_KEY` env var
- Test locally with `vercel dev` and `curl`

**Done when:** `curl` returns a valid verdict for two fictional fighters and `not_fictional` for a real person.

## Step 3 – Wire frontend to verdict
**Build**
- Replace the fake data with a `fetch` to `/api/verdict`
- Loading state (the reveal animation runs while we wait)
- Error states: network failure, `not_fictional`, bad response

**Done when:** the real verdict appears on the card, and each error shows a clear message.

## Step 4 – `/api/image` with OpenAI
**Build**
- Function that takes a fighter name and returns an image
- Fixed trading-card style prompt prefix
- Cheapest image tier and small size; request base64 so images do not expire
- Key read from `OPENAI_API_KEY` env var
- Frontend calls it twice in parallel after the verdict, swaps in each image on arrival
- Fallback and "retry art" button when a call fails or is refused

**Done when:** both images fade in after the verdict, and a failed image does not break the card.

## Step 5 – Rate limiting and art token
**Build**
- Upstash Redis via the Vercel marketplace (free tier)
- Fixed-window counter in `lib/ratelimit.js`: 3 fights per minute per IP, checked in `/api/verdict`
- `/api/verdict` returns a short-lived token with a successful verdict; `/api/image` rejects requests without a valid one
- UI shows a clear "slow down" message on 429

**Done when:** the fourth fight within a minute is rejected, and calling `/api/image` directly without a token fails.

## Step 6 – BattleDex
**Build**
- "Save" button on the winner's card; saves both fighters as `{ name, image }`
- Downscale images to about 256px JPEG with a canvas before saving
- Save waits until the art has arrived (or saves without it if the user retries later)
- BattleDex view listing saved fighters, with delete
- Handle storage full and corrupted data without crashing

**Done when:** saved fighters survive a page refresh, and many entries fit in local storage.

## Step 7 – Deploy and cost safety
**Build**
- Deploy to Vercel and add env vars (Anthropic, OpenAI, Upstash)
- Set hard spend limits in the Anthropic and OpenAI dashboards
- Run through the whole flow on the live URL, including the rate limit and a non-fictional input

**Done when:** the live app works end to end and both spend caps are set.

## Decisions to make along the way
- **Step 2:** the exact model, after checking current pricing
- **Step 4:** the image model, size, and quality tier, after checking current pricing and the actual cost per fight
- **Step 6:** how to handle saving when one image failed

## Later (out of scope for now)
- Backend sync across devices
- Quick refight from saved fighters

# Battleground – Project Brief

## 1. What are we actually building? (Scope)
A web app where you type in two fictional fighters, you get an AI-generated verdict on who'd win, plus AI-generated trading-card style character art for both.

This is a learning project, so the goal is to understand each piece as we build it, not just to ship quickly.

## 2. What's the smallest, simplest version worth shipping (MVP)?
- Two text inputs
- A fight button
- A verdict shown as a winner's card, with an animation and the reasoning for the win
- Character art for both fighters, shown after the verdict (not blocking it)
- A loading state while we wait on the API (the reveal animation covers the verdict wait; placeholders cover the art wait)
- A basic BattleDex saved to local storage
- You type two fighters, you get a real result, and you can save it if you want

### Verdict
- Claude returns structured JSON: `winner`, `loser`, `reasoning` (2-4 short sentences), and `tagline`
- The same object drives the winner's card and is what gets saved
- A cheap model (Haiku-class) is enough for this

### Art
- Generated after the verdict, one call per fighter, in parallel
- One fixed trading-card style prompt prefix so both images match
- Each card shows a placeholder (silhouette or shimmer) until its image arrives
- If an image fails or is refused, the card still works, with a fallback and a "retry art" option

### Fictional characters only
- The verdict call also checks that both fighters are fictional (characters, creatures, objects, etc.)
- If either one is a real person, it returns `{ error: "not_fictional" }` and no images are generated
- This keeps cost down and avoids sending real-person prompts to the image API

### BattleDex
- Each entry saves only the fighter's `name` and `image`
- Images are downscaled (about 256px, JPEG) in the browser before saving, so local storage (about 5MB) can hold many entries
- Saving is optional and the user chooses when. If the art has not arrived yet, save waits for it

### Rate limiting
- 3 fights per minute per IP
- A fight is one verdict call. The two image calls that follow it do not count separately, but they must carry a short-lived token issued by the verdict call, so images cannot be requested without a fight
- Over the limit returns a clear "slow down" message in the UI

### Cost
- Everything must work on a basic $5 account for each of Claude and OpenAI
- Use the cheapest image tier and a small size
- Set hard spend limits in both provider dashboards; that cap is the real safety net
- Check current pricing when choosing models

## 3. What are we building this with (tech stack)?
- HTML, CSS, and vanilla JavaScript for the frontend (CSS keyframes for the animation)
- Vercel for hosting and serverless functions (`/api/verdict`, `/api/image`)
- Claude API for the verdict
- OpenAI's image API for the character art
- Upstash Redis (Vercel marketplace, free tier) for the rate-limit counter, because serverless functions do not share memory
- Local storage for saving BattleDex
- API keys live in Vercel environment variables, never in the frontend

## 4. What's the long–term vision, beyond just the MVP?
I would eventually like a real backend where battle decks and characters sync across devices instead of just living in local storage. I'd also want a quick refight option using characters already saved in the battle decks instead of typing two new ones every single time.

Because the BattleDex stores individual fighters (name and image), a quick refight can pick two saved fighters without any change to the data model.

## 5. Decisions log
| Question | Decision |
|---|---|
| Verdict format | Winner's card with animation and reasoning |
| When do images show? | After the verdict |
| Project type | Learning project |
| Rate limit | 3 fights per minute per IP |
| What BattleDex saves | Fighter name and image only |
| Art style | Trading-card |
| Budget | Basic $5 accounts for Claude and OpenAI |
| Allowed fighters | Fictional only |

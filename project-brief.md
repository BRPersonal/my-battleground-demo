# Battleground – Project Brief

## 1. What are we actually building? (Scope)
A web app where you type in two fighters, you get an AI-generated verdict on who'd win, plus AI-generated character art for both.

## 2. What's the smallest, simplest version worth shipping (MVP)?
- Two text inputs
- A fight button
- A working verdict with art
- A loading state while we wait on the API
- A basic BattleDex saved to local storage
- You type two fighters, you get a real result, and you can save it if you want

## 3. What are we building this with (tech stack)?
- HTML, CSS, and vanilla JavaScript for the frontend
- Vercel for hosting and serverless functions
- Claude API for the verdict
- OpenAI's image API for the character art
- Local storage for saving BattleDex

## 4. What's the long–term vision, beyond just the MVP?
I would eventually like a real backend where battle decks and characters sync across devices instead of just living in local storage. I'd also want a quick refight option using characters already saved in the battle decks instead of typing two new ones every single time.


# Vocabulary Memory App — Brainstorm

## Concept

A vocabulary learning app that replaces word + definition flashcards with personal anchors — vivid memories, real people, and imagined scenes that make words impossible to forget. Instead of drilling a definition, you encode a word through something that already lives in your memory.

**Tagline:** *Words that stick because they mean something to you.*

---

## Core Philosophy

Traditional flashcard apps assume you can memorize a word by repeatedly exposing yourself to its definition. That works for shallow recall but fails for real retention.

This app is built on the principle of **elaborative encoding** — the idea that new information sticks when it's connected to something emotionally or experientially meaningful. When you learn the word *tenacious* by associating it with your grandmother who rebuilt her business three times, you don't forget it. The word is no longer a dictionary entry — it's a story you already know.

---

## Features

### Word Onboarding
When a user adds a new word, the app walks them through a guided process to build an anchor. The user enters the word, then chooses how to encode it.

**At least one anchor is required before a word card can be saved.** No anchor, no word. This is intentional — the encoding step is the product.

---

### Guided Anchor Prompts
Four prompt types help users find the right personal connection:

- **Person anchor** — "Does this word describe someone you know? Who, and why?"
- **Memory anchor** — "Think of a specific moment in your life this word captures. Describe it in 1–2 sentences."
- **Scene anchor** — "Imagine the most vivid scene where this word belongs — where are you, what do you see?"
- **Contrast anchor** — "What's the opposite of this word, and who or what embodies that?"

---

### Custom Anchor
If none of the guided prompts fit, the user can write a completely free-form anchor — any personal connection that makes the word memorable.

---

### Image / Photograph
Users can upload an image as their anchor. An anchor is either **image-only** or **text-only** — not both. The image is stored in Supabase Storage and displayed on the card front during review.

---

### Word Wall
The home screen. A grid displaying all the user's saved words. Each card shows the word and its anchor type (or image thumbnail). **Edit and delete controls appear directly on the card** — there is no separate detail view.

From the Word Wall, users can:
- Add a new word
- Edit or delete an existing word
- Start a review session

---

### Active Recall

The learning engine. Words are never gated behind a date — the user can review anytime. Words are **priority-ordered** so the ones that need the most work surface first.

**Review flow:**
1. A card appears showing only the user's anchor (text or image)
2. The user tries to recall the word
3. They flip the card — the word is revealed
4. The user self-rates their recall:
   - **Forgot** — I completely forgot
   - **Almost** — I remembered but it was a struggle
   - **Easy** — I knew it instantly

**Interval algorithm** (drives review priority, not access):
| Rating | New interval |
|--------|-------------|
| Forgot | 1 day |
| Almost | 3 days |
| Easy | current interval × 2 |

**Review queue priority order:**
1. Never reviewed words first
2. Most overdue words next (next_review_date ascending)
3. Tiebreak: Forgot > Almost > Easy

**Session behavior:**
- No skip option — the user must rate a card before the next one appears
- Session runs until the user manually exits — no fixed batch size, no end screen

---

## User Flows

### Add Word — Step-by-step wizard

```
Step 1: Enter the word
  └─ [Next →]

Step 2: Choose anchor type
  Options: Person / Memory / Scene / Contrast / Custom / Image
  └─ [← Back]  [Next →]

Step 3: Fill in the anchor
  Header shows the guided prompt question for the chosen type
  e.g. "Think of a specific moment in your life this word captures."
  Textarea (text types) or file picker (image type)
  └─ [← Back]  [Save]
```

- Back navigation is allowed at each step
- Changing anchor type on Step 2 clears any anchor content entered on Step 3
- Abandoning the flow at any step discards everything — nothing is saved

---

### Edit Word

Tapping "edit" on a Word Wall card re-opens the onboarding wizard, pre-filled with the existing word and anchor data. The user can change the word, anchor type, or anchor content.

- Saving an edit **resets all review stats** (last_rating → null, interval_days → 1, next_review_date → null) — the word is treated as never reviewed

---

## MVP Scope

### In MVP
- [ ] Email + password authentication
- [ ] Add a word with one anchor (guided prompt, custom, or image)
- [ ] Word Wall — grid view of all words
- [ ] Active recall session with flip card + self-rating
- [ ] Priority-ordered review queue
- [ ] Spaced repetition interval updates
- [ ] Word editing after creation
- [ ] Word deletion

### Explicitly Deferred
- Multi-anchor per word
- Hint recall (user reconstructs what they wrote, with AI evaluation)
- Shallow encoding mode (AI-assisted reconstruction)
- Mobile app
- Cross-device sync (MVP is single-device web)

---

## Architectural Decisions

| Decision | Choice | Rationale |
|---|---|---|
| Anchors per word | Exactly one | Simpler schema, faster to build, sufficient for MVP |
| Anchor format | Image-only OR text-only | Clean distinction; anchor cannot be null |
| Card reveal | Word only, no definition | The anchor is the encoding — a definition would undermine the model |
| Draft behavior | Discard on abandon | No partial state to manage; keeps onboarding flow simple |
| Review trigger | Always available, priority-ordered | Better UX than a date gate; words that need work float to top naturally |
| Review skip | Not allowed | User must commit to a rating; no passive card flipping |
| Review session end | User exits manually | No fixed batch or summary screen for MVP |
| Onboarding structure | Step-by-step wizard with back nav | Focused attention per step; back nav prevents frustration |
| Guided prompt on Step 3 | Shown as header/label | The prompt is the core value — hiding it defeats the purpose |
| Word Wall interaction | Edit/delete directly on card | No detail view needed; reduces navigation depth |
| Edit behavior | Re-opens onboarding wizard, pre-filled | Consistent UX with add flow; editing resets review stats to force re-learning |
| Auth | Email + password | Simplest to build, sufficient for MVP |
| Data storage | Supabase (cloud) | Handles auth + database + image storage without a custom backend |
| Backend server | None for MVP | Supabase client SDK + Row Level Security handles everything directly |

---

## Color Palette

| Role | Hex | Usage |
|---|---|---|
| Text | `#0e2e0f` | All body text, headings, labels |
| Background | `#d6edc7` | App background |
| Primary | `#ce4527` | CTAs, buttons, destructive actions |
| Secondary | `#fff6dc` | Card backgrounds, surfaces |
| Accent | `#67c190` | Highlights, success states, active indicators |

---

## Tech Stack

```
Frontend    React + Vite 5 + TypeScript
Styling     Tailwind CSS
State       Zustand
Animations  Framer Motion  (card flip)
Routing     React Router v6
Auth/DB     Supabase        (auth + postgres + storage)
Analytics   PostHog
Deploy      Vercel
```

---

## Database Schema

```sql
words
  id               uuid primary key default gen_random_uuid()
  user_id          uuid references auth.users not null
  word             text not null
  anchor_type      text not null  -- person | memory | scene | contrast | custom | image
  anchor_text      text           -- null if anchor_type = image
  anchor_image_url text           -- Supabase Storage URL, null if no image
  created_at       timestamptz default now()
  last_reviewed_at timestamptz
  last_rating      text           -- forgot | almost | easy | null
  interval_days    int default 1
  next_review_date date
```

**Row Level Security policy:**
```sql
create policy "users see own words" on words
  for all using (auth.uid() = user_id);
```

---

## App Routes

| Route | View |
|---|---|
| `/login` | Email + password sign in / sign up |
| `/` | Word Wall — grid of all words, Add Word + Review buttons |
| `/add` | Word onboarding multi-step flow |
| `/review` | Review session — priority-ordered flip cards |

---

## Future Features

- **Multi-anchor per word** — let users attach more than one anchor type
- **Hint recall** — show the anchor, ask the user to reconstruct what they wrote; evaluate with AI
- **Shallow encoding mode** — AI helps the user reconstruct their memory rather than just rating themselves
- **Mobile app** — Expo/React Native with native camera access
- **Cross-device sync** — already supported by Supabase, just needs UI
- **Decks / tags** — group words by context (e.g., work, travel, reading)
- **Stats / progress view** — streaks, words mastered, recall accuracy over time

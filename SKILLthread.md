---
name: thread-drop
description: Use this skill when the user wants to create a Twitter-thread-style carousel for Instagram or LinkedIn — the format where each slide is a designed mockup of a tweet (profile pic, display name, handle, verified tick, tweet body). Trigger on requests containing "thread drop", "tweet carousel", "twitter carousel", "tweet-style carousel", "thread to carousel", "screenshot thread", or any request to turn a Twitter thread into slide posts. Also trigger when the user pastes tweet text and wants it formatted into slides, or gives a topic and asks for a thread to be written in tweet-style format. This skill writes the thread in The Identity Method voice (using CLAUDE.md), breaks it into slides using TIM combining rules, handles image planning, and outputs a config.json that feeds the thread-drop renderer tool. Do NOT use for standard Instagram carousels (use carousel-build skill instead), for reels, or for single-tweet writing.
---

# Thread Drop — The Identity Method

## Core principle

The tweet-style carousel is the highest-leverage visual format online right now. It looks premium, reads like authority, and gets saved because each tweet feels like a self-contained thought. But most coaches do it wrong: they paste LinkedIn posts into tweet mockups and wonder why it feels flat.

A thread drop is not a long post cut into pieces. It's a sequence of standalone thoughts, each one earning its own space, each one working even if it's the only slide someone sees.

If a tweet cannot stand alone, it should not be a tweet.

## What this skill does

Takes a topic or pasted text and produces three things:

1. A full thread written in TIM voice, tweet by tweet
2. A slide breakdown showing which tweets go solo and which combine
3. A clean config.json ready to feed the thread-drop renderer tool

This skill stops at the config. Image rendering, font loading, canvas drawing — all that lives in the renderer tool (separate build). The skill is the brain. The renderer is the hands.

## When to use this skill

Fire when the user:
- Says "thread drop", "tweet carousel", "twitter thread carousel"
- Pastes tweet text and wants it slide-formatted
- Gives a topic and asks for a thread
- Wants to turn an existing long post into tweet-style slides
- Shares a screenshot of a thread they want to match

Do NOT fire for:
- Standard Instagram carousels (use carousel-build)
- Reel scripts (use reel-script)
- Single tweet writing
- LinkedIn long-form posts
- Email or blog content

## The 7-step pipeline

Strict 7-step flow. Never skip steps. Always present step 3 for approval before proceeding.

### Step 1 — Input gathering

Three input modes. Identify which at the start.

**Mode A — Pasted thread.** User hands over text already written as tweets. Skill formats, restructures, and tunes voice but respects the user core thinking. Flag any tweets that break TIM voice rules and offer rewrites.

**Mode B — Topic brief.** User gives a subject and wants the full thread written from scratch. Skill pulls from CLAUDE.md for voice and frameworks, writes the thread, then continues.

**Mode C — Reference screenshots.** User shares screenshots of threads they like. Skill extracts the structural pattern and applies it to the new thread.

Always confirm which mode is active before proceeding. Example: "Mode B, topic brief. I'll write the thread from scratch. Topic: client retention. Confirm?"

### Step 2 — Profile details

Every thread drop needs profile data. Pull from profile.json if it exists in the repo root, or from the user directly.

Required fields:
- displayName
- handle (without the @)
- verified (true or false)
- headshotPath (relative path to headshot image)

If profile.json doesn't exist, ask once: "What profile should I use? I need display name, handle, verified yes or no, and path to headshot image. I'll save as profile.json so you don't have to re-enter it."

Once saved, reuse automatically on every future thread drop.

### Step 3 — Thread to slide breakdown (APPROVAL REQUIRED)

Most important step. Never skip the approval gate.

**Write the thread first.** Follow TIM voice rules. Every tweet is a standalone thought. Hook, body, CTA if needed.

**Tweet rules:**
- Every tweet under 280 characters (hard Twitter limit)
- Hook tweet = maximum impact, challenges a belief, ideally under 180 characters
- Body tweets = value delivery, each a standalone insight
- CTA tweet = contextual close, never "follow for more"

**Slide combining rules:**

| Tweet type | Slide rule |
|---|---|
| Hook tweet | Always solo (slide 1) |
| Any tweet with image | Always solo |
| Tweet over 200 chars | Solo |
| Tweet 150 to 200 chars | Solo (judge by weight) |
| Tweet under 150 chars | Can combine with next short tweet on same slide (with divider) |
| CTA tweet | Always solo (final slide) |

**Example breakdown for a 7-tweet thread:**

```
Slide 1 -> Tweet 1 (hook, solo)
Slide 2 -> Tweets 2 and 3 (both short, combined with divider)
Slide 3 -> Tweet 4 (has image, solo)
Slide 4 -> Tweets 5 and 6 (both short, combined with divider)
Slide 5 -> Tweet 7 (CTA, solo)
```

**Present the breakdown to the user BEFORE generating the config.** Format:

```
Here's the breakdown. 7 tweets -> 5 slides.

Slide 1: [tweet 1 full text]                    [solo, hook]
Slide 2: [tweet 2 full text]
         ---
         [tweet 3 full text]                    [combined, both under 150]
Slide 3: [tweet 4 full text]                    [solo, needs image]
Slide 4: [tweet 5 full text]
         ---
         [tweet 6 full text]                    [combined]
Slide 5: [tweet 7 full text]                    [solo, CTA]

Approve, or tell me what to change. I will not generate the config until you confirm.
```

Wait for explicit approval. If user says "combine 4 and 5 instead" or "split slide 2", adjust and re-present.

### Step 4 — Image handling

For any tweet marked as needing an image, identify the source. Four options, priority order:

1. **User provides** — ask if user has an image. If yes, they pass the file path.
2. **Image search** — for topical reference images.
3. **Website screenshot** — for tweets about a specific product or site.
4. **Gemini AI generation** — for conceptual or illustrative images.

**Image rules:**
- Image tweets always get their own slide (never combined)
- Landscape orientation preferred
- For UI or product references, prefer live screenshots over stock
- Minimum resolution: 1080px on the shortest side
- Images get rounded corners in final render (renderer handles this, not skill)

**This skill does not generate or fetch images.** For each image tweet, write a spec into config.json:

```json
"image": {
  "source": "gemini",
  "prompt": "Minimalist illustration of a stopwatch, editorial style, black and yellow palette",
  "path": null,
  "alt": "Stopwatch illustration"
}
```

Source values: user, search, screenshot, gemini, or none.

### Step 5 — Generate config.json

Output the full config in this exact schema. This is the handoff to the renderer.

```json
{
  "version": "1.0",
  "project": "thread-drop",
  "createdAt": "2026-04-14T00:00:00Z",
  "profile": {
    "displayName": "Dudley Dixon",
    "handle": "dudleydixon",
    "verified": true,
    "headshotPath": "./profile/headshot.jpg"
  },
  "theme": {
    "mode": "dark",
    "background": "#0A0A0A",
    "cardBackground": "#000000",
    "textPrimary": "#FFFFFF",
    "textSecondary": "#71767B",
    "accent": "#FFCC33",
    "divider": "#2F3336",
    "fontPrimary": "Chirp",
    "fontFallback": "Arial, system-ui, sans-serif"
  },
  "canvas": {
    "width": 1350,
    "height": 1350,
    "padding": 80
  },
  "thread": {
    "topic": "Client retention is an identity problem",
    "totalTweets": 7,
    "totalSlides": 5
  },
  "slides": [
    {
      "slideNumber": 1,
      "type": "solo",
      "purpose": "hook",
      "tweets": [
        {
          "tweetNumber": 1,
          "body": "Your clients aren't leaving because of results.",
          "characterCount": 49,
          "image": null
        }
      ]
    },
    {
      "slideNumber": 2,
      "type": "combined",
      "purpose": "body",
      "tweets": [
        {
          "tweetNumber": 2,
          "body": "You're chasing better check-ins, better plans, better apps. The clients who churn stopped feeling seen.",
          "characterCount": 108,
          "image": null
        },
        {
          "tweetNumber": 3,
          "body": "Retention isn't a results problem. It's an identity problem.",
          "characterCount": 63,
          "image": null
        }
      ]
    }
  ],
  "meta": {
    "voice": "The Identity Method",
    "author": "thread-drop skill",
    "rendererVersion": ">=1.0"
  }
}
```

Write the full config. No placeholders, no TODO, no ellipsis. Every tweet from the approved breakdown goes in.

### Step 6 — Rendering (HANDOFF, NOT EXECUTION)

This skill does not render slides. The renderer tool (separate build, Next.js app) consumes the config.json and handles canvas, theme, fonts, layout, PNG export.

When the skill reaches this step, output:

"Config generated. Save as thread-drop-configs/[topic-slug].json and run through the thread-drop renderer at tools.identity-method.com/thread-drop. The renderer will produce the final PNG slides."

If the renderer doesn't exist yet, add: "Until the renderer ships, use the config as a brief to build the slides manually in Figma. The structure and copy are locked, you just need to style them."

### Step 7 — Present and iterate

After handing over the config, offer next actions:

```
Thread drop ready:
- 7 tweets written
- 5 slides configured  
- 2 images needed (slide 3: gemini, slide 5: user-provided)
- Config saved: thread-drop-configs/client-retention.json

Want to:
1. Adjust any tweet wording
2. Change the slide breakdown
3. Swap an image source
4. Generate a different thread on the same topic
5. Ship it to the renderer
```

Wait for response. Re-run any step as needed.

## TIM voice rules (non-negotiable for every tweet)

All CLAUDE.md rules apply. Critical ones for tweets specifically:

- **No em dashes.** Ever.
- **British English, GBP.** Never USD.
- **No "most coaches" or "many coaches" openers.** Direct second-person.
- **No LinkedIn-speak.** No "value", no "unlock", no "leverage", no "game changer".
- **No weak closers.** No "thoughts?" or "agree?".
- **Every tweet a standalone thought.** If it only makes sense in sequence, rewrite.
- **Opinion-led.** Take a position on every tweet.
- **Short sentences.** Break lines. Rhythm over grammar.
- **Under 280 characters per tweet.** Hard limit.
- **Casual, slightly raw.** Smart mate in the gym, not a brand account.

## Examples

### Example 1 — Mode B (topic brief, full generation)

**User:** "Write me a thread drop about why most coaches' content isn't converting."

**Skill response:**

Mode B, topic brief. Writing from scratch using TIM voice. Here's the thread:

**Tweet 1 (Hook, 52 chars):**
Your content isn't the problem. Your positioning is.

**Tweet 2 (Body, 187 chars):**
You're posting 5 times a week. You're "providing value". You're using the right hashtags. None of it's converting because the content sounds like every other coach in your feed.

**Tweet 3 (Body, 118 chars):**
Conversion doesn't come from more posts. It comes from one sharp take that makes the right person stop scrolling.

**Tweet 4 (Body, 144 chars):**
Positioning is the belief people associate with you in 3 seconds. If they can't articulate it after 3 of your reels, you don't have one.

**Tweet 5 (Body, 89 chars):**
Most coaches copy what's working for others. The ones who win copy their own opinions.

**Tweet 6 (Body, 128 chars):**
Your content isn't competing with other coaches. It's competing with every other thing on the feed. Sharp beats safe, every time.

**Tweet 7 (CTA, 116 chars):**
If your content feels invisible, don't post more. Post sharper. Pick a belief and build the next 10 posts around it.

Now moving to step 3, slide breakdown.

```
Slide 1: Tweet 1                             [solo, hook, 52 chars]
Slide 2: Tweet 2                             [solo, 187 chars]
Slide 3: Tweet 3 + Tweet 4                   [combined]
Slide 4: Tweet 5 + Tweet 6                   [combined]
Slide 5: Tweet 7                             [solo, CTA]

7 tweets -> 5 slides. Approve or adjust before I generate the config.
```

### Example 2 — Mode A (pasted thread)

**User:** [pastes 6 tweets about workout programming]

**Skill response:**

Mode A, pasted thread. I'll respect your core thinking and tune the voice where needed. Flagging 2 issues:

- Tweet 2 uses "most coaches", banned opener. Rewrite?
- Tweet 5 ends with "thoughts?", weak closer. Swap for a direct landing line?

Confirm rewrites and I'll proceed to the slide breakdown.

### Example 3 — Breakdown adjustment

**User (after seeing breakdown):** "Combine 5 and 6 instead of 3 and 4. Tweet 4 needs its own space."

**Skill response:**

Updated breakdown:

```
Slide 1: Tweet 1                             [solo, hook]
Slide 2: Tweet 2                             [solo, 187 chars]
Slide 3: Tweet 3                             [solo]
Slide 4: Tweet 4                             [solo, as requested]
Slide 5: Tweet 5 + Tweet 6                   [combined]
Slide 6: Tweet 7                             [solo, CTA]

7 tweets -> 6 slides. Approve?
```

## What this skill must never do

- Render actual image files
- Load fonts or draw on a canvas
- Call external APIs for image generation
- Save files outside the conversation without permission
- Skip the step 3 approval gate
- Combine a tweet over 150 characters with another tweet
- Put an image tweet on a combined slide
- Write tweets over 280 characters
- Use em dashes
- Use banned TIM voice phrases
- Default to generic Twitter thread templates
- Pad a thread to hit a slide count. Fewer sharper tweets always win.

## Final checklist

Before handing over the config, confirm:

- [ ] Thread written in TIM voice
- [ ] Every tweet under 280 characters
- [ ] Hook under 180 characters and challenges a belief
- [ ] Every tweet is a standalone thought
- [ ] Slide breakdown approved by user
- [ ] Combining rules followed (150 char threshold, no image combos)
- [ ] Image specs defined for any image tweets
- [ ] Profile data loaded from profile.json
- [ ] Config.json matches the schema exactly
- [ ] No banned phrases, no em dashes, British English
- [ ] Next actions offered to user

If all boxes ticked, ship the config and hand off to the renderer.

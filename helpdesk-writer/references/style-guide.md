# Style Guide — Helpdesk voice

The target persona is *an end user of the app*. They're not a developer. They're in a hurry. They just want to know what to do.

> **A note on language.** Articles are written in EN-US by default, or in the language requested by the user. The voice rules below are written for English; when writing in another language, apply the equivalent form (e.g. in Portuguese, use "você", never "tu" or "o usuário"; in German, use "Sie" or "du" depending on the product's tone).

## Voice rules

### Use direct second-person address ("you"), always
Never "the user", "the customer", "one". Talk to the reader directly.

### Imperative verb, action first
- ✅ "Click **Register product**."
- ❌ "The user must click the register-product button located in the top-right corner of the screen."

### One action per paragraph (in step-by-steps)
Don't stack three clicks into one sentence. Each paragraph: short, one action, one screenshot if needed.

### Short sentences
The breath test: if the sentence doesn't fit in one breath, break it.

### Address the question before the screenshot, not after
- ✅ "If you've never registered a product before, start in the **Catalog** tab:" → [screenshot]
- ❌ [screenshot] → "Above you can see the Catalog tab, which is located..."

### Don't explain the obvious
Don't say "click the blue button on the right that says Save". The reader is looking at the screenshot. Say "Click **Save**".

### Bold for button and screen names
Anything the user sees written in the interface becomes **bold**. Helps scanning.

## Standard structures

### Step-by-step article

```markdown
# How to [infinitive verb] [object]

One sentence explaining when and why you do this. Two sentences max.

## Before you start

Prerequisites as short bullets. Skip this section if there are none.

## Step by step

### 1. [Short action]

Text introducing the step (1-2 sentences).

![image description](assets/step-01.webp)

Closing text if needed (1 sentence).

### 2. [Short action]
...

## Done!

A short closing sentence. Link to the next related article.

## Didn't work?

A list of 2-3 common problems + link to the troubleshooting article.
```

### Error / troubleshooting article

```markdown
# Message: "[exact error text]"

## What happened

1-2 sentences explaining, no jargon.

## Why it happened

The real cause. Use `recon.md` to be precise.

## How to fix it

Numbered steps. Short. Each one ends with the expected result.

## Still not working?

Bullet with support channels and what to send (logs, IDs).
```

### Conceptual / "what is" article

```markdown
# What is [concept]

A one-sentence definition. As if you were explaining it to someone in an elevator.

## How it works, in practice

A concrete example, with real names. No "imagine that..."

## When to use it

Bullets with use cases.

## When NOT to use it

Limitations. Important. Saves a support ticket later.

## Next steps

Links to related how-to articles.
```

## Explaining errors — the formula

**What happened → Why it happened → What to do.** In that order. No technical jargon.

### Bad example
> "Error 429: Rate limit exceeded. The user has hit the per-minute request limit of their plan's API."

### Good example
> ## What happened
> You generated a lot of images in a short time and the app had to pause for a few seconds.
>
> ## Why it happened
> Every plan has a limit on how many images can be generated per minute, so the service stays fast for everyone.
>
> ## What to do
> 1. Wait one minute and try again — most of the time, that's it.
> 2. If it happens often, it may be time to upgrade. See the limits in [Plans and quotas](link).

## Limits users HATE discovering later

Any time you document a feature, **list limits and formats up front**, before the step-by-step. A surprise in the middle of the tutorial generates a support ticket.

Examples:

- "Maximum image size: 5 MB."
- "Accepted formats: JPG, PNG, WEBP. HEIC from iPhone is converted automatically."
- "You can register up to 100 products at a time."
- "Available on the Pro plan and above."

## Tone

- Direct, not fawning. No "How great that you're here learning about our amazing product!"
- Empathetic on errors. "This usually happens when..." instead of "You did it wrong."
- Confident. No "maybe", "probably", "we believe that".
- No emoji in articles (unless the product itself uses emoji in the UI — then quote it).

## Article cover

Every cover follows the same format: a screenshot of the feature's main screen, with no annotations, at 1280x640. `screenshot-conventions.md` has the details.

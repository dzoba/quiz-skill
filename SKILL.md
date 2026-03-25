---
name: quiz
description: Quiz the user on recent unmerged code changes in their repository to verify understanding. Use this skill whenever the user wants to review, understand, or test their knowledge of recent code changes — especially after AI-generated code. Trigger on phrases like "quiz me", "test my understanding", "do I understand this code", "review changes with me", "what did the AI write", or any request to verify comprehension of recent diffs. Also trigger when a user seems uncertain about changes they're about to merge.
---

# Quiz: Test Your Understanding of Recent Code Changes

You're a code instructor who creates targeted multiple-choice quizzes about unmerged changes in the user's repository. The goal is to help humans genuinely understand code that was written for them (often by AI) before they merge it. This matters because merging code you don't understand creates a maintenance trap — you can't debug, extend, or explain it later.

## How it works

### Step 1: Find the changes

Determine the base branch and get the diff:

```bash
# Try main first, fall back to master
git diff main...HEAD 2>/dev/null || git diff master...HEAD
```

If the user isn't on a feature branch (they're on main/master), fall back to uncommitted changes:
```bash
git diff HEAD
```

If there's still nothing, try the last N commits:
```bash
git diff HEAD~5...HEAD
```

If you truly can't find any unmerged changes, tell the user and ask them to specify what they'd like to be quizzed on.

### Step 2: Analyze the diff

Read through the changes and identify:
- New functions/methods and what they do
- Design patterns or architectural choices
- State management and data flow
- Error handling strategies
- Edge cases and boundary conditions
- Interactions between changed files
- Non-obvious logic (the stuff that would confuse someone reading it for the first time)

Focus on the parts that matter most — the core logic and design decisions, not boilerplate or import statements.

### Step 3: Generate questions

Create questions at three difficulty levels:

**Comprehension** (what does this code do?)
- "What does the `processQueue` function return when the queue is empty?"
- "Which component is responsible for validating user input?"

**Reasoning** (why was it done this way?)
- "Why does `fetchData` use a retry with exponential backoff instead of a simple retry?"
- "What's the advantage of using a Map here instead of a plain object?"

**Prediction** (what would happen if...?)
- "If `maxRetries` were set to 0, what would happen when the API returns a 500?"
- "What would break if you removed the `useCallback` wrapper around `handleSubmit`?"

Aim for a mix across these levels. The number of questions defaults to 5, but the user can request more or fewer via `/quiz <number>`.

### Step 4: Present the quiz

Present questions **one at a time** using the `AskUserQuestion` tool. This gives the user a clickable selection UI instead of making them type answers.

For each question:

1. **Use `AskUserQuestion`** with the question text and 4 options (A through D). Make the wrong answers plausible — they should represent common misconceptions or reasonable-but-incorrect interpretations, not obviously silly options. Use the `label` field for the short answer (e.g., "A: The function returns null") and `description` for additional context if needed. Set `header` to something like "Q1 of 5".

2. **After the user selects an answer**, explain:
   - Whether they got it right
   - **Why** the correct answer is correct, referencing the specific code (include file paths and line context)
   - If they got it wrong, explain why their choice was a reasonable mistake and what the key distinction is
   - A brief "the bigger picture" note connecting this to the overall design when relevant

3. Then immediately present the next question via `AskUserQuestion`. Keep the flow moving — don't ask "ready for the next one?"

### Step 5: Score summary

After all questions, show:
- Score: X/N correct
- A brief assessment:
  - **All correct**: "You've got a solid understanding of these changes. Ship it."
  - **Most correct (>70%)**: "Good grasp overall. You might want to re-read [specific area where they struggled]."
  - **Some correct (40-70%)**: "There are some gaps worth filling before you merge. I'd suggest reviewing [specific files/concepts]."
  - **Few correct (<40%)**: "It might be worth walking through these changes more carefully. Want me to explain any of the concepts in more detail?"

Don't be patronizing — the whole point is that the user is being responsible by checking their understanding. Frame everything as collaborative.

## Tone

Be direct and conversational, like a colleague quizzing you at a whiteboard. Not a professor, not a tutor — a peer who's making sure you're solid before you ship. Keep explanations concise but complete. Reference actual code, not abstractions.

## Arguments

The skill accepts an optional number argument: `/quiz 3` for 3 questions, `/quiz 10` for 10. Default is 5. If the user passes something that isn't a number, interpret it as a topic filter (e.g., `/quiz auth` to focus questions on authentication-related changes).

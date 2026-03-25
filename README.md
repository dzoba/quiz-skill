# Quiz Skill for Claude Code

A skill that quizzes you on recent unmerged code changes in your repository. Helps you verify you actually understand code before you merge it -- especially useful when AI wrote it for you.

## What it does

Run `/quiz` and it will:

1. Find your unmerged changes (diff against main/master, uncommitted changes, or recent commits)
2. Generate multiple-choice questions about the code -- what it does, why it was designed that way, and what would happen if you changed it
3. Present questions one at a time, wait for your answer, then explain the correct answer with references to the actual code
4. Give you a score and point you to areas worth reviewing

## Installation

### Option 1: Clone into your skills directory

```bash
git clone https://github.com/dzoba/quiz-skill.git ~/.claude/skills/quiz
```

### Option 2: Add as a project skill

```bash
cd your-project
git clone https://github.com/dzoba/quiz-skill.git .claude/skills/quiz
```

## Usage

```
/quiz        # 5 questions (default)
/quiz 3      # 3 questions
/quiz 10     # 10 questions
/quiz auth   # focus questions on auth-related changes
```

## Example

```
> /quiz 3

Question 1 of 3 (Comprehension)

What does the `splitName` function return when given "Robert Downey Jr."?

  A) { firstName: "Robert Downey Jr.", lastName: "" }
  B) { firstName: "Robert", lastName: "Downey Jr." }
  C) { firstName: "Robert Downey", lastName: "Jr." }
  D) { firstName: "Robert", lastName: "Downey" }

Your answer?

> B

Correct. The suffix detection in PersonNode.tsx identifies "Jr." as a
known suffix and groups it with the preceding name...
```

# Example: Full Blog Pipeline

Complete example of the personal blog pipeline from interview to publication.

---

## Trigger

User wants to write a blog post about building AI agents.

---

## Phase 1: Interview

**Agent**: blog_interviewer (Claude Sonnet 4.5, temp 0.3)

```bash
blog_interviewer --topic "Building AI Agents"
```

**Conversation**:

**Interviewer**: What's the core idea you want to share about building agents?

**User**: That agents aren't as magical as people think. They're basically loops with tools.

**Interviewer**: That's a strong take. Who is this for - people who've never built agents, or people who've tried and struggled?

**User**: People who've read about agents and think it's complicated. I want to demystify it.

**Interviewer**: What's the simplest agent you've built that actually worked?

**User**: A file organizer. It looks at files, decides where they go, moves them. That's it. Three tools: list files, categorize, move.

**Interviewer**: Perfect example. What mistake do people make when they start building agents?

**User**: They overthink it. They want orchestration frameworks before they've written a single loop.

**Interviewer**: Last question - after reading this, what should someone be able to do?

**User**: Build a simple agent themselves. Not use a framework. Actually write the loop.

**Output**: `interview-brief.md`
```markdown
## Interview Brief

### Core Concept
Agents are simpler than they seem - essentially loops with tools, not magic.

### Target Audience
Developers who've read about agents and think it's complicated.
Want to demystify, not add complexity.

### Key Points
- Agents = loops + tools
- Simplest working example: file organizer (list, categorize, move)
- Common mistake: reaching for frameworks before writing a loop
- Goal: reader builds their own simple agent

### Examples/Stories
- File organizer agent (3 tools)
- "They overthink it" - framework before loop

### Desired Outcome
Reader can build a simple agent themselves, without a framework.
```

---

## Phase 2: Planning

**Agent**: blog_planner (GPT-5.2, temp 0.1)

```bash
blog_planner --interview-brief interview-brief.md
```

**Output**: `plan-blog.md`
```markdown
## Blog Plan

### Headline Variants
1. "Agents Aren't Magic: Build One in 50 Lines"
2. "Stop Overthinking Agents: A Loop and Three Tools"
3. "The Simplest AI Agent That Actually Works"
4. "Before You Pick a Framework, Write a Loop"
5. "Agents Are Just Loops (And That's Fine)"

### Structure (Destination-First)

**Opening**
Reader arrives at: "Oh, agents might not be as hard as I thought"
- Hook: The agent mystique
- Destination: They're loops with tools

**Section 1: The Simplest Agent**
Reader arrives at: "I could build this"
- File organizer example
- Three tools: list, categorize, move
- The loop (actual code)

**Section 2: Why This Works**
Reader arrives at: Understanding the pattern
- Tool → Decision → Action → Repeat
- When to stop
- Error handling (keep it simple)

**Section 3: When Complexity Helps**
Reader arrives at: Knowing when to add more
- Multiple agents
- Memory
- Orchestration
- "You'll know when you need it"

**Closing**
Reader arrives at: Ready to build
- Recap the pattern
- Challenge: Build one this week
- What to try next

### Internal Link Opportunities
- Previous post on tool design
- Post about Claude's tool use
```

---

## Phase 3: Drafting

**Agent**: blog_drafter (Claude Sonnet 4.5, temp 0.28)

```bash
blog_drafter --plan plan-blog.md --headline "Agents Aren't Magic: Build One in 50 Lines"
```

**Output**: `draft-blog.md` (excerpt)
```markdown
# Agents Aren't Magic: Build One in 50 Lines

I keep seeing developers freeze when they hear "AI agents." They imagine
complex orchestration, memory systems, multi-step reasoning chains. Then
they reach for a framework before writing a single line of code.

Here's the thing: the simplest useful agent is a loop with tools. That's it.

## The File Organizer

Let me show you an agent I built in an afternoon. It organizes files.
Three tools:

```python
def list_files(directory: str) -> list[str]:
    """Return files in directory."""
    return os.listdir(directory)

def categorize(filename: str) -> str:
    """Ask the model to categorize this file."""
    response = client.messages.create(
        model="claude-sonnet-4-5-20250514",
        messages=[{
            "role": "user",
            "content": f"Categorize this file: {filename}. Return: docs, images, code, or other."
        }]
    )
    return response.content[0].text.strip()

def move_file(source: str, category: str) -> None:
    """Move file to category folder."""
    dest = f"organized/{category}/{os.path.basename(source)}"
    os.makedirs(os.path.dirname(dest), exist_ok=True)
    shutil.move(source, dest)
```

The agent loop:

```python
files = list_files("messy_folder")
for file in files:
    category = categorize(file)
    move_file(file, category)
    print(f"Moved {file} to {category}")
```

That's the whole agent. No framework. No orchestration layer. A loop and
three tools.

## Why This Works

The pattern is: Tool → Decision → Action → Repeat.

The model makes one decision per iteration: what category? Then the code
acts on it. No complex state. No multi-step planning. Just a loop.

...

## When to Add Complexity

You'll know when you need more. Signs:

- The loop needs to make multiple decisions per item
- You're repeating yourself across agents
- State needs to persist between runs
- Errors need sophisticated recovery

Until then, keep it simple. Write the loop first.

## Build One This Week

Pick something annoying that you do manually. Email sorting. File naming.
Code review triage. Write a loop with tools.

You don't need a framework. You need a loop, a few tools, and an hour.

The agent mystique is overrated. The loop is underrated.
```

---

## Phase 4: Editing

**Agent**: blog_editor (Claude Opus 4.5, temp 0.15)

```bash
blog_editor --draft draft-blog.md
```

**3-Pass Edit**:

**Pass 1: Voice**
- Changed "Let me show you" to "Here's" (less formal)
- Removed "I keep seeing" intro (started with action instead)

**Pass 2: Meat**
- Added error handling example
- Strengthened "when to add complexity" with specific criteria

**Pass 3: Anti-LLM**
- Removed one em dash
- Changed "seamlessly" (caught in draft) to specific mechanism
- Verified no passive voice

**Output**: `edited-blog.md`

---

## Phase 5: Publishing

**Agent**: blog_publisher (GPT-5.1, temp 0.05)

```bash
blog_publisher --edited edited-blog.md --slug agents-arent-magic
```

**Output**: `published-blog.mdx`
```mdx
---
title: "Agents Aren't Magic: Build One in 50 Lines"
description: "The simplest useful AI agent is a loop with tools. Here's how to build one."
date: "2025-01-04"
tags: ["agents", "ai", "python", "practical"]
---

# Agents Aren't Magic: Build One in 50 Lines

Developers freeze when they hear "AI agents." They imagine complex orchestration,
memory systems, multi-step reasoning chains. Then they reach for a framework
before writing a single line of code.

Here's the thing: the simplest useful agent is a loop with tools. That's it.

## The File Organizer

Here's an agent I built in an afternoon. It organizes files. Three tools:

```python
def list_files(directory: str) -> list[str]:
    """Return files in directory."""
    return os.listdir(directory)

def categorize(filename: str) -> str:
    """Ask the model to categorize this file."""
    response = client.messages.create(
        model="claude-sonnet-4-5-20250514",
        messages=[{
            "role": "user",
            "content": f"Categorize this file: {filename}. Return: docs, images, code, or other."
        }]
    )
    return response.content[0].text.strip()

def move_file(source: str, category: str) -> None:
    """Move file to category folder."""
    dest = f"organized/{category}/{os.path.basename(source)}"
    os.makedirs(os.path.dirname(dest), exist_ok=True)
    shutil.move(source, dest)
```

The agent loop:

```python
files = list_files("messy_folder")
for file in files:
    category = categorize(file)
    move_file(file, category)
    print(f"Moved {file} to {category}")
```

That's the whole agent. No framework. No orchestration layer. A loop and
three tools.

[... rest of content ...]

---

*Build something simple this week. You don't need a framework. You need a
loop, a few tools, and an hour.*
```

---

## Quality Report

```yaml
summary: Blog post completed (1,247 words)
voice_check: PASSED
meat_check: PASSED
anti_llm_check: PASSED
word_count: 1,247 (target: 1,000-2,000)
internal_links: 2 added
```

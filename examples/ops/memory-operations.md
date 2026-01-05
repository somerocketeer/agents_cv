# Example: Memory Operations

Examples of common memory operations in Silas.

---

## Observing Patterns

### Recording a Correction

User says: "Actually, the meeting is at 3pm, not 2pm"

```javascript
// Immediately observe the correction
memory.observe("correction", {
  what_was_wrong: "Meeting time stated as 2pm",
  what_was_expected: "Meeting is at 3pm",
  category: "calendar"
})

// Update the calendar
await ops.calendar.update(event_id, { time: "15:00" })

// Response
"Got it - updated to 3pm."
```

### Recording a Time Preference

User says: "I hate morning meetings"

```javascript
memory.observe("time_preference", {
  time_of_day: "morning",
  day_of_week: null,  // all days
  preference: "avoid"
}, 0.8)  // confidence 0.8
```

### Recording Communication Style

User says: "Can you keep responses shorter?"

```javascript
memory.observe("communication", {
  preference: "concise",
  example: "User asked for shorter responses",
  context: "general"
})

// Also self-tune
memory.config.set(
  "behavior.response_length",
  "concise",
  "User prefers shorter responses"
)
```

### Recording a Person Interaction

User mentions: "I talked to Sarah about the budget"

```javascript
memory.observe("person_interaction", {
  person_name: "Sarah",
  relationship: "Product Manager",
  context: "budget discussion",
  sentiment: "neutral"
})

// Update person record
await memory.person.update("Sarah", {
  last_contact: new Date().toISOString(),
  topics: ["budget"]  // added to existing topics
})
```

---

## Searching Memory

### Lexical Search

```javascript
// Fast exact matching
const results = await memory.search("budget meeting")
```

**Use for**: Known terms, specific lookups

### Semantic Search

```javascript
// Vector similarity (conceptual)
const results = await memory.semantic("how does user like to start their day")
```

**Use for**: Conceptual questions, pattern discovery

### Hybrid Recall

```javascript
// Facts + episodic snippets
const results = await memory.recall("What's the status of Q1 planning?")
```

**Response**:
```json
{
  "facts": [
    { "predicate": "project.status", "value": "in_progress", "entity": "Q1 Planning" },
    { "predicate": "project.deadline", "value": "2025-03-31", "entity": "Q1 Planning" }
  ],
  "snippets": [
    { "content": "Dec 28: Discussed Q1 scope with Sarah...", "relevance": 0.92 },
    { "content": "Dec 20: Initial Q1 planning kickoff...", "relevance": 0.87 }
  ]
}
```

---

## CRM Operations

### Get Person

```javascript
const sarah = await memory.person.get("Sarah")
```

### Add Person

```javascript
await memory.person.add({
  name: "Alex",
  relationship: "Engineering Lead",
  contact_priority: "high",
  topics: ["architecture", "team"],
  notes: "Weekly 1:1s on Mondays"
})
```

### Find Stale Contacts

```javascript
// People not contacted in 14 days
const stale = await memory.person.stale(14)
```

### Get Follow-ups

```javascript
// People with pending follow-up actions
const followUps = await memory.person.followUps()
```

### Update Person

```javascript
await memory.person.update("Sarah", {
  last_contact: new Date().toISOString(),
  next_action: "Review Q1 proposal she sent"
})
```

---

## Context Management

### Get Current Context

```javascript
const ctx = await memory.context.all()
```

**Response**:
```json
{
  "primary_project": "Q1 Planning",
  "blocked_on": null,
  "waiting_for": ["Sarah - budget numbers"],
  "upcoming_deadlines": ["Jan 15 - Q1 proposal due"],
  "active_threads": ["budget approval", "auth refactor"],
  "parking_lot": ["look into new monitoring tool"]
}
```

### Set Context

```javascript
memory.context.set("primary_project", "Auth Refactor")
memory.context.set("blocked_on", "Waiting for security review")
```

### Add to Multi-Value Context

```javascript
memory.context.add("waiting_for", "Alex - architecture review")
memory.context.add("parking_lot", "Consider team offsite")
```

### Remove from Context

```javascript
memory.context.remove("waiting_for", "Sarah - budget numbers")
```

---

## Self-Tuning

### View Current Config

```javascript
const config = await memory.config.all()
```

**Response**:
```json
{
  "memory.half_life_days": 14,
  "learning.min_pattern_confidence": 0.3,
  "behavior.ask_clarifying_questions": true,
  "behavior.proactive_suggestions": true
}
```

### Adjust Config

```javascript
// User says "you forget things too fast"
memory.config.set(
  "memory.half_life_days",
  21,
  "User indicated memories decaying too quickly"
)

// User says "stop asking so many questions"
memory.config.set(
  "behavior.ask_clarifying_questions",
  false,
  "User prefers inference over questions"
)
```

---

## Learning Cycles

### Pattern Review

```javascript
// Trigger pattern analysis
const patterns = await memory.learn.patterns()
```

**Response**:
```json
{
  "patterns": [
    {
      "pattern": "User prefers morning focus time",
      "confidence": 0.85,
      "evidence_count": 12
    },
    {
      "pattern": "User checks email after standup",
      "confidence": 0.72,
      "evidence_count": 8
    }
  ]
}
```

### Confirm Pattern

```javascript
// User confirms a pattern
memory.learn.confirm("User prefers morning focus time")
```

### Memory Stats

```javascript
const stats = await memory.stats()
```

**Response**:
```json
{
  "total_events": 1247,
  "total_facts": 89,
  "total_people": 12,
  "patterns_confirmed": 15,
  "oldest_memory": "2024-06-15",
  "health": "good"
}
```

---

## Notes

- Corrections are highest signal - always observe them
- Don't observe every mundane interaction
- Use semantic search for conceptual questions
- Self-tune config based on user feedback
- Run pattern confirmation in weekly reviews

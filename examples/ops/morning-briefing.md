# Example: Morning Briefing

Complete example of Silas running a morning briefing.

---

## Trigger

User runs `/morning` command or starts a new session in the morning.

---

## Execution

### Step 1: Load Memory

```javascript
const state = await memory.load()
// Returns: { time, focus, stats }
```

**Response**:
```json
{
  "time": "2025-01-04T08:30:00-08:00",
  "focus": {
    "primary_project": "Q1 Planning",
    "blocked_on": null,
    "waiting_for": ["Sarah - budget numbers"]
  },
  "stats": {
    "people": 12,
    "patterns": 47
  }
}
```

### Step 2: Get Calendar

```javascript
const calendar = await ops.calendar.today()
```

**Response**:
```json
{
  "events": [
    { "time": "09:00", "title": "Team standup", "duration": 15 },
    { "time": "10:00", "title": "Q1 Planning with Sarah", "duration": 60 },
    { "time": "14:00", "title": "1:1 with Alex", "duration": 30 }
  ],
  "focus_time": [
    { "start": "11:00", "end": "13:30", "type": "deep_work" }
  ]
}
```

### Step 3: Get Todos (Grouped by Energy)

```javascript
const todos = await ops.todos.list()
const grouped = groupByEnergy(todos)
```

**Response**:
```json
{
  "quick_win": [
    { "title": "Review PR #234", "priority": "high" },
    { "title": "Reply to vendor email", "priority": "medium" }
  ],
  "creative": [
    { "title": "Draft project proposal", "priority": "high" }
  ],
  "deep_work": [
    { "title": "Implement auth refactor", "priority": "medium" }
  ],
  "admin": [
    { "title": "Submit expense report", "priority": "low" }
  ]
}
```

### Step 4: Check Stale Contacts

```javascript
const stale = await memory.person.stale(14)
```

**Response**:
```json
[
  { "name": "Jordan", "last_contact": "2024-12-15", "days_ago": 20 },
  { "name": "Casey", "last_contact": "2024-12-18", "days_ago": 17 }
]
```

### Step 5: Check Follow-ups

```javascript
const followUps = await memory.person.followUps()
```

**Response**:
```json
[
  { "name": "Sarah", "action": "Get budget numbers", "due": "today" }
]
```

---

## Output

```markdown
Good morning. Here's your day:

**Calendar**
- 09:00 Team standup (15 min)
- 10:00 Q1 Planning with Sarah (1 hr) - waiting on her budget numbers
- 14:00 1:1 with Alex (30 min)

**Focus Time**: 11:00-13:30 available for deep work

**Today's Priorities**

Quick Wins (between meetings):
- Review PR #234
- Reply to vendor email

Deep Work (use your focus block):
- Implement auth refactor

**Heads Up**
- Jordan (20 days), Casey (17 days) - might be worth a check-in
- Sarah's budget numbers due today for Q1 meeting

Anything specific you want to tackle first?
```

---

## Notes

- Silas doesn't narrate what it's loading ("Loading memory...")
- Briefing is concise, not exhaustive
- Energy types match time of day and calendar
- Stale contacts only mentioned if actionable
- Ends with open question, not prescriptive

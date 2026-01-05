# Example: Meeting Preparation

Complete example of Silas preparing for a meeting with delegation to silas-meeting-prep.

---

## Trigger

```bash
/prep "Q1 Planning with Sarah"
```

---

## Execution

### Step 1: Identify Meeting

```javascript
const calendar = await ops.calendar.today()
const meeting = calendar.events.find(e => e.title.includes("Sarah"))
// { time: "10:00", title: "Q1 Planning with Sarah", duration: 60 }
```

### Step 2: Get Person Context

```javascript
const sarah = await memory.person.get("Sarah")
```

**Response**:
```json
{
  "name": "Sarah",
  "relationship": "Product Manager",
  "contact_priority": "high",
  "topics": ["Q1 planning", "budget", "roadmap"],
  "communication_style": "Direct, likes data",
  "last_contact": "2024-12-28",
  "next_action": "Get budget numbers",
  "notes": "Prefers morning meetings, always on time"
}
```

### Step 3: Delegate to silas-meeting-prep

```javascript
Task({
  subagent_type: "silas-meeting-prep",
  description: "Prep for Q1 planning with Sarah",
  prompt: `
    Meeting: Q1 Planning with Sarah
    Time: Today 10:00 (1 hour)

    Person Context:
    - Role: Product Manager
    - Style: Direct, likes data
    - Topics: Q1 planning, budget, roadmap
    - Pending: Get budget numbers from her

    Gather:
    1. Recent Jira tickets related to Q1 planning
    2. Prior meeting notes with Sarah
    3. Current project status
    4. Any blockers or dependencies

    Return:
    - Context summary
    - Suggested agenda
    - Key questions to ask
    - Data points to have ready
  `
})
```

### Step 4: silas-meeting-prep Executes

**Jira Query**:
```javascript
const tickets = await atlassian.jira.search({
  jql: 'project = "Q1" AND updated >= -14d ORDER BY updated DESC',
  maxResults: 10
})
```

**Memory Search**:
```javascript
const priorNotes = await memory.search("meeting Sarah Q1")
const projectStatus = await memory.recall("Q1 planning current status")
```

**Output from silas-meeting-prep**:
```markdown
## Meeting Prep: Q1 Planning with Sarah

### Context
- Last met: Dec 28 (discussed initial Q1 scope)
- She's waiting on final budget approval from finance
- 3 Q1 tickets updated this week: AUTH-123, API-456, PERF-789

### Key Status
- Auth refactor: 60% complete, on track
- API improvements: Blocked on auth refactor
- Performance work: Not started, waiting for Q1 confirmation

### Suggested Agenda
1. Budget status (5 min) - get numbers from her
2. Priority confirmation (15 min) - auth → API → perf still correct?
3. Timeline review (20 min) - any shifts needed?
4. Dependencies/blockers (10 min)
5. Next steps (10 min)

### Questions to Ask
- Has finance approved the Q1 budget?
- Any new priorities from leadership?
- Are there team capacity concerns I should know about?

### Data to Have Ready
- Auth refactor progress: 60% (JIRA: AUTH-123)
- Estimated completion: Jan 20
- API work can start: Jan 21 (dependent on auth)
```

### Step 5: Present to User

```markdown
Ready for your 10:00 with Sarah.

**Quick Context**
Last met Dec 28. She's been waiting on finance for budget approval. Three Q1 tickets updated this week.

**Suggested Agenda**
1. Budget status (get her numbers)
2. Priority confirmation (auth → API → perf)
3. Timeline review
4. Dependencies
5. Next steps

**Key Questions**
- Budget approved yet?
- New priorities from leadership?
- Team capacity concerns?

**Status to Reference**
- Auth refactor: 60%, on track for Jan 20
- API work: Can start Jan 21

Want me to adjust anything before the meeting?
```

---

## Notes

- Silas delegates the heavy lifting to silas-meeting-prep
- silas-meeting-prep has access to Jira, memory, and can do deep research
- Silas presents a concise summary, not the full research
- Offers to adjust (not prescriptive)
- Person context (communication style) informs the prep

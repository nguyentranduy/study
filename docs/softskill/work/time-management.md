# Quản lý thời gian

Quản lý thời gian (Time Management) là kỹ năng quan trọng giúp developer làm việc hiệu quả, meet deadlines và maintain work-life balance.

## Tại sao Time Management quan trọng?

- **Meet deadlines** - Deliver đúng hẹn
- **Reduce stress** - Không bị overwhelmed
- **Better quality** - Có thời gian làm kỹ
- **Work-life balance** - Không phải OT liên tục
- **Career growth** - Được đánh giá cao

---

## Estimation

Ước lượng thời gian là một trong những skills khó nhất.

### Tại sao estimate sai?

```
Common mistakes:
• Optimistic bias - "Chắc 2 ngày là xong"
• Ignore unknowns - Không tính thời gian research
• Forget overhead - Meetings, code review, testing
• Scope creep - Requirements thay đổi
• Dependencies - Chờ người khác
```

### Estimation Techniques

#### 1. Break Down Tasks

```
❌ "Implement user authentication" - 5 days

✅ Break down:
   • Research auth options - 0.5 day
   • Setup JWT library - 0.5 day
   • Implement login endpoint - 1 day
   • Implement register endpoint - 1 day
   • Password reset flow - 1 day
   • Write tests - 1 day
   • Code review & fixes - 0.5 day
   • Documentation - 0.5 day
   ─────────────────────────────
   Total: 6 days
```

#### 2. Add Buffer

```
Rule of thumb:
• Simple task: +20% buffer
• Medium task: +50% buffer
• Complex/unknown: +100% buffer

Example:
Estimate: 3 days
Complexity: Medium
With buffer: 3 × 1.5 = 4.5 days → Round to 5 days
```

#### 3. Use Historical Data

```
Track actual time vs estimates:
┌──────────────────┬──────────┬────────┬───────┐
│ Task             │ Estimate │ Actual │ Ratio │
├──────────────────┼──────────┼────────┼───────┤
│ API endpoint     │ 2 days   │ 3 days │ 1.5x  │
│ Bug fix          │ 0.5 day  │ 1 day  │ 2x    │
│ UI component     │ 1 day    │ 1 day  │ 1x    │
└──────────────────┴──────────┴────────┴───────┘

Average ratio: 1.5x
→ Multiply future estimates by 1.5
```

#### 4. Three-Point Estimation

```
O = Optimistic (best case)
M = Most likely
P = Pessimistic (worst case)

Estimate = (O + 4M + P) / 6

Example:
O = 2 days (everything goes smoothly)
M = 4 days (normal case)
P = 8 days (major issues)

Estimate = (2 + 4×4 + 8) / 6 = 4.3 days
```

---

## Prioritization

### Eisenhower Matrix

```
                    URGENT              NOT URGENT
              ┌─────────────────┬─────────────────┐
              │                 │                 │
   IMPORTANT  │    DO FIRST     │    SCHEDULE     │
              │                 │                 │
              │ • Production bug│ • Learning      │
              │ • Deadline today│ • Refactoring   │
              │ • Client meeting│ • Documentation │
              │                 │                 │
              ├─────────────────┼─────────────────┤
              │                 │                 │
NOT IMPORTANT │    DELEGATE     │    ELIMINATE    │
              │                 │                 │
              │ • Some meetings │ • Social media  │
              │ • Some emails   │ • Unnecessary   │
              │ • Admin tasks   │   meetings      │
              │                 │                 │
              └─────────────────┴─────────────────┘
```

### MoSCoW Method

Cho prioritizing features/tasks:

| Priority | Meaning | Example |
|----------|---------|---------|
| **Must** | Critical, non-negotiable | Core functionality |
| **Should** | Important but not critical | Nice-to-have features |
| **Could** | Desirable if time permits | UI improvements |
| **Won't** | Not this time | Future enhancements |

### Daily Prioritization

```
Morning routine:
1. Review calendar - meetings, deadlines
2. Check messages - urgent items
3. List tasks for today
4. Prioritize: What MUST be done today?
5. Block time for deep work

Example daily list:
┌─────────────────────────────────────────┐
│ TODAY'S PRIORITIES                      │
├─────────────────────────────────────────┤
│ 🔴 Must: Fix payment bug (2h)           │
│ 🔴 Must: Sprint planning meeting (1h)   │
│ 🟡 Should: Review PR #123 (30m)         │
│ 🟡 Should: Write unit tests (2h)        │
│ 🟢 Could: Refactor UserService (1h)     │
└─────────────────────────────────────────┘
```

---

## Focus & Deep Work

### The Problem

```
Typical developer day:
09:00 - Start working
09:15 - Slack notification
09:20 - Back to work
09:35 - Meeting
10:30 - Back to work
10:45 - Email
11:00 - Colleague asks question
11:15 - Back to work
11:30 - Lunch

→ Only 1 hour of actual coding!
```

### Deep Work

Deep Work = Focused, uninterrupted work on cognitively demanding tasks.

```
Benefits:
• Produce better quality work
• Learn faster
• Get more done in less time
• Feel more satisfied
```

### Techniques

#### 1. Time Blocking

```
Calendar:
┌─────────────────────────────────────────┐
│ 09:00 - 11:00  Deep Work (coding)       │
│                [No meetings, no Slack]  │
├─────────────────────────────────────────┤
│ 11:00 - 12:00  Meetings, emails         │
├─────────────────────────────────────────┤
│ 12:00 - 13:00  Lunch                    │
├─────────────────────────────────────────┤
│ 13:00 - 15:00  Deep Work (coding)       │
├─────────────────────────────────────────┤
│ 15:00 - 16:00  Code review, meetings    │
├─────────────────────────────────────────┤
│ 16:00 - 17:00  Admin, planning tomorrow │
└─────────────────────────────────────────┘
```

#### 2. Pomodoro Technique

```
┌─────────────────────────────────────────┐
│         POMODORO TECHNIQUE              │
├─────────────────────────────────────────┤
│                                         │
│  🍅 Work: 25 minutes                    │
│  ☕ Short break: 5 minutes              │
│                                         │
│  Repeat 4 times, then:                  │
│  🌴 Long break: 15-30 minutes           │
│                                         │
└─────────────────────────────────────────┘

Tools:
• Pomofocus.io
• Forest app
• Simple timer
```

#### 3. Minimize Distractions

```
Do:
• Turn off notifications during deep work
• Use "Do Not Disturb" mode
• Close unnecessary tabs
• Use website blockers
• Wear headphones (signal to others)

Don't:
• Check Slack every 5 minutes
• Keep email open
• Have social media tabs open
• Sit in high-traffic area
```

---

## Dealing with Interruptions

### Types of Interruptions

| Type | Example | Strategy |
|------|---------|----------|
| **Urgent** | Production down | Handle immediately |
| **Important** | Colleague needs help | Schedule time |
| **Nice-to-have** | "Quick question" | Batch or defer |
| **Unnecessary** | Random chat | Politely decline |

### Handling "Quick Questions"

```
Scenario: Colleague asks "Do you have 5 minutes?"

Option 1: Defer
"I'm in the middle of something. Can I come to you at 3pm?"

Option 2: Async
"Can you Slack me the details? I'll get back to you within an hour."

Option 3: Time-box
"I have 5 minutes now. What's up?"
(Set timer, stick to it)
```

### Protect Your Time

```
Strategies:
• Block "focus time" on calendar
• Set Slack status: "Deep work until 11am"
• Use headphones as "do not disturb" signal
• Batch similar tasks (emails, reviews)
• Say no to unnecessary meetings
```

---

## Meetings

### Meeting Efficiency

```
Before accepting:
• Is my presence necessary?
• Is there an agenda?
• Can this be an email/Slack?

During:
• Start and end on time
• Stick to agenda
• Take notes
• Assign action items

After:
• Share notes
• Follow up on action items
```

### Decline Meetings Politely

```
"Thanks for including me. I don't think I need to be in this one - 
can you share the notes with me afterward?"

"I have a conflict at that time. Can you send me a summary 
or recording?"

"My plate is full this week. Is there someone else who can 
represent the team?"
```

---

## Avoiding Burnout

### Signs of Burnout

```
Physical:
• Constant fatigue
• Frequent illness
• Sleep problems

Emotional:
• Cynicism
• Feeling detached
• Lack of motivation

Professional:
• Decreased productivity
• Missing deadlines
• Making more mistakes
```

### Prevention

```
Daily:
• Take regular breaks
• Don't skip lunch
• End work at reasonable time
• Exercise, sleep well

Weekly:
• Have at least 1 day completely off
• Do non-work activities
• Spend time with family/friends

When overwhelmed:
• Talk to manager
• Reprioritize tasks
• Ask for help
• Take time off if needed
```

### Work-Life Balance

```
Boundaries:
• Set work hours and stick to them
• Don't check work email after hours
• Have a shutdown ritual

Shutdown ritual example:
1. Review what was done today
2. Write tomorrow's priorities
3. Close all work apps
4. Say "Shutdown complete" (seriously, it helps!)
```

---

## Tools

### Task Management

| Tool | Best for |
|------|----------|
| **Jira** | Team projects, Agile |
| **Trello** | Visual boards, simple projects |
| **Todoist** | Personal tasks |
| **Notion** | All-in-one workspace |

### Time Tracking

| Tool | Best for |
|------|----------|
| **Toggl** | Simple time tracking |
| **RescueTime** | Automatic tracking |
| **Clockify** | Free, team tracking |

### Focus

| Tool | Best for |
|------|----------|
| **Forest** | Gamified focus |
| **Freedom** | Block distracting sites |
| **Brain.fm** | Focus music |

---

## Weekly Review

Dành 30 phút cuối tuần để review:

```
Weekly Review Template:
─────────────────────────────────────────

1. WINS
   • What did I accomplish?
   • What am I proud of?

2. CHALLENGES
   • What didn't go well?
   • What blocked me?

3. LEARNINGS
   • What did I learn?
   • What would I do differently?

4. NEXT WEEK
   • Top 3 priorities
   • Meetings to prepare for
   • Deadlines coming up

5. SELF-CARE
   • How was my energy level?
   • Did I take enough breaks?
   • What do I need to recharge?

─────────────────────────────────────────
```

---

## Checklist

### Daily

- [ ] Plan day in the morning
- [ ] Identify top 3 priorities
- [ ] Block time for deep work
- [ ] Take regular breaks
- [ ] End work at reasonable time

### Weekly

- [ ] Weekly review
- [ ] Plan next week
- [ ] Clear inbox
- [ ] Review goals progress

### When Overwhelmed

- [ ] List all tasks
- [ ] Prioritize ruthlessly
- [ ] Delegate or defer non-essential
- [ ] Talk to manager if needed
- [ ] Take a break

---

## Tài nguyên

### Books

- "Deep Work" - Cal Newport
- "Getting Things Done" - David Allen
- "The 7 Habits of Highly Effective People" - Stephen Covey
- "Atomic Habits" - James Clear

### Articles

- [Maker's Schedule, Manager's Schedule](http://www.paulgraham.com/makersschedule.html) - Paul Graham
- [The Pomodoro Technique](https://francescocirillo.com/pages/pomodoro-technique)

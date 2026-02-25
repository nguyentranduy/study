# Làm việc nhóm

Làm việc nhóm (Teamwork) là kỹ năng thiết yếu trong môi trường phát triển phần mềm hiện đại, nơi hầu hết các dự án đều được thực hiện bởi team.

## Tại sao Teamwork quan trọng?

- **Dự án phức tạp** - Không ai có thể làm một mình
- **Diverse skills** - Mỗi người có thế mạnh khác nhau
- **Knowledge sharing** - Học hỏi lẫn nhau
- **Better solutions** - Nhiều góc nhìn = giải pháp tốt hơn
- **Faster delivery** - Chia việc = hoàn thành nhanh hơn

---

## Agile/Scrum Team

### Các vai trò

```
┌─────────────────────────────────────────────────────────┐
│                    SCRUM TEAM                           │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐ │
│  │   Product   │  │    Scrum    │  │   Development   │ │
│  │    Owner    │  │    Master   │  │      Team       │ │
│  └─────────────┘  └─────────────┘  └─────────────────┘ │
│                                                         │
│  - Define backlog  - Facilitate    - Developers        │
│  - Prioritize      - Remove blocks - QA/Testers        │
│  - Accept/Reject   - Coach team    - Designers         │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### Scrum Events

| Event | Mục đích | Thời gian |
|-------|----------|-----------|
| **Sprint Planning** | Lên kế hoạch sprint | 2-4 giờ |
| **Daily Standup** | Sync hàng ngày | 15 phút |
| **Sprint Review** | Demo kết quả | 1-2 giờ |
| **Retrospective** | Cải thiện process | 1-2 giờ |

### Daily Standup

Mỗi người trả lời 3 câu hỏi:

```
1. Hôm qua tôi đã làm gì?
   "Hôm qua em đã hoàn thành API authentication và 
   fix bug #123 về login timeout."

2. Hôm nay tôi sẽ làm gì?
   "Hôm nay em sẽ implement user profile endpoint 
   và viết unit tests."

3. Có blockers nào không?
   "Em đang chờ design cho profile page từ team design.
   Có thể anh A follow up giúp em được không?"
```

!!! tip "Tips cho Daily Standup"
    - Đúng giờ, ngắn gọn
    - Focus vào progress, không phải details
    - Raise blockers sớm
    - Discussions chi tiết để sau meeting

---

## Communication

### Async Communication

Phần lớn communication trong team là async (không real-time).

**Channels:**

| Tool | Khi nào dùng |
|------|--------------|
| **Slack/Teams** | Quick questions, updates, casual |
| **Email** | Formal, external, documentation |
| **Jira/Trello** | Task updates, requirements |
| **Confluence/Notion** | Documentation, decisions |
| **GitHub/GitLab** | Code-related discussions |

**Best Practices:**

```
❌ "Hey, are you there?"
   (Đợi reply rồi mới hỏi tiếp)

✅ "Hey! I have a question about the payment API. 
   When a user cancels an order, should we refund 
   immediately or wait for admin approval? 
   Context: ticket #456"
   (Đầy đủ context, người nhận có thể reply khi rảnh)
```

### Sync Communication

Meetings, calls, pair programming.

**Effective Meetings:**

```
Before:
- Có agenda rõ ràng
- Invite đúng người
- Share materials trước

During:
- Start đúng giờ
- Stick to agenda
- Take notes
- Assign action items

After:
- Share meeting notes
- Follow up on action items
```

### Written Communication

```
Slack message tốt:
─────────────────────────────────────────
@team Heads up: I'll be deploying v2.3.0 
to staging at 3pm today.

Changes:
• New payment gateway integration
• Bug fixes for checkout flow

Please test your features after deployment.
Any concerns, let me know before 2pm.
─────────────────────────────────────────

Email tốt:
─────────────────────────────────────────
Subject: [Action Required] API Changes - Breaking Changes in v3.0

Hi team,

We're releasing API v3.0 next Monday with breaking changes.

What's changing:
1. Authentication: JWT instead of API keys
2. Response format: camelCase instead of snake_case

Action needed:
- Update your client code by Friday
- Test with staging environment

Documentation: [link]
Questions? Reply to this email or ping me on Slack.

Thanks,
[Name]
─────────────────────────────────────────
```

---

## Code Review

Code review là một trong những activities quan trọng nhất của teamwork.

### Tại sao Code Review?

- **Catch bugs** sớm
- **Knowledge sharing** - Học từ code của nhau
- **Consistency** - Đảm bảo code style nhất quán
- **Better design** - Nhiều góc nhìn
- **Mentoring** - Senior giúp junior improve

### Làm Reviewer

```
Checklist:
□ Code có đúng requirements không?
□ Logic có correct không?
□ Có edge cases nào bị miss?
□ Code có readable không?
□ Có tests không? Tests có đủ không?
□ Có security issues không?
□ Performance có OK không?
```

**Cách comment:**

```
❌ "This is wrong"
❌ "Why did you do this?"
❌ "This is stupid"

✅ "Consider using Optional here to handle null case"
✅ "What do you think about extracting this to a separate method 
   for better readability?"
✅ "Nice solution! One suggestion: we could use Stream API here 
   to make it more concise"
```

**Levels of feedback:**

```
🔴 Blocker: "This will cause a null pointer exception in production"
🟡 Suggestion: "Consider renaming this variable for clarity"
🟢 Nitpick: "Minor: extra whitespace on line 42"
💚 Praise: "Great use of the builder pattern here!"
```

### Làm Author

```
Before submitting:
□ Self-review code của mình
□ Run tests locally
□ Write clear PR description
□ Keep PR small (<400 lines)
□ Link to ticket/issue

PR Description template:
─────────────────────────────────────────
## What
Brief description of changes

## Why
Context and motivation

## How
Technical approach

## Testing
How to test these changes

## Screenshots (if UI changes)
[images]
─────────────────────────────────────────
```

**Responding to feedback:**

```
❌ "It works fine"
❌ "I don't agree" (without explanation)
❌ Ignore comments

✅ "Good point! Fixed in commit abc123"
✅ "I considered that, but chose this approach because [reason]. 
   What do you think?"
✅ "Thanks for catching this! I've added a test case for it"
```

---

## Pair Programming

Hai developers làm việc cùng nhau trên một máy.

### Roles

```
┌─────────────────────────────────────────┐
│           PAIR PROGRAMMING              │
├─────────────────────────────────────────┤
│                                         │
│  Driver          Navigator              │
│  ────────        ─────────              │
│  • Viết code     • Review real-time     │
│  • Focus details • Think big picture    │
│  • Keyboard      • Suggest, guide       │
│                                         │
│  Swap roles every 15-30 minutes         │
│                                         │
└─────────────────────────────────────────┘
```

### Khi nào Pair?

- **Onboarding** - Senior pair với new member
- **Complex problems** - Cần nhiều góc nhìn
- **Knowledge transfer** - Spread expertise
- **Stuck** - Khi một người bị stuck

### Tips

```
Do:
• Communicate constantly
• Take breaks
• Be patient
• Share keyboard
• Celebrate wins together

Don't:
• Grab keyboard without asking
• Zone out as navigator
• Be dismissive of ideas
• Pair for too long (max 4 hours)
```

---

## Conflict Resolution

Conflicts là bình thường trong team. Quan trọng là cách xử lý.

### Types of Conflicts

| Type | Ví dụ | Approach |
|------|-------|----------|
| **Technical** | Chọn framework A hay B | Data-driven discussion |
| **Process** | Workflow, conventions | Team discussion, vote |
| **Personal** | Communication style | 1-1 conversation |
| **Priority** | Feature A hay B trước | Escalate to PO/Manager |

### Resolution Steps

```
1. ACKNOWLEDGE
   "Tôi hiểu bạn prefer approach A vì [reasons]"

2. UNDERSTAND
   "Có thể bạn giải thích thêm về concern với approach B?"

3. FIND COMMON GROUND
   "Chúng ta đều muốn solution tốt nhất cho users"

4. PROPOSE SOLUTIONS
   "Có thể chúng ta thử prototype cả hai và compare?"

5. AGREE ON NEXT STEPS
   "OK, vậy chúng ta sẽ spike 2 ngày rồi decide"
```

### Technical Disagreements

```
Scenario: Bạn và đồng nghiệp không đồng ý về architecture

❌ "Cách của bạn sai"
❌ Escalate ngay lên manager
❌ Làm theo ý mình không nói

✅ Steps:
1. Discuss pros/cons của mỗi approach
2. Consider: performance, maintainability, team familiarity
3. Prototype nếu cần
4. Involve team/tech lead nếu không resolve được
5. Document decision và reasoning
```

---

## Remote Team Collaboration

### Challenges

- Timezone differences
- Communication gaps
- Isolation
- Distractions at home

### Best Practices

```
Communication:
• Over-communicate (better too much than too little)
• Use video calls khi có thể
• Document everything
• Set clear expectations

Availability:
• Define core hours (overlap time)
• Update status (available/busy/away)
• Respect others' time zones

Tools:
• Slack/Teams for chat
• Zoom/Meet for calls
• Miro/FigJam for collaboration
• Notion/Confluence for docs
```

---

## Building Trust

Trust là foundation của teamwork hiệu quả.

### Cách build trust

```
1. RELIABILITY
   • Deliver on commitments
   • Meet deadlines
   • Follow through on action items

2. TRANSPARENCY
   • Share progress openly
   • Admit mistakes
   • Ask for help when needed

3. RESPECT
   • Value others' opinions
   • Give credit
   • Be inclusive

4. COMPETENCE
   • Do quality work
   • Continuously improve
   • Share knowledge
```

### Trust-building activities

- Team lunches/coffee chats
- Non-work Slack channels (#random, #pets)
- Team retrospectives
- Pair programming
- Knowledge sharing sessions

---

## Checklist

### Daily

- [ ] Attend standup, share updates
- [ ] Respond to messages timely
- [ ] Update task status
- [ ] Help teammates when asked

### Weekly

- [ ] Review PRs của teammates
- [ ] Share learnings
- [ ] Participate in team meetings
- [ ] Give/receive feedback

### Monthly

- [ ] Retrospective participation
- [ ] 1-1 với manager
- [ ] Knowledge sharing session
- [ ] Team bonding activity

## Tiếp theo

- [Quản lý thời gian](time-management.md)

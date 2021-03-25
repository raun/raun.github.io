---
layout: post
title: Framework to decide priority of Bugs
categories: [Agile, Framework]
---
## Why?
Softwares are supporting multiple use-cases today, have multiple stakeholders. Software bugs are common and we have not reached a point where we can avoid bugs altogether in our software development process. From the eyes of stakeholders, their bug is the most critical one and they want a resolution as quickly as possible.

Prioritizing bugs can be a confusing in such an environment. Having a clear framework/policy can help here. Not only it helps build alignment across teams on the priority of bug but also give them clarity on when will your bug get the attention of a Software Engineer.

## The Framework
The priority of a bug can be decided based on two factors:

- **Functionality/Productivity Impaired by the Bug**: From the perspective of other internal teams
- **Business Impact of the Bug**: From the perspective of end-user of the product

Based on this you can define the levels of Business Impact and Productivity impact. In my past experiences, I kept 5 levels and defined the threshold accordingly. 

| Priority Level | Functionality Impaired | Business/User Impact | Expected Resolution time | Expected First Response |
| -------------- | ---------------------- | -------------------- | ------------------------ | ----------------------- |
| P0 | Multiple orgs/teams are blocked on the issue | More than 10% of users are impacted due to the issue| 24 Hours | 2 Hours |
| P1 | An org/team is blocked due to the issue | 5-10% of users are impacted due to the issue | 3 days | 24 Hours |
| P2 | A subset of a team(more than 2 people) are blocked due to the issue | 0-5% of users are impacted due to the issue | 1 week | 36 Hours |
| P3 | An individual is blocked due to the issue | Not impacting users right now but might start impacting them in the near future(next couple of weeks). The functionality is working but giving a degraded experience to the user. | 2 weeks | 36 Hours |
| P4 | No-one is blocked due to the issue | Not impacting users right now and will not impact in the near future | Have to be discussed with Stakeholder | During backlog grooming |


*Your company and team could be different so please feel free to define more or less number of level. The thresholds and expectation can also differ based on the environment.*

**Definitions**

1. **First Response time**: This is the time to have the first response on the JIRA issue. It could be as simple as “I am looking into it”. 
2. **Resolution time**: The time taken to achieve either of these items is called Resolution time for an issue:
	- Mitigate the issue so that the user impact can be minimized(fewer users are impacted or hotfix to resolve the problem for now)
	- Provide alternative paths so that individuals/teams/orgs can get unblocked. And make sure those are acceptable to them.

As you would have noticed, both the factors are not mutually exclusive. So we start from the top of the list and evaluate both the factors in light of the bug, if either one of the conditions are met, then first match is final priority of the bug.

Let me explain by some examples:

Issue 1

- **Functionality Impaired**: Blocking Individual
- **User Impact**: Impacting 7% of users

Priority: **P1**

Issue 2

- **Functionality Impaired**: Blocking a team
- **User Impact**: Not impacting user now but will impact them in the near future

Priority: **P1**

Issue 3

- **Functionality Impaired**: No-one is blocked
- **User Impact**: Impacting 20% of users

Priority: **P0**

These example should have clarified the framework even further.

## Conclusion
The framework takes into account the user impact and how a bug is impacting someone within the team. It tries to balance these 2 forces and help you focus on the fixing bugs. Provide common ground to decide correct priority of the bug.

Do let me know how you felt about the framework on the email provided [here](https://raun.github.io/about/)


**Check out my other interesting posts**
- [How to think about company culture?](https://raun.github.io/tech/culture/2021/03/10/how-to-think-about-company-culture/)
- [Understanding Concurrency?](https://raun.github.io/web-service/threads/asyncio/python/2021/02/25/Understanding-concurrency/)


# SRE Practice and Incident Management

## 目录

- [Introduction to Incident](#introduction-to-incident)
- [Incident Stages](#incident-stages)
- [Good and Bad Incident Management](#good-and-bad-incident-management)
- [Getting Ready for On-Call](#getting-ready-for-on-call)
- [Incident Detection](#incident-detection)
- [Incident Response](#incident-response)
- [Incident Recovery](#incident-recovery)
- [Incident Analysis: Learning from Failure](#incident-analysis-learning-from-failure)
- [Incident Management](#incident-management)
- [HOT Incident Management](#hot-incident-management)
- [SlackOps](#slackops)
- [Useful Links](#useful-links)

## Introduction to Incident

### What is an Incident?

An incident is an event that causes or is likely to cause disruption or a reduction in the quality of service, which requires an emergency response.

### Is it an Incident?

Examples:

- Free disk space of a web server drops from 10GB to 10MB within 5 minutes.
- Autoscaling function of a production cluster is broken (not scaling or over-scaling).
- Jenkins CI/CD pipeline is broken.
- Weird UI issues after a release.

## Incident Stages

1. Detect
2. Respond
3. Recover
4. Learn
5. Improve

## Good and Bad Incident Management

### Good Practices

- Clear communication channels
- Well-defined roles and responsibilities
- Regular training and drills
- Blameless postmortems
- Continuous improvement

### Bad Practices

- Poor communication
- Lack of preparation and training
- Blame culture
- Ineffective post-incident analysis

## Getting Ready for On-Call

### Life as an On-Call Engineer

- Handle outages and operations affecting teams.
- Available to perform operations on production systems within minutes.
- Acknowledge page and triage problems.
- Handle lower priority and non-production system events during business hours.

**Reference:** [A Day on Call in a DevOps Team](https://dzone.com/articles/a-day-on-call-in-a-devopsteam)

### Balance in Quality and Quantity

- Day/Night shifts
- Number of engineers on-call
- Primary/Secondary on-call roles
- Incident priority (non-prod and lower priorities)

### Compensation

- Time-in-lieu
- Cash
- Proportion of overall salary
- Time-based compensation

### Best Practices

- Clear escalation policy
- Well-defined incident management procedure
- Blameless postmortem culture
- Identify recurring issues and prioritize work

## Incident Detection

### Detection Methods

- Customers (Jira Service Desk, Service Now, Zendesk)
- Employees
- Monitoring systems
  - Metrics Monitoring
  - Error & Exception Monitoring
  - Application Performance Monitoring
  - Uptime & Synthetic Monitoring

**Goal:** Detect incidents before customers do.

## Incident Response

### Steps for Proper Response

1. Acknowledge the alert (within 15 minutes)
2. Gather missing information and fill in the gaps
3. Validate the incident (is this an actual incident?)
4. Set up a communication channel
5. Gather related persons

### Gathering Information

- What is the incident behavior?
- What time did it occur? Is it a new behavior?
- Error message or screenshot?
- What environment? How many customers affected?
- Is it an internal system? Any vendor involved?
- Any public discussion (Twitter, etc.)?

### Bug or Low Priority Incident?

- Bugs usually don’t require urgent mitigation and can be fixed in a sprint as a high priority task.
- Low priority incidents require urgent mitigation during working hours but cannot be planned into a sprint.

### Incident Communication

#### Internal Communication

- Email
- Slack
- ServiceNow thread

#### External Communication

- Status Page

**Examples:**

- Dropbox
- Facebook

## Incident Recovery

### Troubleshooting

- **Troubleshooting is learnable.**
- Skills and knowledge of the system are essential.

### Problem Report

- Expected behavior
- Actual behavior
- Steps to reproduce

### Triage

- Assess severity (all-hands-on-deck?)
- Stop the bleeding (divert traffic, drop traffic if data is corrupt, degrade the system)

### Diagnose

- Simplify and reduce the problem
- Ask "what," "where," and "why"
- Determine what touched the system last
- Build specific tools to diagnose the system

### Test and Treat

- Have a list of possible causes
- Ideal tests should have mutually exclusive alternatives
- Prioritize high-probability causes
- Avoid misleading tests and those with side effects

### Cure

- Find probable causal factors
- Systems are complex; fixes may not be reproducible in live production
- Implement fixes and perform postmortem analysis

## Incident Analysis: Learning from Failure

### Postmortem

- A written record of an incident
- Impact, actions taken, root cause(s), and follow-up actions
- **When to write a postmortem:**
  - User-visible downtime
  - Data loss
  - On-call engineer intervention (rollback, etc.)
  - Long resolution time
  - Monitoring failure, user-reported issue

### 5-Why Example

1. Why did the user complain about not being able to use the "Send email" feature?
   - Because there was a bug in the latest release.
2. Why was there a bug in the latest release?
   - Because we didn’t test this scenario.
3. Why didn’t we test this scenario?
   - Because we only tested the features developed in the current sprint and didn't do regression tests on other features.
4. Why didn’t you test all the other features?
   - Because "Send email" was created before the current sprint, and it's impractical to test all features for every release.
5. Why is it impractical to test all features?
   - Because our application is vast, and performing regression testing on every feature manually would require excessive time and delay the process.

### Formal Review and Publication

- Ensure key incident data is collected
- Complete impact assessments
- Identify deep root causes
- Create an appropriate action plan and prioritize resulting bug fixes
- Share outcomes with relevant stakeholders

### No Blame Culture

- Assume everyone is good
- Focus on fixing the system, not blaming people

## Incident Management

### Unmanaged Incident Story

**What went wrong:**

- Sharp focus on the technical problem
- Poor communication
- Freelancing

### Elements of Incident Management

- Separation of Responsibilities:
  - Incident Command
  - Operational Work
  - Communication
  - Planning (longer term)
- A Recognized Command Post (war room, IRC)
- Live Incident State Document
- Clear, live handoff

**Reference:** [Google SRE Incident Document](https://sre.google/sre-book/incident-document/)

### Managed Incident Story

- On-call engineer Mary detects an issue with 1-2 datacenters down.
- Sabrina takes command and sends an email.
- Sabrina updates a live incident document with "Users not impacted".
- As more datacenters go down, Sabrina loops in Joseph and Robin volunteers to help.
- Sabrina instructs Mary on actions, and Robin advises against reversion.
- Sabrina arranges a handoff at 5pm with a briefing, and the next day, the issue is mitigated.

### Incident Management Best Practices

- **Prioritize:** Stop the bleeding, then find the root cause.
- **Prepare:** Develop procedures before incidents happen.
- **Trust:** Give full autonomy within assigned roles to all incident participants.
- **Introspect:** When panicking, ask for help.
- **Consider alternatives:** Is it right? Should I do it differently?
- **Practice:** Routinely use the process until it becomes second nature.
- **Change roles:** Ensure everyone practices different roles.

## HOT Incident Management

- Jira Service Desk
- ServiceNow
- Zendesk
- Rootly

## SlackOps

- Connect your alert system with your chat
- Set intelligent thresholds for alerts
- Set up a separate room for each major incident
- Invite multiple teams
- Save chat transcripts

## Useful Links

- **Five Whys:** [MindTools Article](https://www.mindtools.com/pages/article/newTMC_5W.htm)
- **Feature Flags:** [LaunchDarkly Blog](https://launchdarkly.com/blog/what-are-feature-flags)
- **Chrome Runtime Performance:** [Chrome DevTools](https://developer.chrome.com/docs/devtools/evaluate-performance/)

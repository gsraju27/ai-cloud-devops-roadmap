# DeployU GitHub Marketing - Weekly Operations Playbook

> **Repository:** https://github.com/gsraju27/ai-cloud-devops-roadmap
> 
> **Goal:** Grow the repo to 10,000+ stars through consistent weekly updates and social media promotion.

---

## Table of Contents

1. [Weekly Schedule](#weekly-schedule)
2. [Task 1: Interview Questions](#task-1-interview-questions)
3. [Task 2: Cheatsheets](#task-2-cheatsheets)
4. [Task 3: Events & Hackathons](#task-3-events--hackathons)
5. [Task 4: Social Media Posts](#task-4-social-media-posts)
6. [Weekly Report Template](#weekly-report-template)
7. [Content Formats](#content-formats)
8. [Rules & Guidelines](#rules--guidelines)
9. [File Locations](#file-locations)

---

## Weekly Schedule

| Day | Task | Deadline |
|-----|------|----------|
| Monday AM | Tasks assigned in Slack | - |
| Thursday | Interview Questions PR due | 5:00 PM |
| Thursday | Cheatsheet PR due | 5:00 PM |
| Friday | Events PR due | 12:00 PM |
| Friday | Social media posts due | 3:00 PM |
| Monday | Weekly report due | 10:00 AM |

---

## Task 1: Interview Questions

### Assigned to: `[NAME]`

### What to do

Add **10 new interview questions** with detailed answers.

### Steps

1. Pick ONE technology from this list:
   - Docker
   - Kubernetes
   - Terraform
   - AWS
   - Azure
   - GCP
   - Python
   - JavaScript
   - React
   - Node.js

2. Find 10 real interview questions from these sources ONLY:
   - https://glassdoor.com → search "[technology] interview questions"
   - https://reddit.com/r/devops
   - https://reddit.com/r/cscareerquestions
   - Your own interview experience

3. Write each question with:
   - The question (1-2 sentences)
   - A scenario (1-2 sentences of context)
   - The answer (3-5 bullet points)
   - Why interviewers ask this (1 sentence)

4. Add questions to the correct file:
   - Docker → `interview-questions/devops/docker.md`
   - Kubernetes → `interview-questions/devops/kubernetes.md`
   - AWS → `interview-questions/cloud/aws.md`
   - Python → `interview-questions/programming/python.md`

5. Create a Pull Request with this title format:
   ```
   Add 10 [Technology] interview questions - Week [X]
   ```

6. Share the PR link in Slack when done.

### Question Format

Copy this format exactly:

```markdown
### Q: [Question text here]

**Scenario:** [Brief context - 1-2 sentences]

**Answer:**
1. [First point]
2. [Second point]
3. [Third point]
4. [Fourth point]
5. [Fifth point]

**Why interviewers ask this:**
[1 sentence explaining what skill this tests]

---
```

### Example

```markdown
### Q: A container keeps restarting. How do you debug it?

**Scenario:** Your team deployed a new service but the pod shows CrashLoopBackOff status.

**Answer:**
1. Check logs: `kubectl logs <pod-name> --previous`
2. Describe pod: `kubectl describe pod <pod-name>`
3. Check events for OOM or image pull errors
4. Verify environment variables and config maps
5. Test container locally with same image

**Why interviewers ask this:**
Tests your systematic debugging approach and Kubernetes troubleshooting skills.

---
```

---

## Task 2: Cheatsheets

### Assigned to: `[NAME]`

### What to do

Create **1 new cheatsheet** OR update an existing one.

### Steps

1. Pick ONE topic from this list:
   - Docker CLI
   - Kubernetes Commands
   - Terraform Commands
   - Git Commands
   - Linux Commands
   - AWS CLI
   - Python Basics
   - SQL Queries

2. Create the cheatsheet with:
   - Title and short description
   - Commands grouped by category
   - Clear examples for each command
   - Keep it under 100 lines

3. Add to file: `cheatsheets/[topic].md`

4. Create a Pull Request with this title format:
   ```
   Add [Topic] cheatsheet - Week [X]
   ```

5. Share the PR link in Slack when done.

### Cheatsheet Format

Copy this format exactly:

```markdown
# [Technology] Cheatsheet

Quick reference for [brief description].

---

## [Category 1]

| Command | Description | Example |
|---------|-------------|---------|
| `command` | What it does | `example usage` |
| `command` | What it does | `example usage` |

---

## [Category 2]

| Command | Description | Example |
|---------|-------------|---------|
| `command` | What it does | `example usage` |
| `command` | What it does | `example usage` |

---
```

### Example

```markdown
# Docker CLI Cheatsheet

Quick reference for common Docker commands.

---

## Container Management

| Command | Description | Example |
|---------|-------------|---------|
| `docker ps` | List running containers | `docker ps` |
| `docker ps -a` | List all containers | `docker ps -a` |
| `docker stop <id>` | Stop a container | `docker stop abc123` |
| `docker rm <id>` | Remove a container | `docker rm abc123` |
| `docker logs <id>` | View container logs | `docker logs -f abc123` |

---

## Image Management

| Command | Description | Example |
|---------|-------------|---------|
| `docker images` | List all images | `docker images` |
| `docker pull <image>` | Download an image | `docker pull nginx:latest` |
| `docker build -t <name> .` | Build an image | `docker build -t myapp:v1 .` |
| `docker rmi <image>` | Remove an image | `docker rmi nginx:latest` |

---
```

---

## Task 3: Events & Hackathons

### Assigned to: `[NAME]`

### What to do

Find **3-5 upcoming tech events** or hackathons and add them to the repo.

### Steps

1. Find events from these sources ONLY:
   - https://devpost.com/hackathons
   - https://mlh.io/seasons/2026/events
   - https://meetup.com (search "devops" or "cloud" + your city)
   - https://linkedin.com/events
   - https://konfhub.com (for India events)

2. Focus on events related to:
   - Cloud (AWS, Azure, GCP)
   - DevOps
   - AI/ML
   - Open Source
   - Coding competitions

3. Add events to file: `resources/events.md`

4. Create a Pull Request with this title format:
   ```
   Add events for [Month] - Week [X]
   ```

5. Share the PR link in Slack when done.

### Event Format

Copy this format exactly:

```markdown
### [Event Name]

- **Date:** [Month Day, Year]
- **Location:** [City, Country / Online]
- **Type:** [Hackathon / Conference / Meetup / Workshop]
- **For:** [Beginners / Experienced / All levels]
- **Topics:** [List 2-3 main topics]
- **Link:** [Registration URL]

---
```

### Example

```markdown
### AWS Community Day Bangalore 2026

- **Date:** February 15, 2026
- **Location:** Bangalore, India
- **Type:** Conference
- **For:** All levels
- **Topics:** AWS, Cloud Architecture, Serverless
- **Link:** https://awscommunitydays.com/bangalore

---

### HackTheCloud 2026

- **Date:** March 1-3, 2026
- **Location:** Online
- **Type:** Hackathon
- **For:** All levels
- **Topics:** Cloud, DevOps, Infrastructure
- **Link:** https://devpost.com/hackathons/hackthecloud

---
```

---

## Task 4: Social Media Posts

### Assigned to: `[NAME]`

### What to do

Create and post **1 LinkedIn post** and **1 Twitter/X post** about the repo.

### Steps

1. Post from your **PERSONAL account** (not company account)

2. Post about this week's updates:
   - New questions added
   - New cheatsheet added
   - New events added

3. Always include the repo link: `https://github.com/gsraju27/ai-cloud-devops-roadmap`

4. Take screenshots of both posts

5. Share screenshot + post links in Slack when done

### LinkedIn Post Templates

**Template 1: New Questions**
```
Just added 10 new [Technology] interview questions to our free DevOps interview prep repo.

These are real questions asked at companies like Google, Amazon, and Microsoft.

Topics covered:
• [Topic 1]
• [Topic 2]
• [Topic 3]

Free and open source - no signup needed.

Check it out: https://github.com/gsraju27/ai-cloud-devops-roadmap

#DevOps #CloudComputing #InterviewPrep #TechJobs #[Technology]
```

**Template 2: New Cheatsheet**
```
New [Technology] cheatsheet just dropped!

Perfect quick reference for:
• [Use case 1]
• [Use case 2]
• [Use case 3]

Part of our free interview prep repo with 320+ questions.

Grab it here: https://github.com/gsraju27/ai-cloud-devops-roadmap

#DevOps #[Technology] #CheatSheet #LearnInPublic
```

**Template 3: Weekly Update**
```
This week's updates to our free DevOps interview prep repo:

✅ 10 new [Technology] questions
✅ [Topic] cheatsheet
✅ [X] upcoming events added

Total: 330+ questions and growing!

Star it if you find it useful: https://github.com/gsraju27/ai-cloud-devops-roadmap

#DevOps #CloudComputing #OpenSource #InterviewPrep
```

### Twitter/X Post Templates

**Template 1:**
```
Added 10 new [Technology] interview questions to our free repo.

Real questions from FAANG interviews.

⭐ github.com/gsraju27/ai-cloud-devops-roadmap

#DevOps #CloudComputing #InterviewPrep
```

**Template 2:**
```
New [Technology] cheatsheet 📝

Commands you'll actually use:
→ [Command 1]
→ [Command 2]
→ [Command 3]

Free: github.com/gsraju27/ai-cloud-devops-roadmap

#DevOps #[Technology]
```

---

## Weekly Report Template

**Submit every Monday by 10:00 AM**

Copy and fill this template:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📊 WEEKLY GITHUB UPDATE REPORT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Week: [Start Date] - [End Date]
Submitted by: [Your Name]
Date: [Today's Date]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
MY TASK:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Task type: [Interview Questions / Cheatsheet / Events / Social Media]
Status: [Completed / Partial / Not Done]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
WHAT I ADDED:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

File updated: [exact file path]
Example: interview-questions/devops/docker.md

Items added:
• [Item 1]
• [Item 2]
• [Item 3]

Pull Request link: [paste PR URL here]
PR Status: [Merged / Pending Review]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SOCIAL MEDIA (if assigned):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

LinkedIn post: [URL]
Twitter/X post: [URL]
Screenshots attached: [Yes / No]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
BLOCKERS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[None / Describe any problems you faced]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Rules & Guidelines

### ✅ YOU MUST DO

| Rule | Why |
|------|-----|
| Always create a Pull Request (PR) | So changes can be reviewed |
| Never push directly to main branch | Main branch is protected |
| Use clear PR titles | Easy to track what was added |
| Write content in your own words | Avoid plagiarism issues |
| Follow the exact format shown above | Keeps repo consistent |
| Submit weekly report every Monday | Track progress |
| Share PR links as proof | Verify work was done |
| Post from personal social accounts | More authentic engagement |

### ❌ YOU MUST NOT DO

| Rule | Why |
|------|-----|
| Do NOT copy-paste from websites | Plagiarism, legal issues |
| Do NOT push directly to main | Branch is protected, will fail |
| Do NOT change files outside your task | Stay in your lane |
| Do NOT delete existing content | Only add, don't remove |
| Do NOT add promotional content | Keep repo educational |
| Do NOT skip the weekly report | Need to track progress |
| Do NOT post from company accounts | Personal accounts only |
| Do NOT use AI-generated content without review | Quality control |

---

## File Locations

### Interview Questions

| Category | File Path |
|----------|-----------|
| Docker | `interview-questions/devops/docker.md` |
| Kubernetes | `interview-questions/devops/kubernetes.md` |
| Terraform | `interview-questions/devops/terraform.md` |
| Jenkins | `interview-questions/devops/jenkins.md` |
| GitHub Actions | `interview-questions/devops/github-actions.md` |
| AWS | `interview-questions/cloud/aws.md` |
| Azure | `interview-questions/cloud/azure.md` |
| GCP | `interview-questions/cloud/gcp.md` |
| Python | `interview-questions/programming/python.md` |
| JavaScript | `interview-questions/programming/javascript.md` |
| React | `interview-questions/programming/react.md` |
| Node.js | `interview-questions/programming/nodejs.md` |
| PostgreSQL | `interview-questions/databases/postgresql.md` |
| MongoDB | `interview-questions/databases/mongodb.md` |
| Redis | `interview-questions/databases/redis.md` |

### Cheatsheets

| Topic | File Path |
|-------|-----------|
| Docker CLI | `cheatsheets/docker-cli.md` |
| Kubernetes | `cheatsheets/kubernetes.md` |
| Terraform | `cheatsheets/terraform.md` |
| Git | `cheatsheets/git.md` |
| Linux | `cheatsheets/linux.md` |
| AWS CLI | `cheatsheets/aws-cli.md` |

### Resources

| Content | File Path |
|---------|-----------|
| Events & Hackathons | `resources/events.md` |
| Communities | `resources/communities.md` |
| Books | `resources/books.md` |

### Company Guides

| Company Type | File Path |
|--------------|-----------|
| Google | `companies/product-based/google.md` |
| Amazon | `companies/product-based/amazon.md` |
| Microsoft | `companies/product-based/microsoft.md` |
| TCS | `companies/service-based/tcs.md` |
| Infosys | `companies/service-based/infosys.md` |
| Wipro | `companies/service-based/wipro.md` |

---

## How to Create a Pull Request

### Step 1: Create a new branch

```bash
git checkout main
git pull origin main
git checkout -b add-docker-questions-week1
```

### Step 2: Make your changes

Edit the appropriate file and add your content.

### Step 3: Commit your changes

```bash
git add .
git commit -m "Add 10 Docker interview questions - Week 1"
```

### Step 4: Push to GitHub

```bash
git push origin add-docker-questions-week1
```

### Step 5: Create the PR

1. Go to: https://github.com/gsraju27/ai-cloud-devops-roadmap
2. Click "Compare & pull request" (yellow banner)
3. Title: `Add 10 Docker interview questions - Week 1`
4. Description: List what you added
5. Click "Create pull request"
6. Copy the PR link and share in Slack

---

## Questions?

- Ask in `#github-marketing` Slack channel
- Tag `@[Manager Name]` for urgent issues

---

*Last updated: January 2026*

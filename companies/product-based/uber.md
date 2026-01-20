# Uber Interview Questions

Complete preparation guide for Software Engineering roles at Uber. This guide covers the end-to-end interview process from Online Assessments to the final Leadership and Architecture rounds.

---

## Interview Process Overview

Uber's selection process is rigorous, focusing on problem-solving, architectural thinking, and writing production-grade code. The process typically consists of an initial assessment followed by 4 to 5 interview rounds.

| Stage | Format | Duration | Focus |
|-------|--------|----------|-------|
| **Online Assessment (OA)** | Coding Platform | 70 min | 4 problems ranging from basic array/string logic to complex 2D DP. |
| **Initial Coding (BPS)** | Technical | 60 min | Algorithmic problem-solving with a heavy focus on code quality. |
| **Algorithms & Data Structures** | Technical | 60 min | Core DSA: DP, Graphs, and complex logic with multiple follow-ups. |
| **Depth in Specialization** | Technical | 60 min | Writing production-ready, modular code (OOP/SOLID principles). |
| **System Design & Architecture** | Whiteboard | 60 min | High-level and Low-level design of scalable systems. |
| **Collaboration & Leadership** | Managerial | 75 min | Behavioral questions and deep-dive into past project decisions. |

---

## Online Assessment (OA)

The OA is the first filter and typically involves solving **4 questions in 70 minutes**:
- **Basics:** 2 problems covering string manipulation and array logic.
- **Medium:** 1 problem involving trees or graphs.
- **Advanced:** 1 complex problem, often involving 2D Dynamic Programming or intricate matrix manipulation.

**Strategy:** Focus on passing all test cases for the first three problems quickly. Even a brute-force solution for the fourth problem that passes partial test cases can help you move to the next stage.

---

## Technical Rounds: Algorithmic & Coding BPS

Uber values not just the solution, but how the code is written.

### Initial Coding Assessment
This round is often an elimination round. 
- **The Focus:** You will be given an algorithmic problem (typically of medium complexity). 
- **Evaluation Criteria:** 
    - **Code Quality:** Use of good naming conventions and modular structure.
    - **Clarity:** Your ability to explain the logic as you code.
    - **Clean Code:** Avoiding "hacky" solutions in favor of maintainable logic.

### Core Algorithms and Data Structures
This round dives deeper into computer science fundamentals.
- **Complexity:** Expect problems involving **Dynamic Programming (DP)**, **Graphs (Dijkstra, BFS/DFS)**, and **2D Grid/Matrix** manipulation.
- **Follow-ups:** Interviewers often start with a base version of a problem and add constraints to increase the complexity. Be prepared to adapt your logic quickly.

---

## Depth in Specialization

This round evaluates your ability to write code that is ready for a production environment. Unlike a pure algorithmic round, the focus here is on **Software Engineering best practices**.

**Key Evaluation Pillars:**
- **Single Responsibility Principle (SRP):** Ensuring each class and method has one clear purpose.
- **Object-Oriented Programming (OOP):** Effective use of classes, interfaces, and objects.
- **Extensibility:** How easy is it to add a new feature to your code?
- **Naming & Readability:** Using descriptive names and proper indentation to make your solution maintainable.

---

## System Design & Architecture

Depending on the role, the focus might shift between backend infrastructure or frontend application architecture. Use frameworks like **RADIO** (for UI-heavy roles) or standard **HLD/LLD** for backend roles.

**Key Components to Cover:**
1. **Requirements Gathering:** Always start by clarifying functional and non-functional requirements.
2. **API & Data Design:** Define the communication protocols and data schemas first.
3. **High-Level Design:** Draw the major components (Load Balancers, Services, Caches, Databases).
4. **Deep Dive:** Explain specific optimizations like sharding, consistent hashing, or rendering strategies.
5. **Bottlenecks:** Identify single points of failure and propose scaling solutions.

---

## Collaboration & Leadership (Behavioral)

The final round is usually conducted by a Manager and focuses on your "soft skills" and past experiences.

**1. Deep Dive into Past Work:**
- Be prepared to walk through a complex system you built previously.
- You must be able to justify your **design decisions** and explain why you chose one technology over another.
- Be honest about what could have been done better.

**2. Behavioral Questions (STAR Method):**
- Prepare stories using the **STAR** (Situation, Task, Action, Result) format.
- **Topics:** Conflict resolution, leading/mentoring others, handling technical debt, and collaborating with cross-functional teams.

---

## Success Checklist & Tips

- [ ] **Talk while you code:** Interviewers value your thought process. If you are stuck, thinking out loud helps them give you a hint.
- [ ] **Clarify Everything:** Never write a single line of code until you have clarified all constraints (e.g., input size, memory limits, edge cases).
- [ ] **Tooling Readiness:** Uber often uses tools like **Miro** for diagramming and **Zoom** for communication. Familiarize yourself with them.
- [ ] **Ask Insightful Questions:** When asked "Do you have any questions for us?", ask about their biggest technical challenges or team culture.

---

## Preparation Resources

- [LeetCode Blind 75](https://leetcode.com/discuss/general-discussion/460515/blind-75-leetcode-questions) - For core DSA concepts.
- [System Design Primer](https://github.com/donnemartin/system-design-primer) - For architectural foundations.
- [Grind 75/NeetCode 150](https://neetcode.io/practice) - For structured pattern-based practice.
- **Recommended Study:** Explore high-scale architecture blogs (Uber Engineering Blog) to understand the problems they solve.

---

*Want to build real cloud projects for your resume? [Try DeployU](https://deployu.ai?ref=github) - Practice deployments on real infrastructure.*

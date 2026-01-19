# Infosys Interview Questions

Complete preparation guide for Infosys campus hiring, InfyTQ, and Specialist Programmer (SP) roles.

---

## Interview Process & Roles

Infosys typically hires for three main roles based on performance in their assessments (InfyTQ, HackWithInfy, or Campus Recruitment).

| Role | Package | Focus |
|------|---------|-------|
| **System Engineer (SE)** | ₹3.6 LPA | Basic Coding, Aptitude, Communication |
| **Digital Specialist Engineer (DSE)** | ₹6.25 LPA | Advanced Coding, DBMS, SDLC |
| **Specialist Programmer (SP)** | ₹9.5 LPA | Competitive Coding, Architecture, System Design |

### The Selection Path
1. **InfyTQ/HackWithInfy:** Competitive assessments for DSE/SP roles.
2. **Technical Interview:** Usually **only one** technical round for SP roles.
3. **Interview Format:** Can be offline; be prepared to code on **pen and paper** (no IDE).

---

## Online Assessment (OA) Breakdown

### Specialist Programmer (L3) Focus
- **Topics:** Primarily Graphs, Dynamic Programming, and Advanced Data Structures.
- **Difficulty:** LeetCode Medium to Hard.
- **Key Strategy:** Focus on Graph traversal (BFS/DFS), Shortest Path (Dijkstra), and String algorithms.

---

## Technical Interview Questions

### Data Structures & Algorithms (Must-Know)

#### 1. Standard Problem List
These problems are frequently asked across DSE and SP interviews:
- **Rotten Oranges:** Standard BFS problem (Very Frequent).
- **Longest Common Subsequence (LCS):** Classic DP.
- **Next Smallest Palindrome:** String/Math logic.
- **Building a Heap:** How to implement `heapify` from an array.
- **Cycle Detection:** Both in Directed Graphs and Singly Linked Lists.
- **Lexicographically Smallest Equivalent String:** Disjoint Set Union (DSU) application.
- **Two Pointers/Sliding Window:** Max sum subarrays, finding pairs.

#### 2. Pattern: Optimized Searches with Sliding Window
**Scenario:** You are building a monitoring tool for an Infosys client. You need to find the maximum sum of $k$ consecutive elements in a large array of server logs.

<details>
<summary><strong>View Answer</strong></summary>

Instead of re-calculating the sum for every subarray ($O(n \times k)$), use a **Sliding Window**:
1. Sum first $k$ elements.
2. Slide the window: `new_sum = old_sum - leftmost_element + next_element`.
3. Complexity: $O(n)$ time.

</details>

#### 3. Complexity Tip
Expect follow-up questions on **Time and Space Complexity**. Always mention the Big O notation for your proposed solution before you start writing code.

---

### System Design & Architecture (SP Level)

#### 4. Scalability Scenario: Twitter Design
**Question:** Design Twitter. How do you scale from 10 users to 1 Million+ users?

<details>
<summary><strong>View Answer</strong></summary>

- **Initial (10-1k):** Single Monolith + SQL DB.
- **Growth (1k-100k):** Add Load Balancer + Read Replicas + Caching (Redis).
- **Scale (100k-1M+):** Microservices + Database Sharding + Message Queues (Kafka) for async timeline updates + CDN for media.

</details>

---

### CS Fundamentals (Rapid Fire)

| Subject | Top Questions |
|---------|---------------|
| **OOP** | Pillars (Encapsulation, etc.), Method Overloading vs Overriding, Abstract Class vs Interface, Multiple Inheritance (Diamond Problem). |
| **DBMS** | ACID properties, Joins (Left vs Right), Truncate vs Delete vs Drop, Normalization (3NF vs BCNF), Primary Key vs Unique Key. |
| **OS** | CPU Scheduling types, Deadlock Prevention, Semaphores vs Mutex, Process Life Cycle, Virtual Memory. |
| **CN** | TCP vs UDP, OSI Layers, Session vs Socket, Connection-oriented communication. |
| **SDLC** | Waterfall vs Agile Model, SDLC Phases. |

---

### Project & Resume Discussion

Interviewers at Infosys place high importance on your projects. Be prepared for:
1. **Implementation:** "Walk me through how you implemented the authentication feature."
2. **Challenges:** "What was the biggest technical challenge you faced, and how did you debug it?"
3. **Role:** "What was your specific contribution in this team project?"
4. **Technology Choice:** "Why did you choose MongoDB over SQL for this specific project?"

---

## HR & Behavioral Questions

1. **Mysore Training:** "How do you feel about the 3-6 month intensive training at the Mysore campus?"
2. **Flexibility:** "If the project requires you to work on an older technology like Mainframe/Java 8, what would be your stance?" (Correct answer: Show learning agility).
3. **Conflict Resolution:** "Tell me about a time you had a teammate who wasn't contributing."
4. **Why Infosys?** Focus on their world-class training and status as a Top IT employer.

---

## Final Tips & Checklist

- [ ] **Pen & Paper Practice:** Dry run your code on paper.
- [ ] **SQL Queries:** Practice writing `JOIN` and `GROUP BY` queries by hand.
- [ ] **Resume Tech:** Don't list anything you haven't used; they will probe deep.
- [ ] **Clarify Reasoning:** Speak out loud while solving. They value your thought process more than a perfect syntax.

---

## Resources

- [LeetCode Infosys Curated List](https://leetcode.com/problem-list/m47xli66/)
- [Strivers A2Z DSA Sheet](https://takeuforward.org/strivers-a2z-dsa-course/strivers-a2z-dsa-course-sheet-2)
- [Java OOP Guide](../../interview-questions/programming/java.md)
- [Python Interview Questions](../../interview-questions/programming/python.md)
- [SQL Joins & Queries](../../interview-questions/databases/postgresql.md)
- [Full DevOps Roadmap](../../roadmaps/devops-engineer.md)

---

*Want to build real cloud projects for your resume? [Try DeployU](https://deployu.ai?ref=github) - Practice deployments on real infrastructure.*

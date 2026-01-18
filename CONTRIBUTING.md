# Contributing to AI, Cloud & DevOps Roadmap

First off, thank you for considering contributing! This project grows stronger with community input.

## How Can You Contribute?

### 1. Add New Interview Questions

We're always looking for real-world interview questions. The best questions are:
- Based on actual interview experiences
- Scenario-based (not just definitions)
- Include working code examples
- Have clear explanations

**Question Template:**

```markdown
### Question: [Title - should be a problem statement]

**Type:** Debugging | Practical | Architecture
**Category:** [Specific area like "Networking", "Security", etc.]
**Difficulty:** Easy | Medium | Hard

#### The Scenario

[Describe a realistic workplace situation where this problem occurs]

#### The Challenge

[What specifically needs to be solved?]

#### Solution

[Step-by-step solution with code examples]

```code
[Working code example]
```

#### Why This Works

[Explain the reasoning behind the solution]

#### Quick Check

**Question:** [Multiple choice question to test understanding]

A. [Option 1]
B. [Option 2]
C. [Option 3]  <- Correct
D. [Option 4]

<details>
<summary>See Answer</summary>

[Explanation of why the correct answer is correct]

</details>
```

### 2. Fix Errors

Found a mistake? Please open an issue or submit a PR with:
- The file path
- What's wrong
- The correct information
- Source/reference (if applicable)

### 3. Improve Explanations

If you think an explanation could be clearer:
- Open a PR with your improved version
- Explain why your version is better
- Keep the same structure/format

### 4. Add New Technologies

Want to add a new technology (e.g., Rust, Kotlin, etc.)?

1. Create a new file in the appropriate folder
2. Include at least 10 quality questions
3. Follow the existing format
4. Add to the main README table

### 5. Improve Roadmaps

Roadmaps should be:
- Realistic timelines
- Based on industry requirements
- Updated for current year
- Include practical projects

## Pull Request Process

1. **Fork** the repository
2. **Create a branch**: `git checkout -b add-rust-questions`
3. **Make your changes**
4. **Test locally** - Ensure markdown renders correctly
5. **Commit**: `git commit -m "Add Rust interview questions"`
6. **Push**: `git push origin add-rust-questions`
7. **Open a Pull Request**

### PR Title Format

```
[TYPE] Brief description

Types:
- [ADD] - Adding new content
- [FIX] - Fixing errors
- [UPDATE] - Updating existing content
- [IMPROVE] - Improving explanations

Examples:
- [ADD] Rust interview questions (12 questions)
- [FIX] Incorrect Docker command in kubernetes.md
- [UPDATE] AWS roadmap for 2026
- [IMPROVE] Better explanation for K8s networking
```

### PR Description Template

```markdown
## What does this PR do?

[Brief description]

## Type of change

- [ ] New interview questions
- [ ] Bug fix
- [ ] Content update
- [ ] Documentation improvement

## Checklist

- [ ] I've followed the existing format
- [ ] Code examples are tested/working
- [ ] Spelling and grammar are correct
- [ ] Links are valid
```

## Style Guide

### Markdown

- Use `#` for main titles, `##` for sections, `###` for subsections
- Use fenced code blocks with language specified
- Use tables for structured data
- Use `<details>` for collapsible content

### Code Examples

- Should be runnable (not pseudo-code)
- Include necessary imports
- Add comments for complex parts
- Use consistent formatting

### Tone

- Professional but approachable
- Direct and concise
- Focus on practical application
- Avoid unnecessary jargon

## Questions?

- Open an issue for discussion
- Tag it with `question` label

## Recognition

Contributors will be:
- Listed in the README (for significant contributions)
- Credited in the commit history
- Thanked publicly (if you want)

---

Thank you for making this resource better for everyone!

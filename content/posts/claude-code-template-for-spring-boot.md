---
title: "Claude Code Template for Spring Boot — A Practical Guide"
date: 2026-04-16
draft: false
tags: ["AI", "Claude Code", "Spring Boot", "Java", "Developer Productivity"]
categories: ["AI"]
summary: "How to set up and use a Claude Code template for generating high-quality Spring Boot applications — covering CLAUDE.md, skills, subagents, and best practices."
---

## Introduction

AI-powered coding assistants like **Claude Code** are reshaping how developers build applications. But here's the thing — simply launching Claude Code and typing a prompt won't get you production-quality code. The real power lies in providing Claude with a well-structured **template** that encodes your conventions, preferred tech stack, and quality expectations.

In this post, I'll walk you through a Claude Code template I've built for **Spring Boot** applications. I've published it as a ready-to-use starter repository on GitHub: [claude-ai-skill](https://github.com/jitendrar292/claude-ai-skill). You can clone it, customize the `CLAUDE.md` file, and start generating apps immediately.

We'll cover:
- Why templates matter for AI-assisted coding
- The structure of the template repository
- How to configure `CLAUDE.md`, skills, and subagents
- A practical workflow for generating a Spring Boot app

## Motivation

The Claude Code [documentation](https://code.claude.com/docs/en/best-practices) lists several best practices for getting the best results. You *can* just launch Claude and enter a prompt — but if you want the generated code to meet your expectations, following these practices is essential.

Most developers, when they start using Claude Code seriously, end up looking for a ready-made repository template that matches their tech stack. Building one from scratch takes considerable time and trial-and-error. Communities around specific technologies should maintain, improve, and share these templates — it benefits everyone.

That's why I created and maintain this template. You can use it as:
- A **ready-to-use starter** that requires only minimal changes to `CLAUDE.md`
- **Inspiration** to build your own template tailored to your team's conventions

## Source Code

The complete template is available on GitHub: [jitendrar292/claude-ai-skill](https://github.com/jitendrar292/claude-ai-skill). Clone the repo and follow along.

Here's the repository structure:

```shell
.
├── .claude
│   ├── agents
│   │   ├── code-reviewer.md
│   │   ├── devops-engineer.md
│   │   ├── docker-expert.md
│   │   ├── java-architect.md
│   │   ├── kubernetes-specialist.md
│   │   ├── security-engineer.md
│   │   ├── spring-boot-engineer.md
│   │   └── test-automator.md
│   ├── settings.local.json
│   └── skills
│       ├── README.md
│       ├── api-contract-review
│       │   └── SKILL.md
│       ├── clean-code
│       │   └── SKILL.md
│       ├── design-patterns
│       │   └── SKILL.md
│       ├── java-architect
│       │   ├── SKILL.md
│       │   └── references
│       │       ├── jpa-optimization.md
│       │       ├── reactive-webflux.md
│       │       ├── spring-boot-setup.md
│       │       ├── spring-security.md
│       │       └── testing-patterns.md
│       ├── java-code-review
│       │   └── SKILL.md
│       ├── jpa-patterns
│       │   └── SKILL.md
│       ├── logging-patterns
│       │   └── SKILL.md
│       ├── spring-boot-engineer
│       │   ├── SKILL.md
│       │   └── references
│       │       ├── cloud.md
│       │       ├── data.md
│       │       ├── security.md
│       │       ├── testing.md
│       │       └── web.md
│       └── spring-boot-patterns
│           └── SKILL.md
├── CLAUDE.md
├── README.md
└── pom.xml
```

The template has three main components: the **CLAUDE.md** file (the brain), **skills** (domain knowledge), and **agents** (specialized workers). Let's dive into each.

## The CLAUDE.md File

`CLAUDE.md` is the heart of the template. It acts as a **built-in briefing** that Claude loads at the beginning of every interaction. Think of it as a config file for AI behavior — it defines coding conventions, workflows, and quality gates that Claude consistently follows.

My `CLAUDE.md` is organized into six key sections:

```markdown
### 1. Plan Mode Default
- Enter plan mode for ANY not-trivial task (3+ steps or architectural decisions)
- Use plan mode for verification steps, not just building
- Write detailed specs upfront to reduce ambiguity

### 2. Self-Improvement Loop
- After ANY correction from the user: update `tasks/lessons.md` with the pattern
- Write rules for yourself that prevent the same mistake
- Ruthlessly iterate on these lessons until the mistake rate drops
- Review lessons at session start for a project

### 3. Verification Before Done
- Never mark a task complete without proving it works
- Diff behavior between main and your changes when relevant
- Ask yourself: "Would a staff engineer approve this?"
- Run tests, check logs, demonstrate correctness

### 4. Demand Elegance (Balanced)
- For non-trivial changes: pause and ask "is there a more elegant way?"
- If a fix feels hacky: "Knowing everything I know now, implement the elegant solution"
- Skip this for simple, obvious fixes. Don't overengineer
- Challenge your own work before presenting it

### 5. Skills Usage
- Use skills for any task that requires a capability
- Load skills from `.claude/skills/`
- Invoke skills with natural language
- Each skill is one independent capability

### 6. Subagents Usage
- Use subagents liberally to keep the main context window clean
- Load subagents from `.claude/agents/`
- For complex problems, throw more compute at it via subagents
- One task per subagent for focused execution on a given tech stack
```

Enabling **plan mode** (Section 1) is critical — it lets you review the implementation strategy before Claude starts generating code. The **self-improvement loop** (Section 2) ensures Claude learns from corrections during a session. And **verification** (Section 3) prevents the classic problem of code that looks right but fails in practice.

### Project-Specific Instructions

Below the workflow rules, I define my project conventions. These are the non-negotiable expectations for any generated code:

```markdown
## Core Principles
- **Simplicity First**: Make every change as simple as possible. Impact minimal code
- **No Laziness**: Find root causes. No temporary fixes. Senior developer standards

## Project General Instructions
- Always use the latest versions of dependencies
- Always write Java code as the Spring Boot application
- Always use Maven for dependency management
- Always create test cases for the generated code both positive and negative
- Always generate the CircleCI pipeline in the .circleci directory to verify the code
- Minimize the amount of code generated
- The Maven artifact name must be the same as the parent directory name
- Use semantic versioning for the Maven project
- Do not use the Lombok library
- Generate the Docker Compose file to run all components used by the application
- Update README.md each time you generate a new version
```

Even if some of these are covered in skills, repeating key expectations in `CLAUDE.md` reinforces them. Feel free to swap CircleCI for GitHub Actions or Jenkins — the template is yours to customize.

## Skills — Extending Claude's Knowledge

Skills are the mechanism to extend Claude's knowledge with domain-specific expertise. Each skill lives in `.claude/skills/` and focuses on one independent capability. You can verify loaded skills by running the `/skills` command inside Claude Code.

The template includes nine skills:

| Skill | Purpose |
|-------|---------|
| `api-contract-review` | API design and contract validation |
| `clean-code` | Code quality and readability standards |
| `design-patterns` | GoF and enterprise patterns |
| `java-architect` | Enterprise Java architecture with Spring Boot |
| `java-code-review` | Java-specific code review checklist |
| `jpa-patterns` | JPA/Hibernate optimization patterns |
| `logging-patterns` | Structured logging best practices |
| `spring-boot-engineer` | Full-stack Spring Boot development |
| `spring-boot-patterns` | Spring-specific design patterns |

Each skill has a header section with metadata that helps Claude understand when to invoke it:

```markdown
---
name: java-architect
description: Use when building, configuring, or debugging enterprise Java
  applications with Spring Boot, microservices, or reactive programming.
  Invoke to implement WebFlux endpoints, optimize JPA queries and database
  performance, configure Spring Security with OAuth2/JWT, or resolve
  authentication issues and async processing challenges.
metadata:
  version: "1.0.0"
  domain: language
  triggers: Spring Boot, Java, microservices, Spring Cloud, JPA, Hibernate, WebFlux
  role: architect
  scope: implementation
  output-format: code
---

# Java Architect

Enterprise Java specialist focused on Spring Boot, microservices architecture,
and cloud-native development using Java 21 LTS.

## Core Workflow
1. **Architecture analysis** — Review project structure, dependencies, Spring config
2. **Domain design** — Create models following DDD and Clean Architecture
3. **Implementation** — Build services with Spring Boot best practices
4. **Data layer** — Optimize JPA queries, implement repositories;
   run `mvn verify` to confirm query correctness
5. **Security & config** — Apply Spring Security, externalize configuration
6. **Quality assurance** — Run `mvn verify`, ensure 85%+ coverage via JaCoCo
```

The `triggers` field in metadata tells Claude Code when to automatically activate a skill based on the technologies mentioned in your prompt. The `references/` subfolder under some skills (like `java-architect` and `spring-boot-engineer`) contains additional reference documents for deeper context.

## Subagents — Specialized AI Workers

Subagents are specialized agents that focus on specific concerns. They keep the main context window clean by offloading focused tasks to dedicated agents. The template ships with eight agents:

| Agent | Role |
|-------|------|
| `spring-boot-engineer` | Core Spring Boot application code |
| `java-architect` | Architecture and design decisions |
| `code-reviewer` | Code quality and best practice review |
| `test-automator` | Test generation and coverage |
| `security-engineer` | Security hardening and vulnerability checks |
| `docker-expert` | Containerization and Docker configuration |
| `devops-engineer` | CI/CD pipelines and DevOps setup |
| `kubernetes-specialist` | K8s deployment manifests and Helm charts |

You can verify the available agents by running the `/agents` command in Claude Code.

> **Token usage note:** Subagents can significantly increase token consumption. If you want to keep costs low, rewrite Section 6 of `CLAUDE.md` to disable subagents and use only the `spring-boot-engineer` agent as your main agent:

```markdown
### 6. Subagents usage
- Load subagents from `.claude/agents/`
- Don't use subagents
- Use only the single custom agent `spring-boot-engineer` as the main agent
```

## Practical Example

After cloning the repository, navigate to its root directory and run Claude Code. Here's a sample prompt to test the template:

```
Create the application that exposes a REST API and connects to a PostgreSQL
database in the current directory. The application should have a Person entity
with an id and typical fields for each person. All REST endpoints should be
protected with JWT and OAuth2. The codebase should use Skaffold to deploy
on Kubernetes.
```

After entering this prompt, Claude Code will:

1. **Enter plan mode** — analyze requirements and create a detailed implementation strategy with token estimates
2. **Divide work into phases** — typically 8-10 phases covering models, repositories, services, controllers, security, tests, and infrastructure
3. **Generate code phase by phase** — Spring Boot application code, JPA entities, REST controllers with security
4. **Write and run tests** — unit tests and integration tests using Testcontainers
5. **Create infrastructure** — Dockerfile, Docker Compose, Kubernetes manifests, Skaffold config, and CircleCI pipeline
6. **Review for quality** — check for overengineering and adherence to best practices

A prompt like this typically generates around 3,700 lines of well-structured, tested code.

## The Recommended Workflow

Anthropic recommends a four-phase workflow for Claude Code:

1. **Explore** — Claude analyzes the codebase and understands the existing context
2. **Plan** — Creates a detailed implementation plan, estimates token usage, and waits for your approval
3. **Implement** — Generates code phase by phase, running tests along the way
4. **Commit** — Verifies everything works and commits the changes

This workflow ensures you stay in control. The plan mode gives you a chance to course-correct before any code is generated, and the verification steps ensure the output actually works.

## Best Practices

Here are the key takeaways from building and using this template:

1. **Always enable plan mode** — It prevents Claude from rushing into bad implementations and gives you a review checkpoint
2. **Keep skills focused** — One skill = one capability. Don't create monolithic skills that try to cover everything
3. **Add verification gates** — Tests, linting, and coverage checks should be part of the workflow, not an afterthought
4. **Maintain lessons learned** — The `tasks/lessons.md` file captures mistakes and patterns, making Claude better over time within a session
5. **Start with the template, then customize** — Clone the repo, adjust `CLAUDE.md` for your conventions, and iterate
6. **Be strategic with subagents** — They improve quality but cost more tokens. Start without them and add selectively

## Conclusion

A well-crafted Claude Code template transforms AI code generation from "hit or miss" to consistently high-quality output. The key ingredients are clear instructions in `CLAUDE.md`, domain-specific skills, and built-in quality verification gates.

This template is still evolving — I'm continuously comparing different configurations and optimizing for the balance between generation time, token usage, and code quality. Feel free to clone the [repository](https://github.com/jitendrar292/claude-ai-skill), try it out, and adapt it to your needs. If you have feedback or ideas for improvement, open an issue or pull request on GitHub.

---

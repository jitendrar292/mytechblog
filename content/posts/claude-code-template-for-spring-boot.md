---
title: "Claude Code Template for Spring Boot — A Practical Guide"
date: 2026-04-16
draft: false
tags: ["AI", "Claude Code", "Spring Boot", "Java", "Developer Productivity"]
categories: ["AI"]
summary: "How to set up and use a Claude Code template for generating high-quality Spring Boot applications — covering CLAUDE.md, skills, subagents, and best practices."
---

## Introduction

AI-powered coding tools like **Claude Code** are changing how we build applications. But just launching Claude Code and typing a prompt isn't enough to get production-quality code. You need a well-structured **template** that guides the AI with your conventions, tech stack, and quality expectations.

In this post, I'll walk through how to set up a Claude Code template for **Spring Boot** applications — covering the `CLAUDE.md` file, skills, subagents, and a practical workflow.

## Why Use a Template?

Without a template, Claude Code generates code based on generic assumptions. With a template, you get:

- **Consistent coding style** across all generated code
- **Your preferred tech stack** (Maven, JPA, Spring Security, etc.)
- **Automated testing** — Claude writes and runs tests
- **Infrastructure-as-code** — Dockerfiles, Kubernetes manifests, CI/CD pipelines
- **Quality gates** — code review, overengineering checks

## Repository Structure

A well-organized template looks like this:

```
.
├── .claude
│   ├── agents
│   │   ├── code-reviewer.md
│   │   ├── java-architect.md
│   │   ├── spring-boot-engineer.md
│   │   ├── security-engineer.md
│   │   ├── docker-expert.md
│   │   └── kubernetes-specialist.md
│   └── skills
│       ├── clean-code/SKILL.md
│       ├── design-patterns/SKILL.md
│       ├── java-architect/SKILL.md
│       ├── jpa-patterns/SKILL.md
│       ├── spring-boot-engineer/SKILL.md
│       └── spring-boot-patterns/SKILL.md
├── CLAUDE.md
├── pom.xml
└── README.md
```

## The CLAUDE.md File

This is the heart of the template. It acts as a **built-in briefing** that Claude loads at the start of every interaction. Here's the structure I recommend:

```markdown
### 1. Plan Mode Default
- Enter plan mode for ANY non-trivial task (3+ steps)
- Use plan mode for verification, not just building
- Write detailed specs upfront to reduce ambiguity

### 2. Self-Improvement Loop
- After ANY correction: update `tasks/lessons.md` with the pattern
- Write rules to prevent the same mistake
- Review lessons at session start

### 3. Verification Before Done
- Never mark a task complete without proving it works
- Run tests, check logs, demonstrate correctness
- Ask: "Would a staff engineer approve this?"

### 4. Demand Elegance
- For non-trivial changes: pause and ask "is there a more elegant way?"
- If a fix feels hacky: implement the elegant solution
- Don't overengineer simple fixes

### 5. Skills Usage
- Load skills from `.claude/skills/`
- Each skill is one independent capability

### 6. Subagents Usage
- Load subagents from `.claude/agents/`
- One task per subagent for focused execution
```

### Project-Specific Instructions

Add your own conventions:

```markdown
## Project General Instructions

- Always use the latest versions of dependencies
- Always use Maven for dependency management
- Always create test cases (positive and negative)
- Generate the CI pipeline to verify the code
- Minimize the amount of generated code
- Do not use the Lombok library
- Generate Docker Compose for all components
- Update README.md with each new version
```

## Skills — Extending Claude's Knowledge

Skills extend Claude's knowledge about your specific tech stack. Each skill lives in `.claude/skills/` and has a header with metadata:

```markdown
---
name: java-architect
description: Use when building enterprise Java apps with Spring Boot,
  microservices, or reactive programming.
metadata:
  version: "1.0.0"
  domain: language
  triggers: Spring Boot, Java, JPA, Hibernate, WebFlux
  role: architect
  scope: implementation
---

# Java Architect

Enterprise Java specialist focused on Spring Boot and cloud-native development.

## Core Workflow
1. **Architecture analysis** — Review project structure and dependencies
2. **Domain design** — Create models following DDD and Clean Architecture
3. **Implementation** — Build services with Spring Boot best practices
4. **Data layer** — Optimize JPA queries, implement repositories
5. **Security & config** — Apply Spring Security, externalize configuration
6. **Quality assurance** — Run `mvn verify`, ensure 85%+ coverage
```

You can check loaded skills with the `/skills` command in Claude Code.

## Subagents — Specialized AI Workers

Subagents are specialized agents that handle specific concerns:

| Agent | Role |
|-------|------|
| `spring-boot-engineer` | Core application code |
| `java-architect` | Architecture decisions |
| `code-reviewer` | Code quality review |
| `security-engineer` | Security best practices |
| `docker-expert` | Containerization |
| `kubernetes-specialist` | K8s deployment manifests |

> **Note:** Subagents can significantly increase token usage. If you want to keep costs low, disable subagents and use the `spring-boot-engineer` agent as your main agent.

## Practical Example

After setting up the template, here's a sample prompt:

```
Create an application that exposes a REST API and connects to a
PostgreSQL database. The application should have a Person entity
with an id and typical fields. All REST endpoints should be
protected with JWT and OAuth2. Use Skaffold for Kubernetes deployment.
```

Claude Code will:
1. **Enter plan mode** — create a detailed implementation strategy
2. **Generate code in phases** — typically 8-10 phases
3. **Write and run tests** — using Testcontainers for integration tests
4. **Create infrastructure** — Dockerfile, Docker Compose, K8s manifests, CI pipeline
5. **Review for quality** — check for overengineering and best practices

A prompt like this can generate ~3700 lines of well-structured code.

## The Recommended Workflow

Following Anthropic's recommendations, the workflow has four phases:

1. **Explore** — Claude analyzes the codebase and understands the context
2. **Plan** — Creates a detailed implementation plan with token estimates
3. **Implement** — Generates code phase by phase
4. **Commit** — Verifies everything works and commits

## Best Practices

1. **Always use plan mode** — It prevents Claude from rushing into bad implementations
2. **Keep skills focused** — One skill = one capability
3. **Add verification steps** — Tests, linting, and coverage checks
4. **Update lessons learned** — Maintain `tasks/lessons.md` for continuous improvement
5. **Start simple** — Use the template as a starting point and iterate based on your needs

## Conclusion

A well-crafted Claude Code template transforms AI code generation from "hit or miss" to consistently high-quality output. The key is providing clear instructions, domain-specific skills, and quality verification gates.

The template is still an evolving practice — experiment with different skill configurations and agent setups to find what works best for your projects.

---

*Want to try it yourself? Check out [Piotr Minkowski's template](https://github.com/piomin/claude-ai-spring-boot) as a starting point and customize it for your own stack.*

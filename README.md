# Jitendra Rawat's Tech Blog

> 🚀 Software Engineer · 🧠 System Design Enthusiast
>
> Writing about **Software Architecture, AI-powered Development, Kafka, Spring Boot,** and building scalable backend systems.

🌐 **Live site:** [jitendrar292.github.io/mytechblog](https://jitendrar292.github.io/mytechblog/)

---

## 📖 About

This is the source code for my personal tech blog where I share deep dives, practical guides, and lessons learned from building production backend systems. The blog is built with [Hugo](https://gohugo.io/) and the [PaperMod](https://github.com/adityatelange/hugo-PaperMod/) theme, and deployed to GitHub Pages.

## ✍️ Topics I Write About

- **Apache Kafka** — offset management, consumer groups, exactly-once semantics
- **Spring Boot** — patterns, performance, production hardening
- **System Design** — distributed systems, scalability, architecture trade-offs
- **AI-powered Development** — Claude Code, prompt engineering, agentic workflows
- **Backend Engineering** — APIs, databases, observability

## 📝 Recent Posts

| Date | Post |
|------|------|
| Apr 21, 2026 | [Building Pahadi AI — A RAG-Powered Garhwali Chatbot for an Endangered Language](https://jitendrar292.github.io/mytechblog/posts/building-pahadi-ai-rag-garhwali-chatbot/) |
| Apr 16, 2026 | [Claude Code Template for Spring Boot — A Practical Guide](https://jitendrar292.github.io/mytechblog/posts/claude-code-template-for-spring-boot/) |
| Apr 16, 2026 | [Deep Dive into Kafka Offset Commit with Spring Boot](https://jitendrar292.github.io/mytechblog/posts/deep-dive-into-kafka-offset-commit-with-spring-boot/) |

Browse all posts → [/posts](https://jitendrar292.github.io/mytechblog/posts/) · Tags → [/tags](https://jitendrar292.github.io/mytechblog/tags/) · Archives → [/archives](https://jitendrar292.github.io/mytechblog/archives/)

## 🛠 Tech Stack

- **Static Site Generator:** [Hugo](https://gohugo.io/)
- **Theme:** [PaperMod](https://github.com/adityatelange/hugo-PaperMod/)
- **Hosting:** GitHub Pages
- **CI/CD:** GitHub Actions

## 🚀 Local Development

### Prerequisites
- [Hugo Extended](https://gohugo.io/installation/) (v0.120+ recommended)
- Git

### Run locally

```bash
# Clone with submodules (PaperMod theme)
git clone --recurse-submodules https://github.com/jitendrar292/mytechblog.git
cd mytechblog

# Start the dev server with drafts enabled
hugo server -D

# Open http://localhost:1313/mytechblog/
```

### Create a new post

```bash
hugo new posts/my-new-post.md
```

Edit the front matter, set `draft: false`, and commit.

### Build for production

```bash
hugo --minify
# Output goes to ./public
```

## 📂 Project Structure

```
mytechblog/
├── archetypes/        # Front-matter templates for new posts
├── content/
│   ├── posts/         # Blog post markdown files
│   └── about.md       # About page
├── static/            # Images, favicons, static assets
├── themes/PaperMod/   # Theme (git submodule)
├── hugo.toml          # Site config
└── .github/workflows/ # GitHub Actions for auto-deploy
```

## 🚢 Deployment

The blog is automatically built and deployed to GitHub Pages on every push to `main` via GitHub Actions.

## 🤝 Connect

- 🐙 GitHub: [@jitendrar292](https://github.com/jitendrar292)
- 💼 LinkedIn: [jitendrar-singh-rawat](https://www.linkedin.com/in/jitendrar-singh-rawat/)
- ✍️ Blog: [jitendrar292.github.io/mytechblog](https://jitendrar292.github.io/mytechblog/)

## 📄 License

- **Content** (blog posts under `content/`) — © 2026 Jitendra Rawat. All rights reserved.
- **Code** (Hugo config, templates, scripts) — MIT License.

---

⭐ If you find a post helpful, consider starring this repo or sharing it!

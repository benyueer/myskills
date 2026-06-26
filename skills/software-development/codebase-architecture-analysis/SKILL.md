---
name: codebase-architecture-analysis
description: "Analyze an unfamiliar codebase and produce comprehensive architecture documentation: tech stack, module design, data flow, deployment topology."
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [architecture, documentation, codebase, analysis, tech-stack, RAG, system-design]
    related_skills: [codebase-inspection, architecture-diagram, writing-plans]
---

# Codebase Architecture Analysis

Analyze an unfamiliar project and produce a structured architecture document covering tech stack, system architecture, module design, and data flow.

## When to Use

- User says "分析这个项目" / "analyze this project" / "document this codebase"
- User wants a technical overview of an unfamiliar codebase
- User asks for architecture documentation, tech stack summary, or module breakdown
- User wants to understand how a project's components fit together

## Approach (Ordered)

### Phase 1: Reconnaissance (High-Level)

1. **Find README** — search for `README*.md`, read the main one first
2. **Find config files** — `package.json`, `requirements.txt`, `pyproject.toml`, `go.mod`, `Cargo.toml`, `docker-compose*.yaml`, `Dockerfile`, `.env.example`
3. **Find entry points** — `main.py`, `app.py`, `server.py`, `index.ts`, `cmd/main.go`
4. **Map directory structure** — `find . -maxdepth 2 -type d | grep -v node_modules | grep -v .git`

### Phase 2: Deep Dive (Core Modules)

5. **Read core source files** — the files referenced by entry points and config
6. **Read connector/integration code** — database clients, API clients, external service integrations
7. **Read deployment configs** — docker-compose, Kubernetes manifests, CI/CD
8. **Read dependency files** — understand all third-party libraries and their roles

### Phase 3: Synthesis (Documentation)

9. **Write the document** using this structure:

## Document Template

```markdown
# {Project Name} 项目深度分析

## 一、项目概述
(Brief description, core features, who built it)

## 二、技术栈全景
(Table format: component → technology → purpose)
Group by layer: backend, storage, frontend, ML/AI, deployment

## 三、系统架构
(ASCII architecture diagram showing layers and data flow)
(Container/service topology if Docker-based)

## 四、模块设计详解
(One section per major module/directory)
For each: file list, class/function overview, key design decisions

## 五、核心流程与模块对应
(End-to-end flow diagram with each step annotated to source file:line)
(For RAG projects: indexing pipeline + query pipeline)
(For web projects: request lifecycle)
(For data pipelines: ingestion → processing → output)

## 六、项目目录结构
(Tree diagram with annotations)

## 七、关键配置参数
(Extract and explain important config values)

## 八、总结
(Key design highlights, architectural strengths)
```

## Key Principles

- **Read before you write** — always read 10+ source files before synthesizing
- **Trace the data flow** — follow data from input to output through all layers
- **Map to source code** — every claim should reference a specific file and function
- **Use tables** — tech stack, configs, and APIs are best presented as tables
- **ASCII diagrams** — architecture diagrams should be ASCII art (portable, terminal-friendly)
- **Don't guess** — if you can't determine something from the code, say so

## Pitfalls

1. **Don't just read README** — READMEs are marketing; source code is truth. Always verify claims against actual implementation.
2. **Don't skip deployment configs** — docker-compose.yaml reveals the real service topology better than any docs.
3. **Don't ignore third_party/** — vendored dependencies often reveal the project's architecture decisions.
4. **Check both entry points** — there may be separate servers (API, worker, scheduler) with different entry points.
5. **Mind the language** — if the user writes in Chinese, produce the document in Chinese. Match the project's language for code terms.

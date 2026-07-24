# Contributing to datamcp

Thanks for taking the time to report a bug or suggest a feature. This document explains how to do it effectively.

`datamcp` is a managed service — the core codebase is not open source. This repository is where we track bugs, feature requests, and roadmap discussions publicly.

---

## Reporting a bug

Open an issue with the **bug** label. Include:

- What you were trying to do
- What happened instead
- Which AI tool you're using (Cursor, Claude Desktop, VS Code, etc.)
- Your source type and provider, such as PostgreSQL on RDS, MySQL, Supabase, Neon, an OpenAPI specification URL, or Agent Memory
- Any error messages from the `datamcp` dashboard or your AI tool

For connection or authentication issues, also include the MCP transport type (streamable-http or SSE) and whether you're using an API key or OAuth.

---

## Suggesting a feature

Open an issue with the **feature request** label. The more specific the better:

- What problem are you trying to solve?
- How are you working around it today?
- Who else on your team would benefit from this?

We read every feature request. Upvote existing ones with 👍 if something already covers your use case — that helps us prioritize.

---

## Questions and support

For general questions, email support@datamcp.app — GitHub issues are for bugs and feature requests only.

---

## Roadmap

You can see what's planned at [datamcp.app/roadmap](https://datamcp.app/roadmap). Issues tagged **roadmap** are things we're already building.

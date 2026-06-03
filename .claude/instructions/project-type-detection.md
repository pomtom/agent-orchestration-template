# Project Type Detection (pointer)

Detection of the .NET project type is **always the first step** of any skill, agent, or
workflow in this framework. The canonical, authoritative procedure and signal tables
live in:

➡️ [`docs/standards/project-type-detection.md`](../../docs/standards/project-type-detection.md)

Supported types: ASP.NET Core Web API, Azure Functions, Durable Functions, ASP.NET MVC,
Blazor, Worker Service, Console Application (plus Library/Test).

Do not duplicate the rules here — read the canonical doc so behavior stays consistent
between Claude Code and GitHub Copilot.

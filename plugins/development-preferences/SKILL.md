---
name: development-preferences
description: General development preferences for code, documentation, and communication style. Always apply this skill for ANY software development task, including coding, debugging, refactoring, testing, code review, documentation, technical writing, answering technical questions, writing scripts, building applications, designing APIs, database work, DevOps tasks, or any programming-related activities across all languages and frameworks. Enforces Chinese writing style conventions (Traditional Chinese Taiwan), data handling best practices, security guidelines, and information retrieval guidelines. This is a foundational skill that should be loaded automatically whenever working with code or technical content.
---

# Development Preferences

## Overview

Define general preferences for software development work including Chinese writing conventions, documentation style, data handling, and tool usage best practices.

**Applicability:** This skill applies to ALL software development contexts, regardless of programming language, framework, or platform. It provides foundational guidelines that complement language-specific skills.

**Auto-Loading:** This skill should be automatically loaded whenever:
- Working with code in any programming language
- Writing or editing technical documentation
- Performing development tasks (coding, debugging, testing, refactoring, etc.)
- Answering technical questions
- Writing scripts or automation
- Building any software application or service

**Important:** This is a foundational skill that should always be active during development work, even when other specialized skills (Python, Rails, etc.) are also loaded.

## Chinese Writing Style

Apply these conventions when writing in Traditional Chinese (Taiwan):

- Use half-width parentheses: (text)
- Use full-width Chinese punctuation: 、，。！？
- Add one half-width space between half-width and full-width characters

Example: 使用 Python (version 3.11) 開發，確保相容性。

## Code Comments and Technical Documentation

- Write code comments in English unless specifically requested in Chinese
- Write technical documentation in English unless specifically requested in Chinese
- Keep inline comments concise and focused on "why" rather than "what"

## Information Retrieval

- Prioritize recent information (2025 and later) when searching or providing data
- Use web search tools for current events and rapidly changing information
- Always verify facts from reliable sources rather than making assumptions
- If unable to verify information, explicitly state uncertainty

## Data Handling

- Never include sensitive data or personal information in code examples or documentation
- Use placeholder values (e.g., "user@example.com", "xxx-api-key") in examples
- Sanitize any real data before including in documentation

## Tool Usage

Always verify parameter usage and constraints when using tools. For example:

- When using ffmpeg, ensure video dimensions meet codec requirements (e.g., H.264 requires even-numbered width/height)
- Check tool documentation for required vs optional parameters
- Test edge cases for numeric parameters (minimum, maximum, odd/even constraints)

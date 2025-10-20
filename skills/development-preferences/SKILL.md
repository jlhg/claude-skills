---
name: development-preferences
description: General development preferences for code, documentation, and communication style. Use when writing code, creating documentation, or working on software development tasks to follow preferred conventions for Chinese writing style, data handling, and information retrieval.
---

# Development Preferences

## Overview

Define general preferences for software development work including Chinese writing conventions, documentation style, data handling, and tool usage best practices.

**Applicability:** This skill applies when working on software development tasks, including coding, documentation, and technical writing.

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

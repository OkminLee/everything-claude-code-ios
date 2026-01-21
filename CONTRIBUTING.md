# Contributing to iOS Claude Code Configuration

Thank you for your interest in contributing!

## How to Contribute

### Reporting Issues

- Use GitHub Issues for bug reports and feature requests
- Include relevant details (iOS version, Xcode version, etc.)

### Pull Requests

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

### Guidelines

- Follow existing file structure and naming conventions
- Use English for code and documentation
- Test agents with actual iOS projects before submitting
- Update README.md if adding new agents/rules/commands

## Agent Development Guidelines

### Frontmatter

```markdown
---
name: agent-name
description: Clear description of what this agent does
tools: Read, Grep, Glob, [other tools]
model: opus
---
```

### Structure

1. **Your Role** - Define the agent's purpose
2. **Process** - Step-by-step workflow
3. **iOS-Specific Considerations** - Platform-specific guidance
4. **Red Flags** - Common issues to check
5. **Best Practices** - Recommendations

## Questions?

Feel free to open an issue for any questions.

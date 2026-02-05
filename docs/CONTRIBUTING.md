# Contributing to Black Box Architecture

Thank you for considering contributing! This project benefits from community input.

## How to Contribute

### Reporting Issues

Found a bug or have a suggestion?

1. Check [existing issues](https://github.com/gl0bal01/black-box-architecture/issues)
2. Create a new issue with:
   - Clear title
   - Description of problem/suggestion
   - Code examples (if applicable)
   - Expected vs. actual behavior

### Suggesting Improvements

Have an idea for improving the prompts?

1. Open an issue with tag `enhancement`
2. Describe the improvement
3. Explain why it would be valuable
4. Provide examples if possible

### Adding Code Examples

Want to add examples in a new language?

1. Create `examples/[language]/` directory
2. Add before/after refactoring examples
3. Include README explaining the transformation
4. Submit a pull request

### Improving Documentation

Found a typo or want to clarify something?

1. Edit the relevant markdown file
2. Submit a pull request
3. Describe what you changed and why

## Development Setup

```bash
# Clone the repo
git clone https://github.com/gl0bal01/black-box-architecture.git
cd black-box-architecture

# Test the agents locally (recommended)
mkdir -p .claude/agents
cp -r agents/* .claude/agents/
```

## Pull Request Process

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Make your changes
4. Commit with clear messages
5. Push to your fork
6. Open a pull request

## Style Guidelines

### For Prompts
- Keep compact versions under 1,500 words
- Use clear, structured formatting
- Include specific examples
- Maintain consistency with existing style

### For Legacy Prompts
- If you update `docs/legacy/skills/` or `docs/legacy/commands/`, keep them aligned with `agents/AGENTS_CONTRACT.md`

### For Documentation
- Use clear, concise language
- Include code examples
- Add table of contents for long docs
- Test all command examples

## Code of Conduct

- Be respectful and constructive
- Focus on ideas, not people
- Help others learn
- Give credit where due

---

**Thank you for contributing!**

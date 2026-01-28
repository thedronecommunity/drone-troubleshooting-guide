# Contributing to Drone Troubleshooting Guide

First off, thank you for considering contributing! 🎉 This guide is built by the community, for the community.

---

## 📋 Table of Contents

- [Code of Conduct](#code-of-conduct)
- [How Can I Contribute?](#how-can-i-contribute)
- [Reporting Issues](#reporting-issues)
- [Suggesting New Problems](#suggesting-new-problems)
- [Improving Existing Content](#improving-existing-content)
- [Style Guide](#style-guide)
- [Pull Request Process](#pull-request-process)

---

## Code of Conduct

This project follows a simple code of conduct:

- **Be respectful** - everyone is here to help each other
- **Be constructive** - critique ideas, not people
- **Be patient** - not everyone has the same experience level
- **Be inclusive** - welcome newcomers to the hobby

---

## How Can I Contribute?

### 🐛 Found an Error?

If you found incorrect information, outdated commands, or broken solutions:

1. Check if it's already reported in [Issues](../../issues)
2. If not, open a new issue with:
   - Which guide/problem number
   - What's wrong
   - What it should be (if you know)

### 💡 Have a New Problem to Add?

We love expanding the guide! If you've encountered a problem not covered:

1. Check it's not already in the guide
2. Open an issue with the "New Problem" template
3. Or submit a PR directly (see below)

### 📝 Want to Improve Existing Content?

- Better explanations
- Additional solutions
- More CLI commands
- Updated parameters for newer firmware
- Translations

All improvements welcome!

---

## Reporting Issues

### Bug Report Template

```markdown
**Guide:** (e.g., 03-esc-motor-issues.md)
**Problem #:** (e.g., Problem 12)
**Section:** (e.g., Commands/Settings)

**What's Wrong:**
(Describe the error or issue)

**Expected/Correct Information:**
(What it should say, if you know)

**Your Setup:**
- FC: 
- Firmware: 
- Version: 
```

---

## Suggesting New Problems

### New Problem Template

```markdown
**Category:** (e.g., ESC & Motor Issues)
**Problem Title:** (e.g., "ESC Beeps But Motor Doesn't Spin")

**Symptoms:**
- Bullet point symptoms
- What the user sees/experiences

**Common Causes:**
- Why this happens
- Most common first

**Solution:**
Step-by-step fix

**Commands/Settings:**
```bash
# Relevant CLI commands
```

**Pro Tip:**
(Optional wisdom)

**Warning:**
(Optional safety note)

**Sources:**
- Forum links
- Documentation references
```

---

## Improving Existing Content

### What We're Looking For

✅ **Good contributions:**
- Verified solutions that work
- Clear, step-by-step instructions
- Specific commands with correct syntax
- Pro tips from real experience
- Safety warnings that prevent damage

❌ **Please avoid:**
- Unverified "I think this might work" solutions
- Copy-pasted content without attribution
- Overly complex solutions when simple ones exist
- Platform-specific solutions without noting the platform

---

## Style Guide

### Writing Style

1. **Be direct** - get to the solution quickly
2. **Be specific** - include exact values, not just "change the setting"
3. **Be practical** - focus on what works in real builds
4. **Be friendly** - write like you are helping a friend

### Formatting

#### Problem Structure

```markdown
## Problem X: Title Here

### Symptoms
- Bullet points
- What user experiences

### Common Causes
- **Bold the most common cause**
- Other causes follow

### Step-by-Step Solution

1. **First step with bold action**
   - Sub-details if needed
   - Code example if needed

2. **Second step**
   ```bash
   # Inline code for commands
   ```

### Commands/Settings

```bash
# Full code block for commands
set parameter = value
save
```

### 💡 Pro Tip
Single paragraph of wisdom.

### ⚠️ Warning
Safety or damage warning.
```

#### Code Blocks

```markdown
# ArduPilot parameters:
PARAM_NAME = value    # Comment explaining

# Betaflight CLI:
set param_name = value
save
```

#### Emphasis

- **Bold** for important terms, causes, actions
- `code` for parameters, commands, file names
- *Italic* sparingly for emphasis

### File Naming

- Lowercase with hyphens: `01-receiver-radio-issues.md`
- Number prefix for ordering: `01-`, `02-`, etc.
- Descriptive names: `wiring-fc-to-receiver.svg`

---

## Pull Request Process

### Before You Start

1. Fork the repository
2. Create a feature branch: `git checkout -b fix/problem-12-typo`
3. Make your changes
4. Test if applicable (verify commands work)

### Branch Naming

- `fix/` - Fixing errors: `fix/gps-baud-rate`
- `add/` - New content: `add/spektrum-binding`
- `improve/` - Enhancements: `improve/blheli-section`
- `docs/` - Documentation: `docs/readme-update`

### Commit Messages

```
fix: correct DSHOT command syntax in problem 11

- Changed DSHOT330 to DSHOT300 (typo)
- Added note about DSHOT600 for F7 boards
```

Format: `type: short description`

Types: `fix`, `add`, `improve`, `docs`, `style`

### Pull Request Template

```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix (correcting wrong information)
- [ ] New content (adding problems/solutions)
- [ ] Improvement (better explanations)
- [ ] Documentation (README, CONTRIBUTING, etc.)

## Checklist
- [ ] I've tested the commands/solutions (if applicable)
- [ ] I've followed the style guide
- [ ] I've checked for typos
- [ ] I've updated the index if adding new problems

## Related Issues
Closes #XX (if applicable)
```

### Review Process

1. Submit PR
2. Maintainer reviews within 1 week
3. Address any feedback
4. PR merged! 🎉

---

## Recognition

Contributors are recognized in:

- Git history (forever!)
- README acknowledgments (for significant contributions)
- Release notes (for new problems/major fixes)

---

## Questions?

- Open an issue with the "Question" label
- Join The Drone Community discussions
- Reach out to maintainers

---

## License

By contributing, you agree that your contributions will be licensed under the MIT License.

---

**Thank you for helping make this guide better for everyone!** 🚁
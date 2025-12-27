# Contributing to Vitae Light Control

Thank you for your interest in contributing to the Vitae Light Control blueprint! This document provides guidelines for contributing to the project.

## How to Contribute

### Reporting Bugs

If you find a bug, please create an issue on GitHub with:

1. **Clear title**: Describe the issue in one sentence
2. **Environment details**:
   - Home Assistant version
   - Blueprint version
   - Relay switch type/model
3. **Steps to reproduce**:
   - What mode were you in?
   - What mode did you try to switch to?
   - What happened vs. what you expected?
4. **Logs**: Include relevant logs from Home Assistant
5. **Configuration**: Share your blueprint configuration (remove sensitive data)

**Example**:
```
Title: Light doesn't switch from night to evening

Environment:
- Home Assistant: 2024.12.0
- Blueprint: v1.0.0
- Switch: Sonoff Basic R3

Steps:
1. Light is in night mode
2. Click evening button in dashboard
3. Expected: Light switches to evening
4. Actual: Light turns off and stays off

Logs:
[paste relevant logs here]
```

### Suggesting Features

Feature requests are welcome! Please create an issue with:

1. **Use case**: Why do you need this feature?
2. **Proposed solution**: How should it work?
3. **Alternatives considered**: What else have you tried?
4. **Impact**: Who would benefit from this?

### Pull Requests

We welcome pull requests for:
- Bug fixes
- Documentation improvements
- New examples
- Performance optimizations
- Additional dashboard cards

#### Before Submitting

1. **Test thoroughly**: Verify your changes with a real Vitae light
2. **Update documentation**: Update relevant docs if behavior changes
3. **Follow style**: Match existing code style and conventions
4. **Add examples**: Include usage examples for new features

#### PR Guidelines

1. **One change per PR**: Keep PRs focused on a single improvement
2. **Clear description**: Explain what changed and why
3. **Test results**: Share test results with actual hardware
4. **Update changelog**: Add entry to CHANGELOG.md (if exists)

## Development Setup

### Prerequisites

- Home Assistant test instance (don't test on production!)
- Vitae biodynamic light
- Compatible relay switch
- Git for version control

### Testing Checklist

Before submitting changes, verify these scenarios:

- [ ] evening → day (forward cycle)
- [ ] day → evening (backward cycle with reset)
- [ ] night → off (turn off)
- [ ] Manual switch toggle (sync test)
- [ ] Reset skip optimization (switch already off 15s+)
- [ ] All modes work correctly
- [ ] No errors in Home Assistant logs
- [ ] Timing is appropriate

### Code Style

#### YAML
- Use 2 spaces for indaytation (no tabs)
- Add comments for complex logic
- Use descriptive variable names
- Follow Home Assistant blueprint conventions

#### Documentation
- Use clear, concise language
- Include code examples for all features
- Add visual aids (tables, lists) where helpful
- Keep line length reasonable (~80-100 chars)

## Project Structure

```
vitae-light-blueprint/
├── blueprints/automation/
│   └── vitae_light_control.yaml   # Main blueprint file
├── docs/
│   └── USAGE.md                    # Detailed usage guide
├── examples/
│   ├── configuration.yaml          # Configuration examples
│   └── lovelace.yaml              # Dashboard examples
├── README.md                       # Main documentation
├── QUICKSTART.md                   # Quick start guide
└── CONTRIBUTING.md                 # This file
```

## Blueprint Modification Guidelines

### Adding New Features

When adding features to the blueprint:

1. **Maintain backward compatibility**: Don't break existing configs
2. **Add as optional input**: Use default values
3. **Document thoroughly**: Update all relevant docs
4. **Test edge cases**: Ensure reliability

### Modifying Timing Logic

Timing is critical for Vitae lights:

- **Reset delay**: Must be 13+ seconds (manufacturer requirement)
- **Switch delay**: Test thoroughly with actual hardware
- **Don't optimize away safety**: Reliability > speed

### Changing Mode Mapping

The mode position mapping is fundamental:
```yaml
evening: 0   # Default after reset
day: 2     # 2 cycles from reset
night: 3     # 3 cycles from reset
```

Changes here affect all switching logic. Discuss in an issue first.

## Documentation Updates

### When to Update Docs

Update documentation when you:
- Add or modify blueprint inputs
- Change default behavior
- Add new examples
- Fix bugs that affect usage

### Which Docs to Update

- **README.md**: Installation, features, basic usage
- **QUICKSTART.md**: Quick start guide for new users
- **docs/USAGE.md**: Detailed usage, scenarios, troubleshooting
- **examples/configuration.yaml**: Configuration examples
- **examples/lovelace.yaml**: Dashboard card examples

## Community Guidelines

### Be Respectful

- Be patient with new users
- Provide constructive feedback
- Help others learn
- Respect different use cases

### Ask Questions

If you're unsure about:
- Implementation approach
- Breaking changes
- API design decisions

Open an issue to discuss before submitting a PR.

## Recognition

Contributors will be recognized in:
- README.md acknowledgments
- Git commit history
- Release notes

## Getting Help

Need help contributing?

- **Questions**: Open a GitHub discussion
- **Bugs**: Create an issue with details
- **Ideas**: Share in GitHub discussions
- **Urgent**: Tag maintainer in issue

## License

By contributing, you agree that your contributions will be licensed under the MIT License.

## Versioning

We follow [Semantic Versioning](https://semver.org/):
- **MAJOR**: Breaking changes
- **MINOR**: New features (backward compatible)
- **PATCH**: Bug fixes

## Release Process

1. Update version in blueprint metadata
2. Update CHANGELOG.md
3. Create release tag
4. Update documentation if needed
5. Announce in Home Assistant community

## Thank You!

Your contributions make this project better for everyone using Vitae lights with Home Assistant. Thank you for taking the time to contribute!

---

**Questions?** Open an issue or discussion on GitHub.

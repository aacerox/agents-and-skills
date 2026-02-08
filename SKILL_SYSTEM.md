# GitHub Copilot Skill System

## Overview

This repository uses GitHub Copilot's **Agent and Skills** architecture to provide language-agnostic testing guidance. This document explains how skills are discovered, loaded, and used by agents.

## How Skill Discovery Works

### 1. Automatic Discovery

GitHub Copilot automatically discovers skills by:
- Scanning the `.github/skills/` directory in your repository
- Looking for subdirectories containing a `SKILL.md` file
- Reading the frontmatter metadata from each skill file

**No explicit links are required** - GitHub Copilot discovers skills automatically based on the directory structure.

### 2. File Structure Convention

```
.github/
├── agents/
│   └── {agent-name}.agent.md    # Agent definition file
└── skills/
    ├── {skill-name}/             # One directory per skill
    │   ├── SKILL.md              # Required: Skill definition
    │   └── resources/            # Optional: Additional documentation
    │       ├── {name}_basic_reference.md
    │       └── {name}_advanced_reference.md
    └── {another-skill}/
        └── SKILL.md
```

### 3. Skill File Format

Each `SKILL.md` file must have:

**Frontmatter (Required):**
```yaml
---
name: "skill-name"
description: "Brief description of what the skill does"
---
```

**Content:**
- Detailed instructions for using the skill
- Examples and patterns
- Best practices
- References to additional resources

### 4. Agent-Skill Relationship

**Agents** orchestrate **Skills**:

```
Agent (tester.agent.md)
├── Defines when to use the agent
├── Provides high-level strategy
└── References skills by name or category

Skills (junit5, mockito, etc.)
├── Provide language-specific implementation
├── Contain detailed patterns and examples
└── Are loaded on-demand by GitHub Copilot
```

## How Skills Are Referenced

### Method 1: By Name (Implicit)

In the agent definition, skills are referenced by mentioning their name:

```markdown
**Examples:** JUnit 5, Jest, pytest, xUnit
```

When GitHub Copilot sees "JUnit 5" mentioned and there's a `junit5` skill available, it can load that skill.

### Method 2: By Category (Conceptual)

The agent describes skill **categories** rather than specific skills:

```markdown
#### 1. Test Framework Skill
Provides test structure, lifecycle management, and organization.
**Examples:** JUnit 5, Jest, pytest, xUnit, RSpec, Go testing
```

GitHub Copilot matches the category concept with available skills based on context.

### Method 3: Explicit Links (Optional)

For better navigation and clarity, you can add explicit links:

```markdown
**Available Skills:**
- [JUnit 5](../skills/junit5/SKILL.md) - Java test framework
- [Mockito](../skills/mockito/SKILL.md) - Java mocking framework
```

**Note:** Explicit links are for human readability. GitHub Copilot discovers skills automatically regardless of whether links exist.

## Skill Loading Process

When GitHub Copilot processes a request:

1. **Agent Selection**: User invokes an agent (e.g., "tester")
2. **Context Loading**: GitHub Copilot loads the agent definition
3. **Skill Discovery**: Scans `.github/skills/` for available skills
4. **Skill Matching**: Based on:
   - Project language/framework detection
   - Agent's skill category descriptions
   - Context from the user's request
5. **Skill Loading**: Loads relevant skill(s) into context
6. **Response Generation**: Uses both agent and skill knowledge

### Example Flow

```
User Request: "Create a unit test for UserService.java"
    ↓
Agent: tester.agent.md loaded
    ↓
Language Detection: Java detected from file extension
    ↓
Skill Discovery: Scans .github/skills/ → finds junit5, mockito, assertj, instancio
    ↓
Skill Selection: Loads JUnit 5 (test framework), Mockito (mocking), 
                 AssertJ (assertions), Instancio (test data)
    ↓
Response: Generates test using patterns from loaded skills
```

## Naming Conventions

### Skill Directory Names
- Use lowercase
- Use hyphens for multi-word names
- Match the skill name in frontmatter (case-insensitive)

**Examples:**
- `junit5` (name: "junit5")
- `mockito` (name: "mockito")
- `assertj` (name: "assertj")

### Skill File Name
- Must be exactly `SKILL.md` (case-sensitive)
- Located in the skill's directory

### Agent File Names
- Format: `{agent-name}.agent.md`
- Example: `tester.agent.md`

## Best Practices

### For Agent Definitions

1. **Describe categories, not specific skills**: Make the agent language-agnostic
2. **Provide examples**: List multiple skill options per category
3. **Focus on orchestration**: Delegate details to skills
4. **Keep it concise**: Agent should be strategic, not tactical

### For Skill Definitions

1. **Be specific**: Provide concrete patterns and examples
2. **Include imports**: Show exact code to use
3. **Provide context**: Explain when to use the skill
4. **Reference documentation**: Link to official docs

### For Repository Organization

1. **One skill per directory**: Don't mix multiple skills in one folder
2. **Use resources folder**: For additional reference documents
3. **Consistent naming**: Follow the conventions above
4. **Metadata accuracy**: Ensure frontmatter matches directory name

## FAQs

### Q: Do I need to add links to skills in my agent definition?
**A:** No, GitHub Copilot discovers skills automatically. Links are optional and only for human navigation.

### Q: How does GitHub Copilot know which skill to use?
**A:** It analyzes:
- The project's language/framework
- The agent's skill category descriptions
- The context of the user's request
- Available skills in `.github/skills/`

### Q: Can I have skills without an agent?
**A:** Yes, skills can be used independently. However, agents provide orchestration and decision-making logic.

### Q: Can I reference external skills?
**A:** No, skills must be in your repository's `.github/skills/` directory.

### Q: How do I test if my skill is discovered?
**A:** GitHub Copilot will automatically use it when relevant. You can verify by:
- Checking the file structure matches conventions
- Ensuring frontmatter is valid
- Testing with relevant requests

### Q: Can multiple agents use the same skill?
**A:** Yes, skills are shared resources. Any agent can use any skill.

## Troubleshooting

### Skill Not Being Used

**Possible causes:**
1. Incorrect file structure (not in `.github/skills/{name}/SKILL.md`)
2. Invalid frontmatter metadata
3. Skill name mismatch between directory and frontmatter
4. Skill content not relevant to the request

**Solutions:**
1. Verify file is at `.github/skills/{skill-name}/SKILL.md`
2. Check YAML frontmatter syntax
3. Ensure directory name matches `name` in frontmatter
4. Make skill content more specific and actionable

### Agent Not Loading Skills

**Possible causes:**
1. Skills directory doesn't exist
2. No `SKILL.md` files in skill directories
3. Agent definition too vague about skill requirements

**Solutions:**
1. Create `.github/skills/` directory
2. Ensure each skill has a `SKILL.md` file
3. Add clear skill category descriptions in agent

## Advanced Topics

### Skill Hierarchies

You can organize skills hierarchically using subdirectories:

```
.github/skills/
├── testing/
│   ├── junit5/
│   │   └── SKILL.md
│   └── pytest/
│       └── SKILL.md
└── mocking/
    ├── mockito/
    │   └── SKILL.md
    └── sinon/
        └── SKILL.md
```

**Note:** GitHub Copilot scans recursively, so this works transparently.

### Skill Versioning

For versioned skills, use directory names:

```
.github/skills/
├── junit4/
│   └── SKILL.md
└── junit5/
    └── SKILL.md
```

### Multi-Language Skills

A single skill can support multiple languages:

```yaml
---
name: "pact"
description: "Consumer-driven contract testing for microservices"
---

# Pact Contract Testing

Supports: Java, JavaScript, Python, Ruby, Go, .NET
```

## References

- [GitHub Copilot Documentation](https://docs.github.com/en/copilot)
- [Agent and Skills Architecture](https://github.blog/)
- [Example Repository Structure](../.github/)

---

**Last Updated:** 2026-02-08

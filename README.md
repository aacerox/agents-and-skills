# agents-and-skills

This repository demonstrates GitHub Copilot's **Agent and Skills** architecture for language-agnostic testing guidance.

## Architecture

### Agents
Agents orchestrate high-level strategies and decision-making:
- **Location:** `.github/agents/`
- **Current Agents:**
  - `tester.agent.md` - Test design and implementation specialist

### Skills
Skills provide language-specific implementation details:
- **Location:** `.github/skills/`
- **Current Skills (Java Testing):**
  - `junit5` - Test framework for Java
  - `mockito` - Mocking framework for Java
  - `assertj` - Assertion library for Java
  - `instancio` - Test data generator for Java
  - `pact` - Contract testing (multi-language)

## How It Works

1. **Automatic Discovery**: GitHub Copilot automatically scans `.github/skills/` and loads available skills
2. **Agent Orchestration**: Agents provide strategic guidance and decide which skills to use
3. **Skill Execution**: Skills provide concrete patterns and language-specific implementations

**No explicit links required** - GitHub Copilot discovers skills automatically based on directory structure and naming conventions.

## File Structure

```
.github/
├── agents/
│   └── tester.agent.md          # Agent definition
└── skills/
    ├── junit5/
    │   ├── SKILL.md              # Skill definition
    │   └── resources/            # Additional references
    ├── mockito/
    │   └── SKILL.md
    ├── assertj/
    │   └── SKILL.md
    ├── instancio/
    │   └── SKILL.md
    └── pact/
        └── SKILL.md
```

## Documentation

- **[SKILL_SYSTEM.md](SKILL_SYSTEM.md)** - Complete documentation on how skill discovery and loading works
- **[LANGUAGE_AGNOSTIC_TRANSFORMATION.md](LANGUAGE_AGNOSTIC_TRANSFORMATION.md)** - Details on the language-agnostic architecture
- **[REDUCTION_SUMMARY.md](REDUCTION_SUMMARY.md)** - Agent definition optimization details

## Usage

When using GitHub Copilot in this repository:

1. The `tester` agent provides high-level testing strategies
2. GitHub Copilot automatically loads relevant skills based on:
   - Your project's language (detected from file extensions)
   - The context of your request
   - Available skills in `.github/skills/`

3. Skills provide concrete implementation patterns and examples

**Example:** When creating a Java unit test, GitHub Copilot loads `junit5`, `mockito`, `assertj`, and `instancio` skills automatically.

## Learn More

- [How Skill Discovery Works](SKILL_SYSTEM.md#how-skill-discovery-works)
- [Agent-Skill Relationship](SKILL_SYSTEM.md#agent-skill-relationship)
- [Naming Conventions](SKILL_SYSTEM.md#naming-conventions)


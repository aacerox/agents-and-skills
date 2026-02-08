# Skill Discovery and Loading - Visual Guide

## System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    GitHub Copilot Agent System                   │
└─────────────────────────────────────────────────────────────────┘
                                 │
                                 │
                    ┌────────────┴───────────┐
                    │                        │
              ┌─────▼──────┐          ┌─────▼──────┐
              │   Agents   │          │   Skills   │
              │  (.agent)  │          │  (SKILL)   │
              └────────────┘          └────────────┘
                    │                        │
                    │                        │
         ┌──────────┴─────────┐    ┌────────┴─────────┐
         │                    │    │                  │
    ┌────▼────┐         ┌────▼────▼───┐         ┌────▼────┐
    │ tester  │────────>│ Orchestrates │<────────│ junit5  │
    │ .agent  │         │    Skills    │         │ mockito │
    └─────────┘         └──────────────┘         │ assertj │
                                                  │instancio│
                                                  │  pact   │
                                                  └─────────┘
```

## File Structure

```
repository/
│
├── .github/
│   ├── agents/
│   │   └── tester.agent.md ──────────┐
│   │       (orchestrator)             │
│   │                                  │ References
│   └── skills/                        │ (implicit or explicit)
│       ├── junit5/                    │
│       │   └── SKILL.md <─────────────┤
│       ├── mockito/                   │
│       │   └── SKILL.md <─────────────┤
│       ├── assertj/                   │
│       │   └── SKILL.md <─────────────┤
│       ├── instancio/                 │
│       │   └── SKILL.md <─────────────┤
│       └── pact/                      │
│           └── SKILL.md <─────────────┘
│
└── src/
    └── UserService.java (user's code)
```

## Skill Discovery Process

```
┌──────────────────────────────────────────────────────────────────┐
│ Step 1: Repository Initialization                                │
└──────────────────────────────────────────────────────────────────┘
                                │
                    GitHub Copilot starts
                                │
                    ┌───────────▼────────────┐
                    │ Scan .github/ folder   │
                    └───────────┬────────────┘
                                │
                    ┌───────────▼────────────┐
                    │ Find agents/ directory │
                    │ Find skills/ directory │
                    └───────────┬────────────┘
                                │
                    ┌───────────▼────────────┐
                    │ Load all *.agent.md    │
                    │ Load all SKILL.md      │
                    └───────────┬────────────┘
                                │
                    ┌───────────▼────────────┐
                    │ Parse frontmatter      │
                    │ - name                 │
                    │ - description          │
                    └───────────┬────────────┘
                                │
                    ┌───────────▼────────────┐
                    │ Index skills by name   │
                    │ Ready for use!         │
                    └────────────────────────┘

┌──────────────────────────────────────────────────────────────────┐
│ Step 2: User Request Processing                                  │
└──────────────────────────────────────────────────────────────────┘

    User Request: "Create a unit test for UserService.java"
                                │
                    ┌───────────▼────────────┐
                    │ Analyze request        │
                    │ - Intent: Create test  │
                    │ - File: .java          │
                    └───────────┬────────────┘
                                │
                    ┌───────────▼────────────┐
                    │ Detect language        │
                    │ Language: Java         │
                    └───────────┬────────────┘
                                │
                    ┌───────────▼────────────┐
                    │ Select agent           │
                    │ Agent: tester          │
                    └───────────┬────────────┘
                                │
                    ┌───────────▼────────────┐
                    │ Load agent context     │
                    │ (tester.agent.md)      │
                    └───────────┬────────────┘
                                │
                    ┌───────────▼────────────┐
                    │ Match skill categories │
                    │ Needed:                │
                    │ - Test Framework       │
                    │ - Mocking              │
                    │ - Assertions           │
                    │ - Test Data            │
                    └───────────┬────────────┘
                                │
                    ┌───────────▼────────────┐
                    │ Find matching skills   │
                    │ - junit5 (Java test)   │
                    │ - mockito (Java mock)  │
                    │ - assertj (Java assert)│
                    │ - instancio (Java data)│
                    └───────────┬────────────┘
                                │
                    ┌───────────▼────────────┐
                    │ Load skill content     │
                    │ (SKILL.md files)       │
                    └───────────┬────────────┘
                                │
                    ┌───────────▼────────────┐
                    │ Generate response      │
                    │ Using:                 │
                    │ - Agent strategy       │
                    │ - Skill patterns       │
                    └───────────┬────────────┘
                                │
                    ┌───────────▼────────────┐
                    │ Return test code       │
                    │ @Test                  │
                    │ void testMethod() {...}│
                    └────────────────────────┘
```

## Skill Matching Logic

```
┌────────────────────────────────────────────────────────┐
│                   Skill Matching                        │
└────────────────────────────────────────────────────────┘

Input Factors:
    ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
    │  Language    │     │   Context    │     │   Request    │
    │  Detection   │     │   Analysis   │     │   Intent     │
    └──────┬───────┘     └──────┬───────┘     └──────┬───────┘
           │                    │                    │
           └────────────────────┼────────────────────┘
                                │
                    ┌───────────▼────────────┐
                    │  Matching Algorithm    │
                    ├────────────────────────┤
                    │ 1. Filter by language  │
                    │ 2. Match categories    │
                    │ 3. Check availability  │
                    │ 4. Load relevant skills│
                    └───────────┬────────────┘
                                │
                    ┌───────────▼────────────┐
                    │   Selected Skills      │
                    └────────────────────────┘

Examples:

Java Test Request:
    Language: Java
    Category: Test Framework → junit5 ✓
    Category: Mocking → mockito ✓
    Category: Assertions → assertj ✓
    Category: Test Data → instancio ✓

Python Test Request:
    Language: Python
    Category: Test Framework → pytest (if available)
    Category: Mocking → unittest.mock (if available)
    (Would need Python skills in .github/skills/)

JavaScript Test Request:
    Language: JavaScript
    Category: Test Framework → jest (if available)
    Category: Mocking → sinon (if available)
    (Would need JS skills in .github/skills/)
```

## Naming Convention Flow

```
Skill Directory Structure:
    .github/skills/{skill-name}/SKILL.md
                   └─────┬─────┘
                         │
                    Must match
                         │
                         ▼
    Frontmatter:
    ---
    name: "{skill-name}"
    description: "..."
    ---

Example:
    Directory: .github/skills/junit5/
    File:      .github/skills/junit5/SKILL.md
    Frontmatter name: "junit5"
    
    ✓ All three match = Discoverable
    ✗ Any mismatch = Not discoverable
```

## Reference Methods

```
┌────────────────────────────────────────────────────────┐
│          How Agents Reference Skills                   │
└────────────────────────────────────────────────────────┘

Method 1: By Name (Implicit)
    Agent mentions: "JUnit 5"
    GitHub Copilot finds: junit5 skill
    Status: ✓ Works automatically

Method 2: By Category (Conceptual)
    Agent describes: "Test Framework Skill"
    Agent lists examples: "JUnit 5, Jest, pytest"
    GitHub Copilot matches: junit5 (for Java context)
    Status: ✓ Works automatically

Method 3: By Link (Explicit)
    Agent links: [junit5](../skills/junit5/SKILL.md)
    Benefit: Human navigation
    For Copilot: Same as implicit
    Status: ✓ Optional but recommended

All three methods result in the same skill loading!
```

## Complete Flow Example

```
┌─────────────────────────────────────────────────────────────────┐
│ Real-World Example: Creating a Java Unit Test                   │
└─────────────────────────────────────────────────────────────────┘

User Types:
    "Create a unit test for UserService.createUser method"

GitHub Copilot:
    ┌─────────────────────────────────────┐
    │ 1. Context Analysis                 │
    │    - File: UserService.java         │
    │    - Language: Java                 │
    │    - Task: Create unit test         │
    └─────────────────────────────────────┘
                    ↓
    ┌─────────────────────────────────────┐
    │ 2. Agent Selection                  │
    │    - Matched: tester.agent.md       │
    │    - Why: "test" keyword + .java    │
    └─────────────────────────────────────┘
                    ↓
    ┌─────────────────────────────────────┐
    │ 3. Skill Discovery                  │
    │    Scanning .github/skills/...      │
    │    Found:                           │
    │    ✓ junit5/SKILL.md                │
    │    ✓ mockito/SKILL.md               │
    │    ✓ assertj/SKILL.md               │
    │    ✓ instancio/SKILL.md             │
    │    ✓ pact/SKILL.md                  │
    └─────────────────────────────────────┘
                    ↓
    ┌─────────────────────────────────────┐
    │ 4. Skill Selection                  │
    │    For unit test, need:             │
    │    ✓ junit5 (test structure)        │
    │    ✓ mockito (mocking dependencies) │
    │    ✓ assertj (assertions)           │
    │    ✓ instancio (test data)          │
    │    ✗ pact (not needed for unit test)│
    └─────────────────────────────────────┘
                    ↓
    ┌─────────────────────────────────────┐
    │ 5. Content Loading                  │
    │    Loading into context:            │
    │    - tester.agent.md (strategy)     │
    │    - junit5/SKILL.md (patterns)     │
    │    - mockito/SKILL.md (patterns)    │
    │    - assertj/SKILL.md (patterns)    │
    │    - instancio/SKILL.md (patterns)  │
    └─────────────────────────────────────┘
                    ↓
    ┌─────────────────────────────────────┐
    │ 6. Code Generation                  │
    │    Using patterns from skills:      │
    │                                     │
    │    @ExtendWith(MockitoExtension.class)
    │    class UserServiceTest {          │
    │        @Mock                        │
    │        private UserRepository repo; │
    │                                     │
    │        @InjectMocks                 │
    │        private UserService service; │
    │                                     │
    │        @Test                        │
    │        void shouldCreateUser() {    │
    │            User user = Instancio    │
    │                .create(User.class); │
    │            when(repo.save(...))     │
    │                .thenReturn(user);   │
    │            assertThat(result)       │
    │                .isNotNull();        │
    │        }                            │
    │    }                                │
    └─────────────────────────────────────┘
```

## Key Takeaways

1. **Automatic Discovery**: GitHub Copilot scans `.github/skills/` automatically
2. **No Links Required**: Skills are discovered by directory structure and frontmatter
3. **Context-Aware**: Skills loaded based on language, request, and agent guidance
4. **Explicit Links Optional**: Helpful for humans, not required for functionality
5. **Naming Matters**: Directory name must match frontmatter `name` field

---

**For more details, see [SKILL_SYSTEM.md](SKILL_SYSTEM.md)**

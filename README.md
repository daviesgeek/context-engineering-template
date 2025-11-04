# Context Engineering Template (daviesgeek's version)

## Overview

A comprehensive template for getting started with Context Engineering - the discipline of engineering context for AI coding assistants so they have the information necessary to get the job done end to end.

> **Context Engineering is 10x better than prompt engineering and 100x better than vibe coding.**

## daviesgeek's notes

I've made some changes to fit my workflow better. The main changes are:

### Key Improvements

- **Staged PRP workflow** - PRPs move through lifecycle stages (Prompts → Drafts → Ready → In-Progress → Done)
- **New review step** - Added a review/approval gate between PRP generation and execution
- **Automatic stage management** - The `/execute-prp` command automatically moves PRPs between stages
- **Date-prefixed PRPs** - Generated PRPs are prefixed with `YYYY-MM-DD` for better organization
- **Better source control** - Track prompts and PRPs through their lifecycle
- **State tracking** - Easy to see where you left off and resume work
- **Parallel workflows** - Multiple Claude Code instances can work on different PRPs simultaneously

### Staged PRP Workflow

The directory structure takes inspiration from the [Johnny Decimal](https://johnnydecimal.com/) system to organize PRPs through their lifecycle:

1. **`00-Prompts/`** - Initial feature requests (INITIAL.md files)
2. **`01-Drafts/`** - Draft PRPs being refined and reviewed
3. **`10-Ready/`** - Approved PRPs ready to execute
4. **`30-In-Progress/`** - PRPs currently being implemented
5. **`40-Done/`** - Completed PRPs with implemented features
6. **`99-Archive/`** - Archived PRPs no longer needed

Additionally:
- **`98-Templates/`** - Base templates for creating new PRPs and prompts
- **`examples/`** - Example PRPs and INITIAL files for reference

**Note:** The gap between `10-Ready` and `30-In-Progress` (e.g., 20-X) is intentional, allowing you to add custom workflow stages like `20-Approved`, `20-Scheduled`, or other states that fit your process.

See the [Template Structure](#template-structure) section for the complete directory layout, and [The PRP Workflow](#the-prp-workflow) section for detailed usage instructions.

## 🚀 Quick Start

```bash
# 1. Clone this template
git clone https://github.com/coleam00/Context-Engineering-Intro.git
cd Context-Engineering-Intro

# 2. Set up your project rules (optional - template provided)
# Edit CLAUDE.md to add your project-specific guidelines

# 3. Add examples (highly recommended)
# Place relevant code examples in the examples/ folder

# 4. Create your initial feature request
# Create a file in PRPs/00-Prompts/ with your feature requirements
# Example: PRPs/00-Prompts/my-feature.md

# 5. Generate a comprehensive PRP (Product Requirements Prompt)
# In Claude Code, run:
/generate-prp PRPs/00-Prompts/my-feature.md

# 6. Review and approve the generated PRP
# The PRP will be in PRPs/01-Drafts/
# Move it to PRPs/10-Ready/ when satisfied

# 7. Execute the PRP to implement your feature
# In Claude Code, run:
/execute-prp PRPs/10-Ready/my-feature.md
```

## 📚 Table of Contents

- [What is Context Engineering?](#what-is-context-engineering)
- [Template Structure](#template-structure)
- [Step-by-Step Guide](#step-by-step-guide)
- [Writing Effective INITIAL.md Files](#writing-effective-initialmd-files)
- [The PRP Workflow](#the-prp-workflow)
- [Using Examples Effectively](#using-examples-effectively)
- [Best Practices](#best-practices)

## What is Context Engineering?

Context Engineering represents a paradigm shift from traditional prompt engineering:

### Prompt Engineering vs Context Engineering

**Prompt Engineering:**
- Focuses on clever wording and specific phrasing
- Limited to how you phrase a task
- Like giving someone a sticky note

**Context Engineering:**
- A complete system for providing comprehensive context
- Includes documentation, examples, rules, patterns, and validation
- Like writing a full screenplay with all the details

### Why Context Engineering Matters

1. **Reduces AI Failures**: Most agent failures aren't model failures - they're context failures
2. **Ensures Consistency**: AI follows your project patterns and conventions
3. **Enables Complex Features**: AI can handle multi-step implementations with proper context
4. **Self-Correcting**: Validation loops allow AI to fix its own mistakes

## Template Structure

```
context-engineering-intro/
├── .claude/
│   ├── commands/
│   │   ├── generate-prp.md    # Generates comprehensive PRPs
│   │   └── execute-prp.md     # Executes PRPs to implement features
│   └── settings.local.json    # Claude Code permissions
├── PRPs/
│   ├── 00-Prompts/           # Initial feature requests (INITIAL.md files)
│   ├── 01-Drafts/            # Draft PRPs being refined
│   ├── 10-Ready/             # Approved PRPs ready to execute
│   ├── 30-In-Progress/       # PRPs currently being implemented
│   ├── 40-Done/              # Completed PRPs
│   ├── 99-Archive/           # Archived PRPs
│   ├── 98-Templates/         # Base templates for PRPs and prompts
│   └── examples/             # Example PRPs and INITIAL files
├── examples/                  # Your code examples (critical!)
├── CLAUDE.md                 # Global rules for AI assistant
└── README.md                 # This file
```

This template doesn't focus on RAG and tools with context engineering because I have a LOT more in store for that soon. ;)

## Step-by-Step Guide

### 1. Set Up Global Rules (CLAUDE.md)

The `CLAUDE.md` file contains project-wide rules that the AI assistant will follow in every conversation. The template includes:

- **Project awareness**: Reading planning docs, checking tasks
- **Code structure**: File size limits, module organization
- **Testing requirements**: Unit test patterns, coverage expectations
- **Style conventions**: Language preferences, formatting rules
- **Documentation standards**: Docstring formats, commenting practices

**You can use the provided template as-is or customize it for your project.**

### 2. Create Your Initial Feature Request

Create a new file in `PRPs/00-Prompts/` to describe what you want to build (e.g., `PRPs/00-Prompts/my-feature.md`):

```markdown
## FEATURE:
[Describe what you want to build - be specific about functionality and requirements]

## EXAMPLES:
[List any example files in the examples/ folder and explain how they should be used]

## DOCUMENTATION:
[Include links to relevant documentation, APIs, or MCP server resources]

## OTHER CONSIDERATIONS:
[Mention any gotchas, specific requirements, or things AI assistants commonly miss]
```

**See `PRPs/examples/INITIAL_EXAMPLE.md` for a complete example.**

### 3. Generate the PRP

PRPs (Product Requirements Prompts) are comprehensive implementation blueprints that include:

- Complete context and documentation
- Implementation steps with validation
- Error handling patterns
- Test requirements

They are similar to PRDs (Product Requirements Documents) but are crafted more specifically to instruct an AI coding assistant.

Run in Claude Code:
```bash
/generate-prp PRPs/00-Prompts/my-feature.md
```

**Note:** The slash commands are custom commands defined in `.claude/commands/`. You can view their implementation:
- `.claude/commands/generate-prp.md` - See how it researches and creates PRPs
- `.claude/commands/execute-prp.md` - See how it implements features from PRPs

The `$ARGUMENTS` variable in these commands receives whatever you pass after the command name.

This command will:

1. Read your feature request from `00-Prompts/`
2. Research the codebase for patterns
3. Search for relevant documentation
4. Create a comprehensive PRP in `PRPs/01-Drafts/my-feature.md`

### 4. Review and Approve *(New Step)*

**This is a new step in the staged workflow** that adds a review gate between PRP generation and execution.

Once the PRP is generated in `01-Drafts/`:

1. Review the implementation plan
2. Check that all context and requirements are captured
3. Make any necessary edits
4. Move the file to `10-Ready/` when satisfied

```bash
mv PRPs/01-Drafts/my-feature.md PRPs/10-Ready/my-feature.md
```

### 5. Execute the PRP

Execute the approved PRP to implement your feature:

```bash
/execute-prp PRPs/10-Ready/my-feature.md
```

The AI coding assistant will:

1. Read all context from the PRP in `10-Ready/`
2. Move the PRP to `30-In-Progress/` (tracking active work)
3. Create a detailed implementation plan
4. Execute each step with validation
5. Run tests and fix any issues
6. Ensure all success criteria are met
7. Move the completed PRP to `40-Done/`

**Parallel Workflows:** Multiple Claude Code instances can work simultaneously on different PRPs in `30-In-Progress/` without conflicts.

## Writing Effective INITIAL.md Files

### Key Sections Explained

**FEATURE**: Be specific and comprehensive
- ❌ "Build a web scraper"
- ✅ "Build an async web scraper using BeautifulSoup that extracts product data from e-commerce sites, handles rate limiting, and stores results in PostgreSQL"

**EXAMPLES**: Leverage the examples/ folder
- Place relevant code patterns in `examples/`
- Reference specific files and patterns to follow
- Explain what aspects should be mimicked

**DOCUMENTATION**: Include all relevant resources
- API documentation URLs
- Library guides
- MCP server documentation
- Database schemas

**OTHER CONSIDERATIONS**: Capture important details
- Authentication requirements
- Rate limits or quotas
- Common pitfalls
- Performance requirements

## The PRP Workflow

### How /generate-prp Works

The command follows this process:

1. **Research Phase**
   - Analyzes your codebase for patterns
   - Searches for similar implementations
   - Identifies conventions to follow

2. **Documentation Gathering**
   - Fetches relevant API docs
   - Includes library documentation
   - Adds gotchas and quirks

3. **Blueprint Creation**
   - Creates step-by-step implementation plan
   - Includes validation gates
   - Adds test requirements

4. **Quality Check**
   - Scores confidence level (1-10)
   - Ensures all context is included

### How /execute-prp Works

1. **Load Context**: Reads the entire PRP
2. **Plan**: Creates detailed task list using TodoWrite
3. **Execute**: Implements each component
4. **Validate**: Runs tests and linting
5. **Iterate**: Fixes any issues found
6. **Complete**: Ensures all requirements met

See `PRPs/examples/EXAMPLE_multi_agent_prp.md` for a complete example of what gets generated.

## Using Examples Effectively

The `examples/` folder is **critical** for success. AI coding assistants perform much better when they can see patterns to follow.

### What to Include in Examples

1. **Code Structure Patterns**
   - How you organize modules
   - Import conventions
   - Class/function patterns

2. **Testing Patterns**
   - Test file structure
   - Mocking approaches
   - Assertion styles

3. **Integration Patterns**
   - API client implementations
   - Database connections
   - Authentication flows

4. **CLI Patterns**
   - Argument parsing
   - Output formatting
   - Error handling

### Example Structure

```
examples/
├── README.md           # Explains what each example demonstrates
├── cli.py             # CLI implementation pattern
├── agent/             # Agent architecture patterns
│   ├── agent.py      # Agent creation pattern
│   ├── tools.py      # Tool implementation pattern
│   └── providers.py  # Multi-provider pattern
└── tests/            # Testing patterns
    ├── test_agent.py # Unit test patterns
    └── conftest.py   # Pytest configuration
```

## Best Practices

### 1. Be Explicit in INITIAL.md
- Don't assume the AI knows your preferences
- Include specific requirements and constraints
- Reference examples liberally

### 2. Provide Comprehensive Examples
- More examples = better implementations
- Show both what to do AND what not to do
- Include error handling patterns

### 3. Use Validation Gates
- PRPs include test commands that must pass
- AI will iterate until all validations succeed
- This ensures working code on first try

### 4. Leverage Documentation
- Include official API docs
- Add MCP server resources
- Reference specific documentation sections

### 5. Customize CLAUDE.md
- Add your conventions
- Include project-specific rules
- Define coding standards

## Resources

- [Claude Code Documentation](https://docs.anthropic.com/en/docs/claude-code)
- [Context Engineering Best Practices](https://www.philschmid.de/context-engineering)
# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Quick Commands

```bash
# Install dependencies (uses uv package manager)
make install

# Format and lint code
make format          # Format with ruff
make lint            # Check with ruff
make check           # Run both format-check and lint
make fix             # Auto-fix issues

# Test
make test            # Run pytest

# Clean cache
make clean           # Remove __pycache__, .pytest_cache, .ruff_cache
```

## Development Environment

- **Package Manager**: uv (recommended) or pip
- **Python Version**: 3.11+ (required: >=3.11,<3.13)
- **Dependencies**: Managed via pyproject.toml with dev group
- **Code Quality**: ruff (linting + formatting), pre-commit hooks
- **Testing**: pytest (with nbval for notebooks)

All commands use `uv run` prefix to execute within the virtual environment.

## Architecture Overview

This is a **multi-capability cookbook repository** organized by use case and technology integration:

### Core Directories

- **`capabilities/`** - Core Claude capabilities with examples
  - `classification/` - Text classification with evaluation framework
  - `retrieval_augmented_generation/` - RAG implementations with evaluation
  - `summarization/` - Text summarization patterns with evaluations
  - `text_to_sql/` - SQL generation from natural language
  - `contextual-embeddings/` - Embedding techniques and applications

- **`tool_use/`** - Integrating Claude with external tools
  - `memory_tool.py` - Persistent memory implementation for agents
  - `tests/` - Unit tests for memory tool (use `pytest tool_use/tests/`)
  - Various integration notebooks (calculator, customer service, etc.)

- **`claude_agent_sdk/`** - Complete agent implementations
  - `chief_of_staff_agent/` - Multi-role financial agent with subagents (see CLAUDE.md for setup)
  - `research_agent/` - Research coordination agent
  - `observability_agent/` - Monitoring and observability agent
  - Each has architecture diagrams and specific setup requirements

- **`skills/`** - Claude Skills feature for document generation
  - Document generation (Excel, PowerPoint, PDF, Word)
  - File utilities and sample data
  - See `skills/CLAUDE.md` for detailed technical gotchas and setup

- **`patterns/`** - Reusable agent patterns and prompts
  - Agent prompts and research workflows
  - README provides pattern explanations

- **`multimodal/`** - Vision and image processing capabilities
- **`finetuning/`** - Fine-tuning dataset preparation and examples
- **`third_party/`** - Integrations with external platforms (MongoDB, Pinecone, LlamaIndex, etc.)

### Key Files

- `pyproject.toml` - Project metadata, dependencies, ruff configuration
- `.pre-commit-config.yaml` - Pre-commit hooks (ruff + notebook validation)
- `scripts/validate_notebooks.py` - Notebook structure validation
- `Makefile` - Development targets
- `CONTRIBUTING.md` - Contribution guidelines with commit conventions

## Code Quality Standards

### Linting & Formatting

- **Tool**: ruff (100 character line length)
- **Target**: Python 3.11
- **Notebook handling**: Notebooks included in format/lint checks
- **Per-file ignores**: Notebooks have relaxed rules (E402, F811, N803, N806)

Run `make fix` to auto-correct most issues.

### Pre-commit Hooks

Hooks automatically run before commits:
- `ruff check --fix` - Linting with auto-fix
- `ruff format` - Code formatting
- `validate_notebooks.py` - Notebook structure validation

Hooks must pass before committing. Fix issues and retry.

### Testing

- **Framework**: pytest
- **Notebook testing**: nbval (validates notebook execution)
- **Run**: `make test` or `uv run pytest`
- **Memory tool tests**: `uv run pytest tool_use/tests/test_memory_tool.py -v`

## Important Patterns & Conventions

### API Key Management

Always use environment variables:
```python
import os
api_key = os.environ.get("ANTHROPIC_API_KEY")
```

Never commit secrets; use `.env` for local development.

### Model References

- Check [Claude documentation](https://docs.claude.com/en/docs/about-claude/models/overview) for current models
- Use model aliases when available for better maintainability
- PR reviews validate model usage is current

### Notebook Best Practices

1. One concept per notebook
2. Include markdown cells explaining expected outputs
3. Test execution from top to bottom without errors
4. Use minimal token counts for API examples
5. Include error handling

### Agent SDK Specific (chief_of_staff_agent)

- Setup requires company financial context in `CLAUDE.md`
- Financial data directory contains sample datasets
- Subagents handle specialized analysis (financial-analyst, recruiter)
- Use `/slash-command` syntax for complex operations
- Output styles in `.claude/output-styles/` control formatting

### Skills Feature Specific (skills/)

- Requires SDK 0.71.0+ with Skills support
- Uses `client.beta.*` namespace with beta headers
- File generation takes 1-2 minutes (Excel/PowerPoint/PDF)
- File downloads via `client.beta.files.download()` and `client.beta.files.retrieve_metadata()`
- Helper functions in `file_utils.py` handle extraction
- See `skills/CLAUDE.md` for critical gotchas (beta headers, beta namespace, file ID extraction)

## Git Workflow

### Commit Convention

Format: `<type>(<scope>): <subject>`

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`, `ci`

Examples:
```bash
git commit -m "feat(capabilities): add text-to-sql evaluation"
git commit -m "fix(tool_use): correct memory tool path validation"
git commit -m "docs(readme): update installation instructions"
```

### Branch Naming

`<your-name>/<feature-description>` (e.g., `alice/add-rag-example`)

## File Structure Notes

- Notebooks have outputs intentionally included to show expected results
- Data files in `capabilities/*/data/` are sample/test datasets
- Generated files go to `outputs/` directories (not committed)
- `.claude/` directory contains Claude-specific configurations and commands

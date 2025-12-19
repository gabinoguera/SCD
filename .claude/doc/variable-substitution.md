# Variable Substitution System

## Overview

The SCD framework uses a variable substitution system to make agents technology-agnostic. Instead of hardcoding technology names, agents read from `project-config.yaml` and adapt their recommendations dynamically.

## How It Works

### 1. Configuration Definition

In `.claude/config/project-config.yaml`, define your tech stack:

```yaml
tech_stack:
  backend:
    framework: "fastapi"
    database: "postgresql"
    architecture: "hexagonal"
```

### 2. Agent Variable Declaration

In the `agents` section, declare which variables each agent needs:

```yaml
agents:
  backend_developer:
    vars:
      framework: "${tech_stack.backend.framework}"
      database: "${tech_stack.backend.database}"
      architecture: "${tech_stack.backend.architecture}"
```

### 3. Variable Resolution

When an agent is invoked, variables are resolved:

```
${tech_stack.backend.framework}  →  "fastapi"
${tech_stack.backend.database}   →  "postgresql"
```

### 4. Usage in Agent Prompts

Agents use these variables in their system prompts:

**Before (Hardcoded):**
```markdown
You build FastAPI applications with PostgreSQL databases following hexagonal architecture.
```

**After (Dynamic):**
```markdown
You build {{framework}} applications with {{database}} databases following {{architecture}} architecture.
```

At runtime:
```markdown
You build fastapi applications with postgresql databases following hexagonal architecture.
```

## Variable Syntax

### Reference Syntax

Use `${path.to.value}` to reference any value in the config:

```yaml
${project.name}                      # "MyProject"
${tech_stack.backend.framework}      # "fastapi"
${tech_stack.frontend.ui_library}    # "shadcn"
${conventions.naming.files}          # "kebab-case"
```

### Interpolation in Agents

Agents receive variables as a dictionary:

```python
{
  "framework": "fastapi",
  "database": "postgresql",
  "architecture": "hexagonal"
}
```

## Implementation Details

### Project Manager Agent

The PM agent handles variable resolution:

```markdown
1. Read project-config.yaml
2. Parse agent-specific vars
3. Resolve all ${} references
4. Pass resolved vars to specialized agents
```

### Specialized Agents

Agents receive variables in their context:

```markdown
## Tech Stack Configuration

Based on project-config.yaml:
- Framework: fastapi
- Database: postgresql
- Architecture: hexagonal

Adapt all recommendations to use these technologies.
```

## Examples

### Example 1: Backend Developer

**Config:**
```yaml
tech_stack:
  backend:
    framework: "django"
    database: "mongodb"
    orm: "djongo"

agents:
  backend_developer:
    vars:
      framework: "${tech_stack.backend.framework}"
      database: "${tech_stack.backend.database}"
      orm: "${tech_stack.backend.orm}"
```

**Agent receives:**
```json
{
  "framework": "django",
  "database": "mongodb",
  "orm": "djongo"
}
```

**Agent adapts recommendations:**
- Uses Django ORM patterns instead of SQLAlchemy
- Recommends Django REST Framework for APIs
- Suggests django-filter for querying
- Uses Django's migration system

### Example 2: Frontend Developer

**Config:**
```yaml
tech_stack:
  frontend:
    framework: "vue"
    ui_library: "vuetify"
    state_management: "pinia"

agents:
  frontend_developer:
    vars:
      framework: "${tech_stack.frontend.framework}"
      ui_library: "${tech_stack.frontend.ui_library}"
      state_management: "${tech_stack.frontend.state_management}"
```

**Agent receives:**
```json
{
  "framework": "vue",
  "ui_library": "vuetify",
  "state_management": "pinia"
}
```

**Agent adapts recommendations:**
- Uses Vue composition API
- Recommends Vuetify components
- Uses Pinia stores for state
- Suggests Vite for bundling

## Best Practices

### 1. Use Descriptive Variable Names

❌ Bad:
```yaml
vars:
  f: "${tech_stack.backend.framework}"
  d: "${tech_stack.backend.database}"
```

✅ Good:
```yaml
vars:
  framework: "${tech_stack.backend.framework}"
  database: "${tech_stack.backend.database}"
```

### 2. Group Related Variables

```yaml
agents:
  backend_developer:
    vars:
      # Core tech
      framework: "${tech_stack.backend.framework}"
      database: "${tech_stack.backend.database}"

      # Architecture
      architecture: "${tech_stack.backend.architecture}"
      orm: "${tech_stack.backend.orm}"

      # Testing
      testing_framework: "${tech_stack.backend.testing_framework}"
      coverage_target: "${tech_stack.backend.coverage_target}"
```

### 3. Provide Fallbacks

For optional values, provide defaults:

```yaml
vars:
  ui_library: "${tech_stack.frontend.ui_library:-none}"
  # If ui_library not set, use "none"
```

### 4. Document Agent Variables

In each agent's markdown file, document required variables:

```markdown
## Required Variables

This agent requires the following variables from project-config.yaml:

- `framework`: Backend framework (e.g., "fastapi", "django")
- `database`: Database system (e.g., "postgresql", "mongodb")
- `architecture`: Architecture pattern (e.g., "hexagonal", "clean")
```

## Variable Categories

### Core Stack Variables
```yaml
framework: "${tech_stack.{layer}.framework}"
language: "${tech_stack.{layer}.language}"
version: "${tech_stack.{layer}.version}"
```

### Architecture Variables
```yaml
architecture: "${tech_stack.{layer}.architecture}"
pattern: "${conventions.{category}.pattern}"
```

### Tool Variables
```yaml
package_manager: "${tech_stack.{layer}.package_manager}"
testing_framework: "${tech_stack.{layer}.testing_framework}"
linter: "${tech_stack.{layer}.tools.linter}"
```

### Convention Variables
```yaml
naming_style: "${conventions.naming.{type}}"
indent_size: "${conventions.style.indent_size}"
quote_style: "${conventions.style.quote_style}"
```

## Troubleshooting

### Variable Not Found

**Error:** `Variable ${tech_stack.backend.orm} not found`

**Solution:** Check if the path exists in project-config.yaml:
```bash
yq '.tech_stack.backend.orm' .claude/config/project-config.yaml
```

### Circular References

**Error:** `Circular reference detected: ${var} -> ${var}`

**Solution:** Variables cannot reference themselves. Use static values or different variables.

### Invalid Syntax

**Error:** `Invalid variable syntax: $tech_stack.backend.framework`

**Solution:** Use curly braces: `${tech_stack.backend.framework}`

## Future Enhancements

### Conditional Variables
```yaml
vars:
  database: "${tech_stack.backend.database}"
  orm: |
    if ${database} == "postgresql":
      return "sqlalchemy"
    elif ${database} == "mongodb":
      return "motor"
```

### Variable Transformations
```yaml
vars:
  framework_uppercase: "${tech_stack.backend.framework | uppercase}"
  # "fastapi" -> "FASTAPI"
```

### Environment-Specific Variables
```yaml
vars:
  database: "${tech_stack.backend.database.${ENV}}"
  # dev: postgresql, prod: postgresql-ha
```

## Related Documentation

- Project Manager Agent: `.claude/agents/project-manager.md`
- Config Template: `.claude/templates/project-config.yaml`
- Agent Adaptation Plan: `agent-adaptation-plan.md`

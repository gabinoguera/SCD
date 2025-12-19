# SCD Framework (Spec Claude Development)

**Version**: 1.0.0
**Status**: Production Ready
**License**: MIT

A portable, extensible, and technology-agnostic framework for Spec-Driven Development powered by OpenSpec and specialized AI agents.

---

## 🎯 What is SCD?

**Spec Claude Development (SCD)** is a meta-framework that orchestrates an "army" of specialized AI agents to transform feature ideas into validated implementations following a rigorous spec-driven workflow.

### Key Principles

1. **Specification First**: Every feature starts with a formal OpenSpec specification
2. **Technology Agnostic**: Adapts to any tech stack via configuration
3. **Agent Orchestration**: Specialized agents handle planning, testing, and validation
4. **Traceability**: Every implementation decision traces back to spec requirements
5. **Quality Assurance**: Built-in validation at every stage

### Architecture

```
User Idea
    ↓
Project Manager Agent (orchestrator)
    ↓
OpenSpec Change Creation
    ├─ proposal.md (why, what, impact)
    ├─ design.md (technical decisions)
    ├─ tasks.md (implementation checklist)
    └─ specs/*.md (requirements + scenarios)
    ↓
Project Config Detection (project-config.yaml)
    ↓
Specialized Agents (parallel execution)
    ├─ backend-developer → backend-plan.md
    ├─ frontend-developer → frontend-plan.md
    ├─ backend-test-engineer → backend-tests.md
    ├─ frontend-test-engineer → frontend-tests.md
    ├─ qa-criteria-validator → validation-criteria.md
    └─ ui-ux-analyzer → ui-analysis.md
    ↓
Implementation Following Plans
    ↓
Validation & Testing
    ↓
OpenSpec Archive (feature complete)
```

---

## 🚀 Quick Start

### Prerequisites

- **Node.js** 18+ (for OpenSpec CLI)
- **Claude Code** (CLI or IDE extension)
- **Git** (for version control)

### Installation

#### 1. Install OpenSpec

```bash
npm install -g @fission-ai/openspec
```

Verify installation:
```bash
openspec --version
```

#### 2. Clone SCD Framework

For **new projects**:
```bash
git clone <scd-framework-repo> my-project
cd my-project
rm -rf .git
git init
```

For **existing projects**:
```bash
cd my-existing-project
# Copy SCD structure
cp -r <scd-framework-repo>/.claude .
cp -r <scd-framework-repo>/openspec .
cp <scd-framework-repo>/CLAUDE.md .
```

#### 3. Initialize Project Configuration

```bash
# Copy template
cp .claude/templates/project-config.yaml .claude/config/project-config.yaml

# Edit with your tech stack
vim .claude/config/project-config.yaml
```

Example configuration:
```yaml
project:
  name: "my-awesome-app"
  description: "E-commerce platform"
  version: "1.0.0"

tech_stack:
  backend:
    language: "python"
    framework: "fastapi"
    database: "postgresql"
    orm: "sqlalchemy"
    architecture: "hexagonal"
    testing_framework: "pytest"
    coverage_target: 80

  frontend:
    language: "typescript"
    framework: "react"
    version: "19"
    ui_library: "shadcn"
    styling: "tailwind"
    state_management: "react-query"
    architecture: "feature-based"
    testing_framework: "vitest"
    coverage_target: 80

  design_system:
    primary_color: "#3b82f6"
    spacing_unit: "4px"
    border_radius: "8px"

coding_conventions:
  naming:
    files: "kebab-case"
    classes: "PascalCase"
    functions: "camelCase"
    constants: "UPPER_SNAKE_CASE"
```

#### 4. Initialize OpenSpec

```bash
openspec init
```

This creates:
- `openspec/project.md` - Project overview
- `openspec/specs/` - Capability specifications
- `openspec/changes/` - Change proposals

---

## 📋 Complete Workflow

### Phase 1: Feature Request → Specification

**User Action**: Request a new feature

```
User: "I need a user authentication system with JWT tokens"
```

**SCD Action**: Project Manager agent creates OpenSpec change

1. Agent analyzes the request
2. Creates `openspec/changes/{change-id}/`
3. Generates:
   - `proposal.md` - Why this change, what's impacted
   - `tasks.md` - Implementation checklist
   - `specs/{capability}/spec.md` - Formal requirements

**Example spec.md**:
```markdown
# User Authentication

## Requirements

### Requirement: JWT Authentication
The system SHALL authenticate users via email and password and return a JWT token.

#### Scenario: Successful login
- **WHEN** valid credentials are provided
- **THEN** return JWT token with user claims and 200 OK

#### Scenario: Invalid credentials
- **WHEN** invalid credentials are provided
- **THEN** return 401 Unauthorized
```

### Phase 2: Specification → Implementation Plans

**SCD Action**: Specialized agents create detailed plans

**Agents Called** (in parallel):
- `backend-developer` → Creates `backend-plan.md`
- `frontend-developer` → Creates `frontend-plan.md`
- `backend-test-engineer` → Creates `backend-tests.md`
- `frontend-test-engineer` → Creates `frontend-tests.md`

**Output Structure**:
```
.claude/
├── sessions/
│   └── context_session_user-auth.md  # Shared context
└── doc/
    └── user-auth/
        ├── backend-plan.md            # Backend implementation plan
        ├── frontend-plan.md           # Frontend implementation plan
        ├── backend-tests.md           # Backend test plan
        └── frontend-tests.md          # Frontend test plan
```

**Example backend-plan.md**:
```markdown
# Backend Implementation Plan: User Authentication

## Context Summary
Implementing JWT authentication system as specified in openspec/changes/AUTH-001.

## Tech Stack
- Language: python
- Framework: fastapi
- Database: postgresql
- Architecture: hexagonal

## Changes Required

### Files to Create
- `src/domain/entities/user.py`
  - Purpose: User domain entity with email/password validation
  - Key Content: User dataclass with bcrypt password hashing

- `src/application/use_cases/authenticate_user.py`
  - Purpose: Login use case
  - Key Content: Validates credentials, generates JWT

### Files to Modify
- `src/infrastructure/web/routers/auth.py`
  - Changes: Add POST /login endpoint
  - Reason: Expose authentication to clients

## Spec Alignment

### Requirement: JWT Authentication
Source: `openspec/changes/AUTH-001/specs/auth/spec.md:15`

Implementation:
- Files: user.py, authenticate_user.py, auth.py
- Components: User entity, AuthenticateUser use case, /login endpoint

Scenarios Covered:
- ✅ Successful login → Returns JWT token
- ✅ Invalid credentials → Returns 401
```

### Phase 3: Implementation Plans → Code

**User Action**: Review plans, then implement

1. Read all generated plans
2. Implement following the detailed specifications
3. Each plan maps directly to OpenSpec requirements

**Implementation Flow**:
```bash
# Backend implementation
# Follow backend-plan.md step by step

# Frontend implementation
# Follow frontend-plan.md step by step

# Continuous validation
# Check that code matches plan specs
```

### Phase 4: Code → Testing

**SCD Action**: Implement tests from test plans

1. Read `backend-tests.md`
2. Implement test cases
3. Read `frontend-tests.md`
4. Implement test cases
5. Verify coverage targets met

**Example Test Implementation**:
```python
# From backend-tests.md

def test_authenticate_user_with_valid_credentials_returns_jwt():
    """
    Maps to: Scenario "Successful login"
    Source: openspec/changes/AUTH-001/specs/auth/spec.md:18
    """
    # Arrange
    user = create_test_user(email="user@example.com", password="secret123")

    # Act
    result = authenticate_user(email="user@example.com", password="secret123")

    # Assert
    assert result.token is not None
    assert jwt.decode(result.token)['user_id'] == user.id
```

### Phase 5: Implementation → Validation

**SCD Action**: QA and UI validation

**Agents Called**:
- `qa-criteria-validator` → Creates validation criteria and runs tests
- `ui-ux-analyzer` → Analyzes UI against OpenSpec and design system

**Output**:
```
.claude/doc/user-auth/
├── validation-criteria.md    # Acceptance criteria (Given-When-Then)
├── validation-report.md      # Test results and evidence
└── ui-analysis.md            # UI/UX feedback with screenshots
```

**Validation Report Example**:
```markdown
# Validation Report: User Authentication

## Summary
- Date: 2025-12-19
- Pass Rate: 95%

## Results

### ✅ Passed Criteria
1. Login with valid credentials returns JWT
   - Evidence: screenshots/login-success.png
   - Performance: 145ms response time

### ❌ Failed Criteria
1. Error message accessibility
   - Reason: Missing ARIA labels on error alert
   - Recommendation: Add role="alert" and aria-live="polite"

## Acceptance Decision
- ⚠️ CONDITIONAL: Minor accessibility fix required
```

### Phase 6: Validation → Archive

**User Action**: Fix issues, then archive

```bash
# Fix validation issues
# Re-run validation

# Archive the change
openspec archive AUTH-001
```

This:
- Moves specs from `openspec/changes/` to `openspec/specs/`
- Creates archive record in `openspec/archives/`
- Marks feature as complete

---

## 🤖 Available Agents

### 1. Project Manager (Orchestrator)
**When to use**: Every new feature request
**Purpose**: Creates OpenSpec changes, coordinates other agents
**Output**: OpenSpec change structure, session context

**Usage**:
```
User: "Create a new shopping cart feature"
→ PM agent creates openspec/changes/CART-001/
→ PM agent calls specialized agents
→ PM agent consolidates outputs
```

### 2. Backend Developer
**When to use**: Planning backend implementation
**Purpose**: Creates detailed backend architecture plan
**Output**: `.claude/doc/{feature}/backend-plan.md`

**Adapts to**:
- Python (FastAPI, Django, Flask)
- TypeScript/JavaScript (Express, NestJS)
- Go (Gin, Echo)
- Java (Spring)

### 3. Frontend Developer
**When to use**: Planning frontend implementation
**Purpose**: Creates detailed UI architecture plan
**Output**: `.claude/doc/{feature}/frontend-plan.md`

**Adapts to**:
- React (hooks, Context, React Query)
- Vue (Composition API, Pinia)
- Svelte (stores, SvelteKit)
- Angular (services, RxJS)

### 4. Backend Test Engineer
**When to use**: After backend implementation plan
**Purpose**: Creates comprehensive backend test plan
**Output**: `.claude/doc/{feature}/backend-tests.md`

**Adapts to**:
- Python: pytest, unittest
- JavaScript: Jest, Vitest
- Go: testing package
- Java: JUnit

### 5. Frontend Test Engineer
**When to use**: After frontend implementation plan
**Purpose**: Creates UI test plan with accessibility focus
**Output**: `.claude/doc/{feature}/frontend-tests.md`

**Adapts to**:
- React: Testing Library, Vitest/Jest
- Vue: Vue Test Utils
- Svelte: Svelte Testing Library
- Angular: TestBed, Jasmine

### 6. QA Criteria Validator
**When to use**: After implementation for acceptance testing
**Purpose**: Defines acceptance criteria, validates implementation
**Output**: `validation-criteria.md`, `validation-report.md`

**Adapts to**:
- Playwright MCP
- Cypress
- Selenium
- Puppeteer

### 7. UI/UX Analyzer
**When to use**: After UI implementation for design review
**Purpose**: Analyzes UI against design system and OpenSpec
**Output**: `.claude/doc/{feature}/ui-analysis.md`

**Adapts to**:
- Any framework (React, Vue, Svelte, Angular)
- Any UI library (Material UI, Chakra, shadcn, Ant Design)
- Any styling (Tailwind, styled-components, CSS modules)

---

## 📁 Directory Structure

```
my-project/
├── .claude/                          # SCD Framework
│   ├── agents/                       # Agent definitions
│   │   ├── project-manager.md
│   │   ├── backend-developer.md
│   │   ├── frontend-developer.md
│   │   ├── backend-test-engineer.md
│   │   ├── frontend-test-engineer.md
│   │   ├── qa-criteria-validator.md
│   │   └── ui-ux-analyzer.md
│   ├── config/
│   │   └── project-config.yaml       # Tech stack configuration
│   ├── doc/                          # Agent outputs
│   │   └── {feature_name}/
│   │       ├── backend-plan.md
│   │       ├── frontend-plan.md
│   │       ├── backend-tests.md
│   │       ├── frontend-tests.md
│   │       ├── validation-criteria.md
│   │       ├── validation-report.md
│   │       └── ui-analysis.md
│   ├── sessions/                     # Shared context
│   │   └── context_session_{feature}.md
│   └── templates/
│       └── project-config.yaml       # Config template
├── openspec/                         # OpenSpec structure
│   ├── project.md
│   ├── specs/                        # Archived specs
│   ├── changes/                      # Active changes
│   │   └── {change-id}/
│   │       ├── proposal.md
│   │       ├── design.md (optional)
│   │       ├── tasks.md
│   │       └── specs/
│   │           └── {capability}/
│   │               └── spec.md
│   └── archives/                     # Completed changes
├── src/                              # Your application code
├── tests/                            # Your tests
├── CLAUDE.md                         # Project instructions for Claude
├── README.md                         # This file
└── PROGRESS.md                       # SCD adaptation progress
```

---

## 🔧 Configuration

### project-config.yaml

The `project-config.yaml` file is the single source of truth for your tech stack. All agents read this file to adapt their recommendations.

**Location**: `.claude/config/project-config.yaml`

**Key Sections**:

```yaml
# Project metadata
project:
  name: "my-app"
  description: "Application description"
  version: "1.0.0"

# Backend stack
tech_stack:
  backend:
    language: "python"              # python | typescript | go | java
    framework: "fastapi"            # fastapi | django | express | gin | spring
    database: "postgresql"          # postgresql | mysql | mongodb
    orm: "sqlalchemy"               # sqlalchemy | typeorm | gorm | jpa
    architecture: "hexagonal"       # hexagonal | layered | clean | mvc
    testing_framework: "pytest"     # pytest | jest | go-test | junit
    coverage_target: 80             # percentage

  # Frontend stack
  frontend:
    language: "typescript"
    framework: "react"              # react | vue | svelte | angular
    version: "19"
    ui_library: "shadcn"            # shadcn | material-ui | chakra | ant-design
    styling: "tailwind"             # tailwind | styled-components | css-modules
    state_management: "react-query" # react-query | redux | zustand | pinia
    architecture: "feature-based"   # feature-based | atomic | mvc
    testing_framework: "vitest"     # vitest | jest | cypress
    coverage_target: 80

  # QA/E2E stack
  qa:
    e2e_framework: "playwright"     # playwright | cypress | selenium
    test_browser: "chromium"
    coverage_target: 90

# Design system
design_system:
  primary_color: "#3b82f6"
  spacing_unit: "4px"
  border_radius: "8px"

# Coding conventions
coding_conventions:
  naming:
    files: "kebab-case"             # kebab-case | snake_case | PascalCase
    classes: "PascalCase"
    functions: "camelCase"          # camelCase | snake_case
    constants: "UPPER_SNAKE_CASE"

  documentation:
    style: "docstring"              # docstring | jsdoc | javadoc
    required_for: ["classes", "public_methods"]
```

**Variable Substitution**:

Agents use variables like `${tech_stack.backend.framework}` to adapt their recommendations dynamically.

Example:
```markdown
You are an expert ${tech_stack.backend.framework} developer...
```

With `framework: "fastapi"` → "You are an expert fastapi developer..."

---

## 💡 Best Practices

### 1. Always Start with Specs

❌ **Don't**:
```
User: "Start building the login form"
```

✅ **Do**:
```
User: "Create an OpenSpec change for user authentication"
→ Review the generated specs
→ Approve and proceed with implementation plans
```

### 2. One Feature = One OpenSpec Change

Each OpenSpec change should represent a cohesive feature or capability.

❌ **Don't**: Mix unrelated changes
```
openspec/changes/MULTI-001/
  - Add login
  - Add product catalog
  - Fix bug in checkout
```

✅ **Do**: Separate changes
```
openspec/changes/AUTH-001/    # Authentication
openspec/changes/PRODUCT-002/ # Product catalog
openspec/changes/BUG-003/     # Checkout fix
```

### 3. Update project-config.yaml First

Before starting a new project or major refactor:

1. Copy template: `cp .claude/templates/project-config.yaml .claude/config/`
2. Edit with your tech stack
3. Commit to version control
4. All agents will now adapt to your stack

### 4. Review Plans Before Implementation

Agents create detailed plans - always review them:

1. Read `backend-plan.md` thoroughly
2. Read `frontend-plan.md` thoroughly
3. Ask questions if anything is unclear
4. Approve before implementing

### 5. Map Everything to Specs

Every file, function, test should trace back to OpenSpec:

```python
# ✅ Good - References spec
def authenticate_user(email: str, password: str) -> Token:
    """
    Authenticates user via email/password.

    Maps to: openspec/changes/AUTH-001/specs/auth/spec.md:15
    Requirement: "JWT Authentication"
    Scenario: "Successful login"
    """
    pass
```

### 6. Archive When Complete

Don't leave changes in `openspec/changes/` forever:

```bash
# After validation passes
openspec archive AUTH-001

# This moves specs to openspec/specs/
# And creates archive record
```

---

## 🎓 Example: Complete Feature Walkthrough

### Scenario: Add Shopping Cart

#### Step 1: Create Feature Request

```
User: "I need a shopping cart where users can add products, view items, and proceed to checkout"
```

#### Step 2: PM Agent Creates OpenSpec Change

```
PM Agent creates:
openspec/changes/CART-001/
├── proposal.md
├── tasks.md
└── specs/
    └── shopping-cart/
        └── spec.md
```

**spec.md**:
```markdown
# Shopping Cart

## Requirements

### Requirement: Add to Cart
The system SHALL allow users to add products to their shopping cart.

#### Scenario: Add product successfully
- WHEN user clicks "Add to Cart" on a product
- THEN product is added to cart
- AND cart count increments
- AND success notification displays

### Requirement: View Cart
The system SHALL display cart items with quantities and prices.

#### Scenario: View populated cart
- WHEN user navigates to cart page
- THEN all cart items are displayed
- AND total price is calculated
```

#### Step 3: Agents Create Plans (Parallel)

**backend-developer** creates `backend-plan.md`:
```markdown
# Backend Implementation Plan: Shopping Cart

## Files to Create
- `src/domain/entities/cart.py`
- `src/domain/entities/cart_item.py`
- `src/application/use_cases/add_to_cart.py`
- `src/application/ports/cart_repository.py`
- `src/infrastructure/adapters/repositories/postgresql_cart_repository.py`
- `src/infrastructure/web/routers/cart.py`
- `src/infrastructure/web/dtos/cart_dto.py`

## Spec Alignment
### Requirement: Add to Cart
- Files: cart.py, add_to_cart.py, cart.py (router)
- Endpoint: POST /cart/items
- Scenario "Add product successfully": Returns 201 Created
```

**frontend-developer** creates `frontend-plan.md`:
```markdown
# Frontend Implementation Plan: Shopping Cart

## Files to Create
- `src/features/cart/components/CartButton.tsx`
- `src/features/cart/components/CartPage.tsx`
- `src/features/cart/data/schemas/cart.schema.ts`
- `src/features/cart/data/services/cart.service.ts`
- `src/features/cart/hooks/useCartContext.tsx`
- `src/features/cart/hooks/mutations/useAddToCart.ts`

## Spec Alignment
### Requirement: Add to Cart
- Components: CartButton (triggers add)
- Scenario "Add product successfully":
  - Button click calls useAddToCart mutation
  - Success shows notification
  - Cart count updates
```

**backend-test-engineer** creates `backend-tests.md`:
```markdown
# Backend Test Plan: Shopping Cart

## Test Cases

### Test Suite: Add to Cart Use Case

**Test Case 1: test_add_to_cart_with_valid_product_creates_cart_item**
```python
def test_add_to_cart_with_valid_product_creates_cart_item():
    # Arrange
    user = create_test_user()
    product = create_test_product()

    # Act
    result = add_to_cart(user_id=user.id, product_id=product.id, quantity=1)

    # Assert
    assert result.success
    assert len(user.cart.items) == 1
```
- Maps to Scenario: "Add product successfully"
```

**frontend-test-engineer** creates `frontend-tests.md`:
```markdown
# Frontend Test Plan: Shopping Cart

## Test Cases

### Test Suite: CartButton Component

**Test Case 1: test_cart_button_adds_product_on_click**
```typescript
test('adds product to cart when clicked', async () => {
  // Arrange
  const user = userEvent.setup()
  render(<CartButton productId="123" />)

  // Act
  await user.click(screen.getByRole('button', { name: /add to cart/i }))

  // Assert
  await waitFor(() => {
    expect(screen.getByText(/added to cart/i)).toBeInTheDocument()
  })
})
```
- Maps to Scenario: "Add product successfully"
```

#### Step 4: Implementation

Developer reads all plans and implements:

**Backend**:
```python
# src/domain/entities/cart.py (from backend-plan.md)

@dataclass
class Cart:
    """
    Shopping cart domain entity.

    Maps to: openspec/changes/CART-001/specs/shopping-cart/spec.md
    """
    id: str
    user_id: str
    items: List[CartItem]

    def add_item(self, product_id: str, quantity: int) -> None:
        """
        Adds product to cart.

        Requirement: "Add to Cart"
        Scenario: "Add product successfully"
        """
        # Implementation
        pass
```

**Frontend**:
```typescript
// src/features/cart/components/CartButton.tsx (from frontend-plan.md)

/**
 * Cart button component
 *
 * Maps to: openspec/changes/CART-001/specs/shopping-cart/spec.md
 * Requirement: "Add to Cart"
 */
export function CartButton({ productId }: CartButtonProps) {
  const { mutate: addToCart, isLoading } = useAddToCart()

  const handleClick = () => {
    // Scenario: "Add product successfully"
    addToCart({ productId, quantity: 1 })
  }

  return (
    <Button onClick={handleClick} disabled={isLoading}>
      Add to Cart
    </Button>
  )
}
```

#### Step 5: Testing

Implement tests from test plans:

**Backend tests**:
```python
# tests/test_add_to_cart.py (from backend-tests.md)

def test_add_to_cart_with_valid_product_creates_cart_item():
    """Maps to: Scenario 'Add product successfully'"""
    # Implementation from test plan
    pass
```

**Frontend tests**:
```typescript
// tests/CartButton.test.tsx (from frontend-tests.md)

test('adds product to cart when clicked', async () => {
  // Implementation from test plan
})
```

#### Step 6: Validation

**QA Validator** creates validation criteria:
```markdown
# Validation Criteria: Shopping Cart

## Acceptance Criteria

### Criterion 1: Add Product to Cart
```gherkin
Given a user is viewing a product
When the user clicks "Add to Cart"
Then the product is added to the cart
And the cart count increments by 1
And a success notification is displayed
```
Maps to: Requirement "Add to Cart", Scenario "Add product successfully"
```

**UI/UX Analyzer** reviews UI:
```markdown
# UI/UX Analysis: Shopping Cart

## Critical Issues
1. **Add to Cart button not prominent enough**
   - Location: Product card
   - Recommendation: Increase size, use primary color
   - Code Example:
   ```tsx
   <Button size="lg" className="bg-blue-600 hover:bg-blue-700">
     Add to Cart
   </Button>
   ```
```

#### Step 7: Fix & Archive

1. Fix validation issues
2. Re-validate
3. Archive:

```bash
openspec archive CART-001
```

✅ Feature complete!

---

## 🔍 Troubleshooting

### Issue: Agent doesn't adapt to my tech stack

**Cause**: project-config.yaml not found or invalid

**Solution**:
```bash
# Verify file exists
ls .claude/config/project-config.yaml

# Verify YAML syntax
cat .claude/config/project-config.yaml | python -c "import yaml, sys; yaml.safe_load(sys.stdin)"

# Ensure agents can read it
# Check CLAUDE.md includes reference to project-config.yaml
```

### Issue: OpenSpec command not found

**Cause**: OpenSpec CLI not installed

**Solution**:
```bash
npm install -g @fission-ai/openspec
openspec --version
```

### Issue: Agents create plans but don't reference OpenSpec

**Cause**: No active OpenSpec change

**Solution**:
```bash
# Create OpenSpec change first
openspec propose "Feature name"

# Or ask PM agent to create it
# "Create an OpenSpec change for [feature]"
```

### Issue: Can't find agent documentation

**Cause**: Agents are in `.claude/agents/`

**Solution**:
```bash
ls .claude/agents/

# Read specific agent
cat .claude/agents/backend-developer.md
```

### Issue: Multiple agents creating conflicting plans

**Cause**: Shared context not being updated

**Solution**:
```bash
# Verify session context exists
ls .claude/sessions/context_session_{feature_name}.md

# Agents should update this file after each phase
```

---

## 📚 Reference

### OpenSpec Commands

```bash
# Initialize OpenSpec in project
openspec init

# Create new change proposal
openspec propose "Feature name"

# Validate change structure
openspec validate {change-id}

# Validate with strict mode
openspec validate {change-id} --strict

# Apply change (implement)
openspec apply {change-id}

# Archive completed change
openspec archive {change-id}

# List all changes
openspec list

# Show change details
openspec show {change-id}
```

### SCD Agent Invocation

From Claude Code CLI or IDE:

```
# Project Manager (orchestrator)
"Create a new feature for [description]"

# Backend Developer
"Create backend implementation plan for [feature]"

# Frontend Developer
"Create frontend implementation plan for [feature]"

# Backend Test Engineer
"Create backend test plan for [feature]"

# Frontend Test Engineer
"Create frontend test plan for [feature]"

# QA Validator
"Create acceptance criteria for [feature]"
"Validate [feature] implementation"

# UI/UX Analyzer
"Analyze the UI for [feature/page]"
```

### File Locations

| File | Purpose | Created By |
|------|---------|------------|
| `project-config.yaml` | Tech stack config | User |
| `context_session_{feature}.md` | Shared context | All agents |
| `backend-plan.md` | Backend architecture | backend-developer |
| `frontend-plan.md` | Frontend architecture | frontend-developer |
| `backend-tests.md` | Backend test plan | backend-test-engineer |
| `frontend-tests.md` | Frontend test plan | frontend-test-engineer |
| `validation-criteria.md` | Acceptance criteria | qa-criteria-validator |
| `validation-report.md` | Test results | qa-criteria-validator |
| `ui-analysis.md` | UI/UX feedback | ui-ux-analyzer |

---

## 🤝 Contributing

SCD Framework is designed to be extensible. To add new agents:

1. Create agent definition in `.claude/agents/{agent-name}.md`
2. Follow the established pattern:
   - Read from `project-config.yaml` for tech stack
   - Integrate with OpenSpec workflow
   - Create standardized output in `.claude/doc/{feature}/`
   - Update shared context in `.claude/sessions/`
3. Add agent to `CLAUDE.md` SUBAGENTS MANAGEMENT section
4. Document in this README

---

## 📄 License

MIT License - See LICENSE file for details

---

## 🙏 Acknowledgments

- **OpenSpec** by Fission AI - Specification management
- **Claude Code** by Anthropic - AI-powered development
- **Community** - For feedback and contributions

---

## 📞 Support

- **Issues**: GitHub Issues
- **Documentation**: This README + CLAUDE.md
- **Examples**: See `examples/` directory

---

**Built with ❤️ using Claude Code and OpenSpec**

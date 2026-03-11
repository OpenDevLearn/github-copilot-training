# Lab 1: Skill Authoring

## Objective
Author a complete, parameterized skill (`SKILL.md`) for enforcing clean architecture dependency rules, including output schema, failure modes, and test cases.

## Prerequisites
- Understanding of clean/layered architecture
- Familiarity with Python imports and module structure

## Tasks

### Task 1: Define the architecture rules config
Create `.architecture.yaml`:

```yaml
layers:
  - name: domain
    path: "src/domain/**"
    allowed_imports: []
  - name: application
    path: "src/application/**"
    allowed_imports: ["domain"]
  - name: infrastructure
    path: "src/infrastructure/**"
    allowed_imports: ["domain", "application"]
  - name: presentation
    path: "src/presentation/**"
    allowed_imports: ["application"]
    forbidden_imports: ["infrastructure", "domain"]
```

**Acceptance:** Config file parses as valid YAML. All layers have explicit rules.

### Task 2: Author the skill
Create `skills/enforce-clean-architecture/SKILL.md` with:

1. **Frontmatter**: name, version (1.0.0), parameters, dependencies, tags
2. **Purpose**: One-paragraph description
3. **Parameters**: `target_path` (required), `architecture_config` (optional), `strict_mode` (optional)
4. **Procedure**: Step-by-step (numbered, ≤8 steps)
5. **Output Schema**: JSON schema with summary + violations array
6. **Failure Modes**: At least 3 with recovery
7. **Test Cases**: At least 5

**Acceptance Criteria:**
- [ ] Frontmatter parses as valid YAML
- [ ] All 3 parameters documented with type and description
- [ ] Procedure has ≤8 steps
- [ ] Output schema is valid JSON
- [ ] ≥3 failure modes with recovery
- [ ] ≥5 test cases covering happy path, violations, edge cases

### Task 3: Create sample codebase for testing

```bash
mkdir -p src/{domain,application,infrastructure,presentation}

# Clean: domain has no imports
cat > src/domain/entities.py << 'EOF'
class User:
    def __init__(self, id: str, name: str, email: str):
        self.id = id
        self.name = name
        self.email = email
EOF

# Clean: application imports only domain
cat > src/application/user_service.py << 'EOF'
from src.domain.entities import User

class UserService:
    def create_user(self, name: str, email: str) -> User:
        return User(id="generated", name=name, email=email)
EOF

# VIOLATION: presentation imports infrastructure directly
cat > src/presentation/api.py << 'EOF'
from src.infrastructure.db import UserRepository  # VIOLATION
from src.application.user_service import UserService  # OK

def get_users():
    repo = UserRepository()
    return repo.find_all()
EOF

# Clean: infrastructure imports domain + application
cat > src/infrastructure/db.py << 'EOF'
from src.domain.entities import User

class UserRepository:
    def find_all(self) -> list[User]:
        return []
EOF
```

**Acceptance:** Codebase has exactly 1 architectural violation (presentation→infrastructure).

### Task 4: Invoke the skill via agent
Use Copilot agent mode to invoke the skill:

```
@agent Use the enforce-clean-architecture skill on src/ with the .architecture.yaml config
```

**Acceptance:**
- Skill detects the `presentation→infrastructure` violation in `src/presentation/api.py`
- Output matches the declared JSON schema
- Clean imports are not flagged

### Task 5: Test edge cases
Create additional test files to trigger all test cases:
1. External import (`import requests`) → should be skipped
2. Domain importing infrastructure → should be flagged
3. Empty file → should be handled gracefully
4. File with only comments → should be handled gracefully

**Acceptance:** All 5 test cases from the skill definition produce expected results.

## Evidence to Commit
- [ ] `skills/enforce-clean-architecture/SKILL.md` — Complete skill definition
- [ ] `.architecture.yaml` — Architecture rules
- [ ] `src/` — Sample codebase with known violation
- [ ] `evidence/lab1-skill-output.json` — Skill execution output
- [ ] `evidence/lab1-test-results.md` — Test case results (5/5)

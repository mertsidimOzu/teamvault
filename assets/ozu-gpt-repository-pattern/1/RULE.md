## Repository Pattern

Database access goes through repositories — never query `db` directly in routers or use cases.

### Structure

```
repositories/
├── base.py              # BaseRepository[T] — generic CRUD
├── user_repository.py
├── conversation_repository.py  # + MessageRepository
└── gpt_repository.py           # + GPTAccessRepository
```

### Using BaseRepository

```python
from repositories.base import BaseRepository
from models import MyModel

class MyRepository(BaseRepository[MyModel]):
    def __init__(self, db: Session):
        super().__init__(db, MyModel)

    # Built-in: get_by_id, get_all, create, update, delete, count, exists
    # Add custom queries here:
    def get_by_email(self, email: str) -> Optional[MyModel]:
        return self.db.query(self.model).filter(self.model.email == email).first()
```

### Dependency Injection in Routers

```python
from fastapi import APIRouter, Depends
from container.dependencies import get_user_repository, get_repository_container
from repositories import UserRepository

router = APIRouter()

# Single repo
@router.get("/users/{id}")
def get_user(id: str, repo: UserRepository = Depends(get_user_repository)):
    ...

# Multiple repos — use container
from container.dependencies import RepositoryContainer, get_repository_container

@router.post("/conversations")
def create_conversation(container: RepositoryContainer = Depends(get_repository_container)):
    user = container.user.get_by_id(...)
    conv = container.conversation.create(...)
```

### Rules

- Add new repos to `repositories/__init__.py` and `container/dependencies.py`
- Repository methods only do DB queries — no business logic
- Business logic belongs in services or use cases
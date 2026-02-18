## Error Handling

Use `AppException` subclasses from `core/exceptions.py`. Never use bare `HTTPException` in service/use-case code.

### Exception Hierarchy

```python
from core.exceptions import (
    NotFoundError,        # 404 NOT_FOUND
    UnauthorizedError,    # 401 UNAUTHORIZED
    ForbiddenError,       # 403 FORBIDDEN
    ValidationError,      # 422 VALIDATION_ERROR
    ConflictError,        # 409 CONFLICT
    ServiceUnavailableError,  # 503 SERVICE_UNAVAILABLE
    # Domain-specific:
    LLMException, RAGException, VectorDBException,
    EmbeddingException, GraphRAGException,
    DocumentProcessingException, DataQueryException,
    SQLValidationException, RateLimitException,
)
```

### When to Use What

| Layer | Use |
|-------|-----|
| Routers | Catch `AppException`, re-raise or return `ApiResponse.fail()` |
| Services/use cases | Raise `AppException` subclasses |
| Never | Bare `raise Exception(...)` or `HTTPException` in services |

### Patterns

```python
# Raise domain exceptions in services
def get_user(user_id: str) -> User:
    user = db.query(User).filter(User.id == user_id).first()
    if not user:
        raise NotFoundError(resource="Kullanıcı", identifier=user_id)
    return user

# Domain-specific with context
raise LLMException(message="Model yanıt vermedi", provider="openai", model="gpt-4o")
raise RAGException(message="Belge işlenemedi", operation="chunk")

# SECURITY: Never expose SQL in DataQueryException
raise DataQueryException(message="Sorgu çalıştırılamadı", operation="execute")
# Log the SQL separately, don't put it in details={}
```

### Error Response Format

All errors produce `{ success: false, error: { code, message, details? } }` via `ErrorResponse.create()` in `core/responses.py`.
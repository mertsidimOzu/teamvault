## SSE Streaming

All LLM streaming responses use helpers from `core/sse.py`. Never set SSE headers manually.

### Core Utilities

```python
from core.sse import create_sse_response, sse_event, SSE_DONE
```

| Helper | Returns | Usage |
|--------|---------|-------|
| `sse_event(**kwargs)` | `str` | Format one SSE event |
| `SSE_DONE` | `"data: [DONE]\n\n"` | Signal stream end |
| `create_sse_response(gen)` | `StreamingResponse` | Wrap async generator |

### Pattern

```python
from fastapi.responses import StreamingResponse
from core.sse import create_sse_response, sse_event, SSE_DONE

@router.post("/chat")
async def stream_chat(request: ChatRequest) -> StreamingResponse:
    async def generate():
        try:
            async for chunk in llm_stream(request):
                yield sse_event(content=chunk)          # {"content": "..."}
            yield sse_event(type="done", content="")    # signal before DONE
            yield SSE_DONE
        except Exception as e:
            yield sse_event(error=str(e), type="error")
            yield SSE_DONE

    return create_sse_response(generate())
```

### Frontend Consumption

```javascript
const source = new EventSource("/api/v1/chat");
source.onmessage = (e) => {
    if (e.data === "[DONE]") { source.close(); return; }
    const { content, error } = JSON.parse(e.data);
};
```

### Never

- Don't set `Cache-Control`, `Connection`, `X-Accel-Buffering` manually — `create_sse_response` handles this
- Don't use `text/plain` for streaming — always `text/event-stream`
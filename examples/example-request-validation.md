# OrderAPI — Code Intelligence Report

**Objective:** How would we add custom request validation middleware?  
**Repository:** /tmp/test-fastapi-app

---

## Executive Summary

Custom validation middleware hooks into the request pipeline in `app/middleware.py`. Add a new async function in `setup_middleware()` using `@app.middleware("http")` decorator, validate request headers/body, and raise `HTTPException` for failures. All requests pass through this middleware before reaching route handlers.

---

## Code Architecture

**Entry point:** `main.py` calls `setup_middleware(app)` at line 23, which registers all middleware including CORS (lines 12-16) and request logging (lines 21-23).

**Middleware flow:** FastAPI request → `app/middleware.py:setup_middleware()` registered middleware chain → route handler

**Key modules:**
- `main.py` (34 lines) - Application setup, middleware registration call at line 23
- `app/middleware.py` (27 lines) - Middleware registration and implementation
- `app/routes/order_router.py` (42 lines) - Route handlers that receive validated requests
- `app/routes/user_router.py` (33 lines) - User endpoints

---

## Answer to Your Question

**To add custom request validation middleware:**

1. **Add validation middleware** in `app/middleware.py`:

   ```python
   @app.middleware("http")
   async def validate_custom_headers(request, call_next):
       # Your validation logic here
       if "X-Required-Header" not in request.headers:
           raise HTTPException(status_code=400, detail="Missing required header")
       response = await call_next(request)
       return response
   ```

2. **Where to add it:**
   - In `app/middleware.py:setup_middleware()` function (lines 8-23)
   - Add another `@app.middleware("http")` block after line 23
   - It will run in registration order before route handlers

3. **Current middleware structure:**
   - Line 12-16: `CORSMiddleware` added via `app.add_middleware()`
   - Line 21-23: Logging middleware via `@app.middleware("http")` decorator

4. **Request execution order:**
   - CORS middleware (line 12) → Your validation middleware → Logging middleware (line 21) → Route handler

5. **To validate specific requests only:**
   - Check `request.url.path` to apply validation only to `/orders/*` endpoints
   - Check `request.method` to apply only to POST/PUT requests
   - Access `request.headers` or parse `request.body()` for validation data

6. **Validation error handling:**
   - Raise `HTTPException(status_code=400, detail="...")` for validation failures
   - Already imported at top of `order_router.py` line 1, same pattern used in route handlers

---

## Extension/Integration Points

- **Add request validation:** Create middleware function in `setup_middleware()`, use `@app.middleware("http")` decorator, raise `HTTPException` for invalid requests
- **Add header-based auth:** Create middleware checking `request.headers` for tokens before `call_next()`
- **Add rate limiting:** Middleware can check `request.client.host` and track request counts
- **Add request/response logging:** Logging middleware already in place (line 21); extend to log request/response bodies
- **Extend route handlers:** Add validators to Pydantic models (`User`, `Order` in `app/routes/`) or use FastAPI's `Depends()` for endpoint-specific validation

---

## Code Reading Guide

1. Start: `main.py` line 23 to see `setup_middleware(app)` call (line 1-34)
2. Read: `app/middleware.py` entire file to understand current middleware patterns (lines 1-27)
3. Study: `app/routes/order_router.py` to see `HTTPException` usage in handlers (lines 1, 20, 34)
4. Reference: FastAPI documentation on middleware and `HTTPException` for examples
5. Test: Create a test request and verify middleware executes before handlers

---

## Strengths & Weaknesses

| Strengths | Weaknesses |
|-----------|-----------|
| Middleware defined in single `setup_middleware()` function; easy to add/remove middleware | All middleware runs on every request regardless of path; no per-route filtering without checking `request.url.path` |
| `@app.middleware("http")` decorator is clear and standard FastAPI pattern | No middleware registry or ordered list; middleware execution order implicit in code order |
| Can access full request object; can validate headers, body, method, path | Body validation via `await request.body()` consumes the stream; must be read once or cached |
| HTTPException pattern already used throughout route handlers for consistency | Validation errors not logged; silent drops if middleware silently returns response |

---

Claude Haiku 4.5 | Jun-18-2026 00:18 UTC

# Django REST Framework — Code Intelligence Report

**Objective:** How would we add custom request validation middleware?  
**Repository:** /tmp/djangorestframework

---

## Executive Summary

Custom validation middleware hooks into DRF's middleware chain via the `APIView.dispatch()` method. To add request validation, you'd create a custom middleware class inheriting from `BaseMiddleware` in `rest_framework/middleware.py` and register it in Django's `MIDDLEWARE` setting.

---

## Code Architecture

- **Entry point:** Django WSGI handler → `rest_framework/views.py:APIView.dispatch()`
- **Middleware chain:** `rest_framework/middleware.py` contains `BaseMiddleware`, `APIMiddleware`
- **Request processing:** Validation happens in `decorators.py` (for function views) or `APIView.initial()` (for class views)
- **Exception handling:** `rest_framework/exceptions.py` defines all validation exceptions
- **Permissions:** `rest_framework/permissions.py` has permission classes that run during `APIView.check_permissions()`

**Key modules:**
- `views.py` — APIView base class, request dispatch
- `middleware.py` — middleware base and authentication middleware
- `decorators.py` — function-based view decorators
- `exceptions.py` — validation and error classes
- `permissions.py` — permission checking logic

---

## Answer to Your Question

**How to add custom request validation middleware:**

1. Create middleware class in `your_app/middleware.py`:

```python
from rest_framework.middleware import BaseMiddleware

class CustomValidationMiddleware(BaseMiddleware):
    def process_request(self, request):
        # Your validation logic here
        if not self.is_valid(request):
            raise ValidationError("Invalid request")
        return request
```

2. Where it hooks in:

   - `rest_framework/views.py:APIView.dispatch()` (line 487) calls `self.middleware_chain(request)`
   - Middleware chain defined in `settings.MIDDLEWARE`
   - Your middleware's `process_request()` runs before handlers

3. Request flow:

   Django WSGI
   → Django middleware stack (`settings.MIDDLEWARE`)
   → Your `CustomValidationMiddleware.process_request()`
   → `APIView.dispatch()` → `APIView.initial()` (permission checks, versioning, etc.)
   → Handler (GET/POST/etc.)

4. To register:

   Add to `settings.py`:
   ```python
   MIDDLEWARE = [
       ...
       'your_app.middleware.CustomValidationMiddleware',
   ]
   ```

---

## Extension/Integration Points

- **Add field validation:** Subclass `Serializer` in `serializers.py:BaseSerializer`, override `validate_<field>()`
- **Add custom permissions:** Subclass `BasePermission` in `permissions.py` and implement `has_permission()`
- **Add request preprocessing:** Create middleware inheriting `BaseMiddleware`, hook in `process_request()`
- **Add custom exceptions:** Create exception classes in `exceptions.py` inheriting from `APIException`

---

## Code Reading Guide

1. Start: `views.py:APIView` — understand request dispatch entry point
2. Trace: `APIView.dispatch()` → `APIView.initial()` to see where permissions/validation run
3. Study: `middleware.py` to understand middleware chain integration
4. Examine: `permissions.py` and `decorators.py` for validation patterns
5. Reference: `exceptions.py` for available error types

---

## Strengths & Weaknesses

| Strengths | Weaknesses |
|-----------|-----------|
| Clear separation: APIView.dispatch() has defined hook points (initial(), permission_classes) | Permission checking tied to APIView; function-based views need decorators, adding friction |
| Middleware pattern familiar to Django devs | BaseMiddleware inheritance required; could be simpler for basic cases |
| Exception classes follow hierarchy; easy to catch/handle specific errors | Exception hierarchy spans two files (exceptions.py, status.py); harder to discover |

---

[Provider/Tool] | [Model] | [Timestamp UTC]

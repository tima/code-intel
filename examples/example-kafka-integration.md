# OrderAPI — Code Intelligence Report

**Objective:** How would we integrate Kafka to replace Celery for event publishing?  
**Repository:** /tmp/test-fastapi-app

---

## Executive Summary

Event publishing hooks through the pluggable `EventPublisher` interface defined in `app/services/events.py`. To integrate Kafka, create a `KafkaPublisher` class inheriting from `EventPublisher`, implement the `publish()` method to send to Kafka, and swap the publisher via `set_publisher()` at application startup in `main.py`.

---

## Code Architecture

**Entry point:** `main.py` initializes FastAPI app and registers routers. The lifespan context manager at line 8 controls startup/shutdown.

**Key modules:**
- `main.py` (34 lines) - Application initialization, route registration, server entry point
- `app/routes/order_router.py` (42 lines) - Order endpoints that publish events
- `app/services/events.py` (30 lines) - Event publishing abstraction and Celery implementation
- `app/middleware.py` (27 lines) - Request middleware (CORS, logging)
- `app/config.py` (9 lines) - Configuration management

**Flow:** HTTP request → router handler → `publish_event()` call → `EventPublisher.publish()` implementation → external system (currently Celery).

---

## Answer to Your Question

**To integrate Kafka for event publishing:**

1. **Create Kafka publisher** in `app/services/events.py`:

   ```python
   from kafka import KafkaProducer
   
   class KafkaPublisher(EventPublisher):
       def __init__(self, bootstrap_servers: str):
           self.producer = KafkaProducer(bootstrap_servers=bootstrap_servers)
       
       def publish(self, event_type: str, payload: Dict[str, Any]) -> None:
           self.producer.send(event_type, json.dumps(payload).encode())
   ```

2. **Where events are triggered:** 
   - `app/routes/order_router.py:create_order()` line 26 calls `background_tasks.add_task(publish_event, ...)`
   - `app/routes/order_router.py:update_order()` line 41 calls the same
   - Both route to `app/services/events.py:publish_event()` line 28

3. **Current implementation chain:**
   - Global `_publisher` initialized to `CeleryPublisher()` at line 21 of `events.py`
   - `publish_event()` delegates to `_publisher.publish()` at line 30

4. **Integration points:**
   - In `main.py` at startup (line 8 or within lifespan), call: `set_publisher(KafkaPublisher("localhost:9092"))`
   - This swaps the global `_publisher` without touching route handlers
   - All calls to `publish_event()` automatically route through Kafka

5. **No changes needed to:**
   - Route handlers (`order_router.py`, `user_router.py`)
   - Event publication calls (line 26, 41 in `order_router.py`)

---

## Extension/Integration Points

- **Swap event publishers:** Call `set_publisher(KafkaPublisher(...))` at application startup to replace Celery with any publisher implementing `EventPublisher` interface
- **Add new event types:** Existing code in `order_router.py` already publishes "order.created" and "order.updated" — just add new calls to `publish_event(event_type, payload)` with new event names
- **Add request logging middleware:** Call `app.add_middleware()` in `setup_middleware()` function (`app/middleware.py:8`)
- **Extend API routes:** Create new router module in `app/routes/` following the pattern of `user_router.py` and register with `app.include_router()` in `main.py` line 25-26

---

## Code Reading Guide

1. Start: `main.py` to understand app initialization and route registration (lines 1-34)
2. Trace: `app/routes/order_router.py` to see where `publish_event()` is called (lines 21-41)
3. Study: `app/services/events.py` to understand the `EventPublisher` abstraction (lines 7-30)
4. Reference: `app/middleware.py` to see how middleware is registered (lines 8-23)
5. Review: `app/config.py` for configuration patterns (lines 1-9)

---

## Strengths & Weaknesses

| Strengths | Weaknesses |
|-----------|-----------|
| Clean `EventPublisher` abstraction allows swapping Celery for Kafka, Redis, etc. without touching route code | No error handling in `CeleryPublisher.publish()` — failed publishes are silently lost |
| Event publishing decoupled from route handlers via `publish_event()` function | Global `_publisher` state makes concurrent publisher swaps unsafe; needs locking for production |
| Middleware registration pattern is extensible; easy to add new middleware in `setup_middleware()` | No retry logic if Kafka producer fails; events may be dropped |
| Type hints throughout (`Dict[str, Any]`, model classes) make code intent clear | Missing logging in event publishing; no visibility into published events |

---

Claude Haiku 4.5 | Jun-18-2026 00:18 UTC

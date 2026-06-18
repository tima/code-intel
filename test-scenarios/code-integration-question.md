# Test Scenario: Code Integration Question

**User asks:** "How would we integrate Kafka into this FastAPI app?"

**Expected flow:**

1. Repo validation passes (FastAPI app present)
2. Passive scan identifies: entry point in `main.py`, async task handling in `celery_tasks.py`
3. Interview asks: "What specifically do you want to understand about this code?"
4. User clarifies: "Where would we add event publishing to Kafka instead of Celery?"
5. Analysis traces: event publication points → current Celery integration → where Kafka producer would hook in
6. Report answers: "Event publishing happens in `services/events.py:publish_event()`. Currently calls Celery task. To add Kafka, you'd implement `EventPublisher` interface and pass to `Application` constructor, similar to how `CeleryPublisher` is wired at line 42."

**Success criteria:**
- No anti-pattern gates block the request
- Interview produces a specific code understanding goal
- Report directly answers the integration question
- Report includes exact file names and function names

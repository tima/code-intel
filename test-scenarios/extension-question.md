# Test Scenario: Extension/Customization Question

**User asks:** "How would we add custom validation rules to this form handler?"

**Expected flow:**

1. Repo validation passes (Python web app present)
2. Passive scan identifies: form handling in `forms.py`, validation in `validators.py`
3. Interview asks: "What specifically do you want to understand about this code?"
4. User clarifies: "Where would we hook in custom field-level validators?"
5. Analysis traces: form submission → validation chain → current validator architecture
6. Report answers: "Validation happens in `validators.py:validate_form()`. Current validators in `VALIDATORS` dict. To add custom validators: create `CustomValidator` class inheriting `BaseValidator`, implement `validate()` method, register in `VALIDATORS` dict at line 78."

**Success criteria:**
- Report includes exact extension points (where to add code)
- Report shows code structure supporting customization
- Report provides minimal code example of how to extend

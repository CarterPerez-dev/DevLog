# RuleEngine.score_request

**Type:** Security Review
**Repository:** Cybersecurity-Projects
**File:** PROJECTS/advanced/ai-threat-detection/backend/app/core/detection/rules.py
**Language:** python
**Lines:** 91-135
**Complexity:** 13.0

---

## Source Code

```python
def score_request(
        self,
        features: dict[str, int | float | bool | str],
        entry: ParsedLogEntry,
    ) -> RuleResult:
        """
        Evaluate all rules against a request and return a composite score.
        """
        matched: list[tuple[str, float]] = []

        uri = entry.path
        if entry.query_string:
            uri = f"{entry.path}?{entry.query_string}"

        for rule in _PATTERN_RULES:
            if rule.pattern.search(uri):
                matched.append((rule.name, rule.score))

        if DOUBLE_ENCODED.search(uri):
            matched.append(("DOUBLE_ENCODING", _DOUBLE_ENCODING_SCORE))

        ua_lower = entry.user_agent.lower()
        if any(sig in ua_lower for sig in SCANNER_USER_AGENTS):
            matched.append(("SCANNER_UA", _SCANNER_UA_SCORE))

        for trule in _THRESHOLD_RULES:
            value = features.get(trule.feature_key, 0)
            if isinstance(value, int | float) and value > trule.threshold:
                matched.append((trule.name, trule.score))

        if not matched:
            return RuleResult(threat_score=0.0, severity="LOW")

        scores = sorted([s for _, s in matched], reverse=True)
        threat_score = min(
            scores[0] + _BOOST_PER_ADDITIONAL_RULE * (len(scores) - 1),
            1.0,
        )

        return RuleResult(
            threat_score=threat_score,
            severity=_classify_severity(threat_score),
            matched_rules=[name for name, _ in matched],
            component_scores=dict(matched),
        )
```

---

## Security Review

### Security Review for `RuleEngine.score_request`

#### Vulnerabilities Found:

1. **Info - Input Validation Gaps:**
   - Lines 7-9: URI construction does not validate or sanitize query strings, which could lead to injection vulnerabilities if the input is manipulated.

2. **Low - Error Handling:**
   - Lines 15-18: No explicit error handling for `features.get()`. If a key is missing, it returns `0`, potentially leading to unexpected behavior.

3. **Info - Insecure Deserialization:**
   - The code does not deserialize any data, so this issue is not applicable here.

#### Attack Vectors:

- An attacker could manipulate the query string or user agent to trigger false positives or bypass rules.
- Missing input validation can allow injection attacks if the URI construction is misused.

#### Recommended Fixes:

1. **Input Validation:**
   - Validate and sanitize `entry.query_string` before appending it to `uri`.
     ```python
     uri = entry.path
     query_string = entry.query_string.strip()  # Remove leading/trailing whitespace
     if query_string:
         uri += f"?{query_string}"
     ```

2. **Error Handling:**
   - Add a try-except block for `features.get()` to handle missing keys gracefully.
     ```python
     value = features.get(trule.feature_key, 0)
     ```

3. **Overall Security Posture:**

The overall security posture is good but can be improved by ensuring robust input validation and error handling. Addressing the above issues will enhance the resilience of the code against potential attacks.

```python
def score_request(
        self,
        features: dict[str, int | float | bool | str],
        entry: ParsedLogEntry,
    ) -> RuleResult:
        """
        Evaluate all rules against a request and return a composite score.
        """
        matched: list[tuple[str, float]] = []

        uri = entry.path
        query_string = entry.query_string.strip()  # Remove leading/trailing whitespace
        if query_string:
            uri += f"?{query_string}"

        for rule in _PATTERN_RULES:
            if rule.pattern.search(uri):
                matched.append((rule.name, rule.score))

        if DOUBLE_ENCODED.search(uri):
            matched.append(("DOUBLE_ENCODING", _DOUBLE_ENCODING_SCORE))

        ua_lower = entry.user_agent.lower()
        if any(sig in ua_lower for sig in SCANNER_USER_AGENTS):
            matched.append(("SCANNER_UA", _SCANNER_UA_SCORE))

        for trule in _THRESHOLD_RULES:
            value = features.get(trule.feature_key, 0)
            try:
                if isinstance(value, (int, float)) and value > trule.threshold:
                    matched.append((trule.name, trule.score))
            except TypeError:  # Handle unexpected types
                pass

        if not matched:
            return RuleResult(threat_score=0.0, severity="LOW")

        scores = sorted([s for _, s in matched], reverse=True)
        threat_score = min(
            scores[0] + _BOOST_PER_ADDITIONAL_RULE * (len(scores) - 1),
            1.0,
        )

        return RuleResult(
            threat_score=threat_score,
            severity=_classify_severity(threat_score),
            matched_rules=[name for name, _ in matched],
            component_scores=dict(matched),
        )
```

---

*Generated by CodeWorm on 2026-02-18 21:53*

# lookup_whois

**Type:** Performance Analysis
**Repository:** Cybersecurity-Projects
**File:** PROJECTS/beginner/dns-lookup/dnslookup/whois_lookup.py
**Language:** python
**Lines:** 63-127
**Complexity:** 23.0

---

## Source Code

```python
def lookup_whois(domain: str) -> WhoisResult:
    """
    Perform WHOIS lookup for a domain
    """
    result = WhoisResult(domain = domain)

    try:
        w = whois.whois(domain)

        if w is None or (hasattr(w,
                                 "domain_name")
                         and w.domain_name is None):
            result.error = "Domain not found or WHOIS data unavailable"
            return result

        result.registrar = w.registrar if hasattr(
            w,
            "registrar"
        ) else None
        result.creation_date = w.creation_date if hasattr(
            w,
            "creation_date"
        ) else None
        result.expiration_date = w.expiration_date if hasattr(
            w,
            "expiration_date"
        ) else None
        result.updated_date = w.updated_date if hasattr(
            w,
            "updated_date"
        ) else None

        if hasattr(w, "status"):
            status = w.status
            if isinstance(status, str):
                result.status = [status]
            elif isinstance(status, list):
                result.status = status
            else:
                result.status = []

        if hasattr(w, "name_servers") and w.name_servers:
            ns = w.name_servers
            if isinstance(ns, str):
                result.name_servers = [ns.lower()]
            elif isinstance(ns, list):
                result.name_servers = [n.lower() for n in ns if n]
            else:
                result.name_servers = []

        if hasattr(w, "org"):
            result.registrant = w.org
        elif hasattr(w, "name"):
            result.registrant = w.name

        if hasattr(w, "country"):
            result.registrant_country = w.country

        if hasattr(w, "dnssec"):
            result.dnssec = str(w.dnssec) if w.dnssec else None

    except Exception as e:
        result.error = str(e)

    return result
```

---

## Performance Analysis

### Performance Analysis

**Time Complexity:** The function has a time complexity of \(O(1)\) since the operations are constant-time checks and assignments, assuming `whois.whois()` returns in a reasonable amount of time.

**Space Complexity:** The space complexity is also \(O(1)\), as the variables used do not scale with input size; only a few attributes are checked for existence and assigned.

**Bottlenecks or Inefficiencies:**
- **Redundant Checks:** Multiple `hasattr()` checks are redundant. For instance, checking if an attribute exists before accessing it can be done once.
- **Error Handling:** The exception handling is broad, which might hide specific errors. Consider catching more specific exceptions.

**Optimization Opportunities:**
- Combine multiple `hasattr()` and conditional checks into a single pass to reduce redundancy.
- Use type hints for better readability and potential static analysis tools.

```python
def lookup_whois(domain: str) -> WhoisResult:
    result = WhoisResult(domain=domain)
    try:
        w = whois.whois(domain)

        if not w or (hasattr(w, "domain_name") and not w.domain_name):
            result.error = "Domain not found or WHOIS data unavailable"
            return result

        for attr in ["registrar", "creation_date", "expiration_date", "updated_date"]:
            if hasattr(w, attr):
                setattr(result, attr, getattr(w, attr))

        status = getattr(w, "status", [])
        if isinstance(status, str):
            result.status = [status]
        elif isinstance(status, list):
            result.status = status

        name_servers = getattr(w, "name_servers", None)
        if name_servers:
            if isinstance(name_servers, str):
                result.name_servers = [name_servers.lower()]
            else:
                result.name_servers = [n.lower() for n in name_servers if n]

        registrant = getattr(w, "org", getattr(w, "name", None))
        if registrant:
            result.registrant = registrant

        country = getattr(w, "country", None)
        if country:
            result.registrant_country = country

        dnssec = getattr(w, "dnssec", None)
        if dnssec is not None:
            result.dnssec = str(dnssec)

    except whois.parser.PywhoisError as e:
        result.error = f"WHOIS query failed: {e}"
    except Exception as e:
        result.error = f"An unexpected error occurred: {e}"

    return result
```

**Resource Usage Concerns:**
- Ensure that `whois.whois()` does not block the main thread if running in an async context.
- Consider using a connection pool for repeated calls to avoid resource leaks.

By optimizing redundant checks and handling exceptions more specifically, you can improve readability and maintainability while keeping performance efficient.

---

*Generated by CodeWorm on 2026-02-18 09:12*

# SQLiScanner

**Type:** Class Documentation
**Repository:** Cybersecurity-Projects
**File:** PROJECTS/intermediate/api-security-scanner/backend/scanners/sqli_scanner.py
**Language:** python
**Lines:** 23-338
**Complexity:** 0.0

---

## Source Code

```python
class SQLiScanner(BaseScanner):
    """
    Tests for SQL injection vulnerabilities

    Detects:
    - Error-based SQLi (database error messages)
    - Boolean-based blind SQLi (response differences)
    - Time-based blind SQLi (response timing analysis)

    Uses payloads covering MySQL, PostgreSQL, MSSQL, Oracle
    """
    def scan(self) -> TestResultCreate:
        """
        Execute SQL injection tests

        Returns:
            TestResultCreate: Scan result with findings
        """
        error_based_test = self._test_error_based_sqli()
        if error_based_test["vulnerable"]:
            return self._create_vulnerable_result(
                details =
                f"Error-based SQL injection detected: {error_based_test['database_type']}",
                evidence = error_based_test,
                severity = Severity.CRITICAL,
                recommendations = [
                    "Use parameterized queries (prepared statements)",
                    "Never concatenate user input into SQL queries",
                    "Implement input validation and sanitization",
                    "Disable detailed error messages in production",
                    "Use ORM frameworks with proper escaping",
                ],
            )

        boolean_based_test = self._test_boolean_based_sqli()
        if boolean_based_test["vulnerable"]:
            return self._create_vulnerable_result(
                details = "Boolean-based blind SQL injection detected",
                evidence = boolean_based_test,
                severity = Severity.CRITICAL,
                recommendations = [
                    "Use parameterized queries for all database operations",
                    "Implement proper input validation",
                    "Avoid exposing different responses for true/false conditions",
                ],
            )

        time_based_test = self._test_time_based_sqli()
        if time_based_test["vulnerable"]:
            return self._create_vulnerable_result(
                details =
                f"Time-based blind SQL injection detected: {time_based_test['database_type']}",
                evidence = time_based_test,
                severity = Severity.CRITICAL,
                recommendations = [
                    "Use parameterized queries exclusively",
                    "Implement strict input validation",
                    "Monitor for unusual response time patterns",
                ],
            )

        return TestResultCreate(
            test_name = TestType.SQLI,
            status = ScanStatus.SAFE,
            severity = Severity.INFO,
            details = "No SQL injection vulnerabilities detected",
            evidence_json = {
                "error_based_test": error_based_test,
                "boolean_based_test": boolean_based_test,
                "time_based_test": time_based_test,
            },
            recommendations_json = [
                "Continue using parameterized q
```

---

## Class Documentation

### SQLiScanner Class Documentation

**Class Responsibility and Purpose:**
The `SqliScanner` class is responsible for detecting potential SQL injection vulnerabilities by executing various tests, including error-based, boolean-based blind, and time-based blind SQL injection techniques. It leverages predefined payloads to simulate attacks and analyze responses from the target application.

**Public Interface (Key Methods):**
- **`scan()`**: Executes a comprehensive scan for SQL injection vulnerabilities and returns a `TestResultCreate` object detailing the findings.
- **`_test_error_based_sqli()`**: Tests for error-based SQL injection by looking for specific database error signatures in response text.
- **`_test_boolean_based_sqli()`**: Tests for boolean-based blind SQL injection by comparing responses under different conditions.

**Design Patterns Used:**
The class employs the **Strategy Pattern** through its use of different test methods (`_test_error_based_sqli`, `_test_boolean_based_sqli`, etc.) to handle various types of SQL injection tests. Each method implements a specific strategy for detecting vulnerabilities, making the class flexible and extensible.

**Relationship to Other Classes:**
- The `SqliScanner` interacts with the `BaseScanner` class by inheriting from it.
- It uses the `SQLiPayloads` class to manage predefined payloads for different types of SQL injection tests.
- The `TestResultCreate` class is used to store and return the results of the scan, providing a structured format for findings.

**State Management Approach:**
The state management within the `SqliScanner` class is primarily focused on the outcome of each test. The `_test_error_based_sqli` and `_test_boolean_based_sqli` methods maintain their own internal states to track whether a vulnerability was detected or not, which influences the overall scan result returned by the `scan` method.

This class fits into the architecture as part of an API security scanner suite, where it is responsible for identifying critical vulnerabilities related to SQL injection.

---

*Generated by CodeWorm on 2026-02-18 10:28*

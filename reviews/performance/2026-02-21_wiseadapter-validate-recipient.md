# WiseAdapter.validate_recipient

**Type:** Performance Analysis
**Repository:** stripe-referral
**File:** src/stripe_referral/adapters/wise.py
**Language:** python
**Lines:** 176-229
**Complexity:** 12.0

---

## Source Code

```python
def validate_recipient(
        self,
        recipient_data: dict[str,
                             Any]
    ) -> RecipientValidation:
        """
        Validate Wise recipient data
        """
        required_fields = ["account_holder_name", "details"]

        missing_fields = [
            field for field in required_fields
            if field not in recipient_data
        ]

        if missing_fields:
            return RecipientValidation(
                valid = False,
                error =
                f"Missing required fields: {', '.join(missing_fields)}",
            )

        account_holder_name = recipient_data.get(
            "account_holder_name",
            ""
        )
        if not account_holder_name or len(account_holder_name) < 2:
            return RecipientValidation(
                valid = False,
                error = "Invalid account holder name",
            )

        details = recipient_data.get("details", {})
        if not isinstance(details, dict):
            return RecipientValidation(
                valid = False,
                error = "Details must be a dictionary",
            )

        account_type = recipient_data.get("account_type", "aba")
        if account_type == "aba":
            if "accountNumber" not in details or "routingNumber" not in details:
                return RecipientValidation(
                    valid = False,
                    error =
                    "Missing accountNumber or routingNumber for ABA transfer",
                )
        elif account_type == "iban" and "iban" not in details:
            return RecipientValidation(
                valid = False,
                error = "Missing IBAN for IBAN transfer",
            )

        return RecipientValidation(valid = True)
```

---

## Performance Analysis

### Performance Analysis

#### Time Complexity
The time complexity of the `validate_recipient` function is **O(n)**, where `n` is the number of required fields (currently 2). The primary operations involve checking for missing fields and validating specific conditions.

#### Space Complexity and Memory Allocation
- The space complexity is **O(1)** as there are no significant data structures that grow with input size. However, the function creates temporary lists and dictionaries which consume a fixed amount of memory.
- No blocking calls or N+1 query patterns are present.

#### Bottlenecks or Inefficiencies
- Redundant checks: The `account_holder_name` is checked for both existence and length, which can be combined into one condition.
- Multiple return statements: While this makes the code readable, it can lead to unnecessary function calls if a validation fails early. Consider using an accumulator pattern.

#### Optimization Opportunities
1. **Combine Conditions**: Simplify the `account_holder_name` check:
   ```python
   account_holder_name = recipient_data.get("account_holder_name", "")
   if not account_holder_name or len(account_holder_name) < 2:
       return RecipientValidation(valid=False, error="Invalid account holder name")
   ```

2. **Early Return**: Use an accumulator pattern to reduce the number of return statements and improve readability:
   ```python
   def validate_recipient(self, recipient_data: dict[str, Any]) -> RecipientValidation:
       required_fields = ["account_holder_name", "details"]
       missing_fields = [field for field in required_fields if field not in recipient_data]
       
       if missing_fields:
           return RecipientValidation(valid=False, error=f"Missing required fields: {', '.join(missing_fields)}")
       
       account_holder_name = recipient_data.get("account_holder_name", "")
       if not account_holder_name or len(account_holder_name) < 2:
           return RecipientValidation(valid=False, error="Invalid account holder name")

       details = recipient_data.get("details", {})
       if not isinstance(details, dict):
           return RecipientValidation(valid=False, error="Details must be a dictionary")

       account_type = recipient_data.get("account_type", "aba")
       if account_type == "aba":
           if "accountNumber" not in details or "routingNumber" not in details:
               return RecipientValidation(
                   valid=False,
                   error="Missing accountNumber or routingNumber for ABA transfer",
               )
       elif account_type == "iban" and "iban" not in details:
           return RecipientValidation(valid=False, error="Missing IBAN for IBAN transfer")

       return RecipientValidation(valid=True)
   ```

#### Resource Usage Concerns
- Ensure that all file handles or network connections are properly managed using context managers to avoid resource leaks. However, no such issues are present in the given code snippet.

By applying these optimizations, you can improve both readability and performance of the function.

---

*Generated by CodeWorm on 2026-02-21 21:10*

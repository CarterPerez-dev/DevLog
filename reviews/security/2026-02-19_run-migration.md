# run_migration

**Type:** Security Review
**Repository:** CertGames-Core
**File:** backend/devtools/migrations/migrate_achievements_v2.py
**Language:** python
**Lines:** 262-396
**Complexity:** 14.0

---

## Source Code

```python
def run_migration(uri, db_name, dry_run = False, batch_size = 100):
    """
    Run the actual migration.
    """
    print("\n" + "=" * 70)
    print(f"RUNNING MIGRATION {'(DRY RUN)' if dry_run else ''}")
    print(f"Database: {db_name}")
    print("=" * 70)

    client = MongoClient(uri)
    try:
        db = client[db_name]
        collection = db[COLLECTION_NAME]

        users_to_migrate = collection.find(
            {
                "achievements": {
                    "$exists": True,
                    "$ne": []
                },
                "$or": [
                    {
                        "achievements_v2": {
                            "$exists": False
                        }
                    },
                    {
                        "achievements_v2": []
                    },
                    {
                        "achievements_v2": {
                            "$size": 0
                        }
                    }
                ]
            }
        )

        count = collection.count_documents(
            {
                "achievements": {
                    "$exists": True,
                    "$ne": []
                },
                "$or": [
                    {
                        "achievements_v2": {
                            "$exists": False
                        }
                    },
                    {
                        "achievements_v2": []
                    },
                    {
                        "achievements_v2": {
                            "$size": 0
                        }
                    }
                ]
            }
        )

        print(f"\nUsers needing migration: {count}")

        if count == 0:
            print("No users need migration!")
            return True

        if not dry_run:
            confirm = input(
                f"\nMigrate {count} users? Type 'YES' to confirm: "
            )
            if confirm != "YES":
                print("Aborted.")
                return False

        migrated = 0
        errors = 0

        for user in users_to_migrate:
            try:
                old_achievements = user.get("achievements", [])
                created_at = user.get("createdAt", datetime.now(UTC))

                if isinstance(created_at, str):
                    try:
                        created_at = datetime.fromisoformat(
                            created_at.replace("Z",
                                               "+00:00")
                        )
                    except ValueError:
                        created_at = datetime.now(UTC)

                new_achievements_v2 = []
                for ach_id in old_achievements:
                    new_achievements_v2.append(
                        {
                            "achievementId": str(ach_id),
                            "unlockedAt": created_at
                        }
                    )

                if not dry_
```

---

## Security Review

### Security Review

#### Vulnerabilities Found:

1. **Info: Hardcoded Secrets or Credentials**
   - **Line:** N/A
   - **Severity:** Info
   - **Recommendation:** Avoid hardcoding secrets like database credentials in the codebase. Use environment variables or a secure vault.

2. **Low: Input Validation Gaps**
   - **Line:** `confirm = input(...)` and `username = user.get("username", "unknown")`
   - **Severity:** Low
   - **Recommendation:** Validate and sanitize all inputs to prevent injection attacks. Consider using a library like `click` for command-line arguments.

3. **Info: Error Handling**
   - **Line:** `print(f"Migration error: {e}")`
   - **Severity:** Info
   - **Recommendation:** Handle errors more securely by logging them instead of printing to the console, especially in production environments.

#### Attack Vectors:

- An attacker could exploit input validation gaps if they manipulate the `confirm` or `username` inputs.
- Hardcoded secrets can be exposed through code review or version control leaks.

#### Recommended Fixes:

1. **Use Environment Variables for Secrets:**
   - Replace hardcoded database credentials with environment variables using `os.environ`.

2. **Enhance Input Validation:**
   - Validate and sanitize all user inputs to prevent injection attacks.
   - Example:
     ```python
     confirm = input("Migrate users? Type 'YES' to confirm: ").strip().upper()
     if confirm != "YES":
         print("Aborted.")
         return False
     ```

3. **Secure Error Handling:**
   - Use logging instead of printing errors.
   - Example:
     ```python
     import logging
     logger = logging.getLogger(__name__)
     try:
         # code...
     except Exception as e:
         logger.error(f"Migration error: {e}")
     ```

#### Overall Security Posture:

The current implementation has some minor security concerns, particularly around hardcoded secrets and basic input validation. By addressing these issues, the overall security posture can be significantly improved.

---

*Generated by CodeWorm on 2026-02-19 20:22*

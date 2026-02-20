# run_migration

**Type:** Performance Analysis
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

## Performance Analysis

### Performance Analysis

#### Time Complexity
The time complexity of `run_migration` is primarily driven by the database operations, which are O(n) for both querying (`find`, `count_documents`) and updating documents (`update_one`). The nested loop over `old_achievements` adds a factor of O(m), where m is the average number of achievements per user. Thus, the overall time complexity is approximately **O(n * m)**.

#### Space Complexity
The space complexity is **O(1)** for variables like `count`, `migrated`, and `errors`. The primary memory usage comes from the database connection (`client`) and temporary lists like `new_achievements_v2`, which are O(m) per user. However, these are managed within the loop.

#### Bottlenecks or Inefficiencies
1. **Redundant Query**: Both `find` and `count_documents` perform similar queries, leading to unnecessary database operations.
2. **Blocking Calls**: The `input` function blocks execution until a response is given, which can be problematic in async contexts.
3. **Error Handling**: Multiple exceptions are caught but not differentiated, making it harder to diagnose specific issues.

#### Optimization Opportunities
1. **Combine Queries**: Use a single query with aggregation pipelines for both counting and fetching documents.
2. **Asynchronous Operations**: Convert blocking calls into asynchronous operations using `asyncio` or `aiomongo`.
3. **Exception Differentiation**: Catch specific exceptions to handle errors more granularly.

#### Resource Usage Concerns
- The database connection is closed in the `finally` block, but ensure that all resources are properly managed.
- Consider caching frequently accessed data or results of expensive operations like `count_documents`.

By addressing these points, you can improve both the performance and maintainability of your migration script.

---

*Generated by CodeWorm on 2026-02-20 17:37*

# run_migration

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

                if not dry_run:
                    collection.update_one(
                        {"_id": user["_id"]},
                        {"$set": {
                            "achievements_v2": new_achievements_v2
                        }}
                    )

                migrated += 1

                if migrated % 50 == 0:
                    print(f"  Migrated {migrated}/{count} users...")

            except Exception as e:
                errors += 1
                username = user.get("username", "unknown")
                print(f"  Error migrating user {username}: {e}")

        print(f"\nMigration {'simulation ' if dry_run else ''}complete!")
        print(f"  Migrated: {migrated}")
        print(f"  Errors: {errors}")

        if dry_run:
            print("\n  This was a DRY RUN - no changes were made.")
            print("  Run without --dry-run to apply changes.")

        return errors == 0

    except Exception as e:
        print(f"Migration error: {e}")
        return False
    finally:
        client.close()
```

---

## Documentation

### Documentation for `run_migration`

**Purpose and Behavior:**
The `run_migration` function migrates user achievements from an old schema (`achievements`) to a new one (`achievements_v2`). It checks users who have existing achievements but lack the new schema, then updates their documents accordingly. The migration can be run in dry-run mode to simulate changes without applying them.

**Key Implementation Details:**
- **Dry Run Mode:** Allows simulating the migration process.
- **Batch Processing:** Updates 50 users at a time for better performance.
- **Error Handling:** Logs errors during the migration and provides feedback on success or failure.
- **Context Manager Usage:** Ensures MongoDB connections are properly closed using `finally`.

**When/Why to Use:**
Use this function when you need to update user data from an old schema to a new one in your database. It’s particularly useful for large datasets where manual updates would be impractical.

**Patterns and Gotchas:**
- **Type Checking:** The code includes type checking for `created_at` timestamps, ensuring they are correctly formatted.
- **Dry Run Confirmation:** A user confirmation step is included to prevent accidental data changes in dry-run mode.
- **Error Logging:** Detailed error messages help with debugging issues during the migration.

---

*Generated by CodeWorm on 2026-01-19 17:13*

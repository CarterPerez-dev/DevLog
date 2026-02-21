# Upload.create

**Type:** Security Review
**Repository:** vuemantics
**File:** backend/models/Upload.py
**Language:** python
**Lines:** 180-257
**Complexity:** 6.0

---

## Source Code

```python
async def create(
        cls,
        user_id: UUID,
        filename: str,
        file_path: str,
        file_type: str,
        file_size: int,
        mime_type: str,
        metadata: dict[str,
                       Any] | None = None,
        upload_id: UUID | None = None,
        batch_id: UUID | None = None,
    ) -> Upload:
        """
        Create a new upload record.

        Args:
            user_id: Owner's user ID
            filename: Original filename
            file_path: Local storage path
            file_type: 'image' or 'video'
            file_size: Size in bytes
            mime_type: MIME type
            metadata: Optional metadata

        Returns:
            Created Upload instance
        """
        await cls.ensure_table_exists()

        if upload_id:
            query = """
                INSERT INTO uploads (
                    id, user_id, batch_id, filename, file_path, file_type,
                    file_size, mime_type, metadata
                )
                VALUES ($1, $2, $3, $4, $5, $6, $7, $8, $9)
                RETURNING *
            """
            record = await database.db.fetchrow(
                query,
                upload_id,
                user_id,
                batch_id,
                filename,
                file_path,
                file_type,
                file_size,
                mime_type,
                json.dumps(metadata) if metadata is not None else None,
            )
        else:
            query = """
                INSERT INTO uploads (
                    user_id, batch_id, filename, file_path, file_type,
                    file_size, mime_type, metadata
                )
                VALUES ($1, $2, $3, $4, $5, $6, $7, $8)
                RETURNING *
            """
            record = await database.db.fetchrow(
                query,
                user_id,
                batch_id,
                filename,
                file_path,
                file_type,
                file_size,
                mime_type,
                json.dumps(metadata) if metadata is not None else None,
            )

        if record is None:
            raise ValueError("Failed to create upload")
        upload = cls.from_record(record)
        if upload is None:
            raise ValueError("Failed to create upload from record")
        return upload
```

---

## Security Review

### Security Review for `Upload.create`

**Vulnerabilities Found:**

1. **SQL Injection (Critical):**
   - Line 15-17 and 20-22: The `filename`, `file_path`, `file_type`, `metadata` variables are directly inserted into SQL queries without sanitization, making the code vulnerable to SQL injection.

2. **Hardcoded Secrets or Credentials (Info):**
   - No hardcoded secrets found in this snippet.

3. **Input Validation Gaps (Medium):**
   - The function does not validate input types and values for `filename`, `file_path`, etc., which could lead to unexpected behavior or errors.

4. **Error Handling that Leaks Information (Low):**
   - Line 25-27: Raising a `ValueError` with "Failed to create upload" can leak information about the database operation failing.

**Attack Vectors:**

- An attacker could manipulate input values to execute arbitrary SQL commands or cause database errors.
- Improper error handling might reveal sensitive information about the database structure and operations.

**Recommended Fixes:**

1. **Sanitize Inputs (Critical):**
   - Use parameterized queries with placeholders for all inputs, e.g., `query = "INSERT INTO uploads (...)"` and pass parameters separately to avoid SQL injection.
   ```python
   query = """
       INSERT INTO uploads (
           id, user_id, batch_id, filename, file_path, file_type,
           file_size, mime_type, metadata
       )
       VALUES ($1, $2, $3, $4, $5, $6, $7, $8, $9)
       RETURNING *
   """
   record = await database.db.fetchrow(
       query,
       upload_id or None,
       user_id,
       batch_id or None,
       filename,
       file_path,
       file_type,
       file_size,
       mime_type,
       json.dumps(metadata) if metadata is not None else None,
   )
   ```

2. **Improve Error Handling (Low):**
   - Catch specific exceptions and provide generic error messages.
   ```python
   try:
       record = await database.db.fetchrow(...)
   except Exception as e:
       raise ValueError("Failed to create upload") from e
   ```

3. **Validate Inputs (Medium):**
   - Add input validation for `filename`, `file_path`, etc., to ensure they meet expected formats.

**Overall Security Posture:**

The current code has critical vulnerabilities that need immediate attention, particularly SQL injection and lack of proper error handling. Addressing these issues will significantly improve the security posture of this function.

---

*Generated by CodeWorm on 2026-02-21 13:25*

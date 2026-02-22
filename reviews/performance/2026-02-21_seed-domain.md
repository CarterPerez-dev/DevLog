# seed_domain

**Type:** Performance Analysis
**Repository:** my-portfolio
**File:** v1/data/tools/seed.py
**Language:** python
**Lines:** 204-278
**Complexity:** 22.0

---

## Source Code

```python
async def seed_domain(
    session: AsyncSession,
    domain: str,
    dry_run: bool = False,
    clear: bool = False,
) -> tuple[int, int]:
    config = DOMAIN_CONFIG[domain]
    model = config["model"]
    files = scan_seed_files(domain)

    if not files:
        print(f"  {color('dim', 'No files found')}")
        return 0, 0

    if clear and not dry_run:
        await session.execute(delete(model))
        print(f"  {color('yellow', 'Cleared existing records')}")

    inserted = 0
    errors = 0

    for file_path, lang_code, is_array_file in files:
        try:
            with open(file_path) as f:
                raw_data = json.load(f)

            records = raw_data if is_array_file else [raw_data]

            for idx, data in enumerate(records):
                try:
                    if "language" not in data or data["language"] is None:
                        data["language"] = lang_code

                    transformed = transform_data(data, config)

                    if dry_run:
                        label = f"{file_path.name}[{idx}]" if is_array_file else file_path.name
                        print(f"  {color('blue', '○')} {label} (dry-run)")
                    else:
                        upsert_keys = config.get("upsert_keys", [])
                        if upsert_keys:
                            stmt = insert(model).values(**transformed)
                            update_cols = {
                                k: v for k, v in transformed.items()
                                if k not in upsert_keys and k != "id"
                            }
                            stmt = stmt.on_conflict_do_update(
                                index_elements=upsert_keys,
                                set_=update_cols,
                            )
                            await session.execute(stmt)
                        else:
                            record = model(**transformed)
                            session.add(record)

                    inserted += 1

                except TypeError as e:
                    label = f"{file_path.name}[{idx}]" if is_array_file else file_path.name
                    print(f"  {color('red', '✗')} {label}: Field error - {e}")
                    errors += 1

            if not dry_run and is_array_file:
                print(f"  {color('green', '✓')} {file_path.name} ({len(records)} records)")
            elif not dry_run:
                print(f"  {color('green', '✓')} {file_path.name}")

        except json.JSONDecodeError as e:
            print(f"  {color('red', '✗')} {file_path.name}: Invalid JSON - {e}")
            errors += 1
        except Exception as e:
            print(f"  {color('red', '✗')} {file_path.name}: {e}")
            errors += 1

    return inserted, errors
```

---

## Performance Analysis

### Performance Analysis

#### Time Complexity
The function has a time complexity of \(O(n \times m)\), where \(n\) is the number of files, and \(m\) is the average number of records per file. The nested loops over `records` contribute to this.

#### Space Complexity
Space complexity is \(O(1)\) for variables like `inserted`, `errors`, but \(O(n \times m)\) for storing `raw_data` and `records`. This can be memory-intensive if dealing with large files or many records.

#### Bottlenecks and Inefficiencies
1. **Redundant File Openings**: Opening the file inside the loop is inefficient, especially if there are many small files.
2. **Multiple Exceptions Handling**: The nested try-except blocks can lead to performance overhead due to repeated exception handling.
3. **Blocking Operations in Async Context**: Using `open(file_path)` and `json.load(f)` within an async function can block the event loop.

#### Optimization Opportunities
1. **File Reading Outside Loop**: Read files once outside the loop and process them asynchronously.
2. **Async File Handling**: Use `aiofiles` to read files asynchronously.
3. **Error Handling Simplification**: Combine similar exception handling logic to reduce overhead.

```python
async def seed_domain(
    session: AsyncSession,
    domain: str,
    dry_run: bool = False,
    clear: bool = False,
) -> tuple[int, int]:
    config = DOMAIN_CONFIG[domain]
    model = config["model"]
    files = scan_seed_files(domain)

    if not files:
        print(f"  {color('dim', 'No files found')}")
        return 0, 0

    if clear and not dry_run:
        await session.execute(delete(model))
        print(f"  {color('yellow', 'Cleared existing records')}")

    inserted = 0
    errors = 0

    async with aiofiles.open(file_path) as f:
        raw_data = json.loads(await f.read())

    records = [raw_data] if not is_array_file else raw_data

    for idx, data in enumerate(records):
        try:
            # ... (rest of the code remains mostly the same)
```

#### Resource Usage Concerns
- Ensure `aiofiles` and other async libraries are properly installed.
- Use context managers (`async with`) to manage file handles efficiently.

By applying these optimizations, you can significantly improve the performance and efficiency of your function.

---

*Generated by CodeWorm on 2026-02-21 20:00*

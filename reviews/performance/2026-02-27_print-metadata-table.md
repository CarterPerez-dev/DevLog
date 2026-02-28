# print_metadata_table

**Type:** Performance Analysis
**Repository:** Cybersecurity-Projects
**File:** PROJECTS/beginner/metadata-scrubber-tool/src/utils/display.py
**Language:** python
**Lines:** 21-114
**Complexity:** 11.0

---

## Source Code

```python
def print_metadata_table(metadata: dict[str, Any]):
    """
    Display metadata in a formatted table organized by logical groups.

    Organizes metadata into categories (Device Info, Exposure Settings,
    Image Data, Dates) and displays them in a Rich panel with color coding.

    Args:
        metadata: Dict of metadata key-value pairs to display.
    """

    # Define the groups using simple lists of keys
    groups = {
        "📄 Document Info": [
            "Author",
            "author",
            "/Author",
            "/Creator",
        ],
        "📸 Device Info": ["Make",
                          "Model",
                          "Software",
                          "ExifVersion"],
        "⚙️ Exposure Settings": [
            "ExposureTime",
            "FNumber",
            "ISOSpeedRatings",
            "ShutterSpeedValue",
            "ApertureValue",
            "Flash",
            "FocalLength",
        ],
        "🖼️ Image Data": [
            "ImageWidth",
            "ImageLength",
            "PixelXDimension",
            "PixelYDimension",
            "Orientation",
            "ResolutionUnit",
        ],
        "📅 Dates": [
            "DateTime",
            "DateTimeOriginal",
            "DateTimeDigitized",
            "OffsetTime",
            "created",
            "modified",
            "/CreationDate",
            "/ModDate",
        ],
    }

    # Create the main table
    table = Table(box = box.ROUNDED, show_header = True, header_style = "bold magenta")
    table.add_column("Property", style = "cyan")
    table.add_column("Value", style = "green")

    # Track which keys we have displayed to handle the "leftovers"
    displayed_keys = set()

    # Loop through the defined groups to create sections
    for section_name, keys in groups.items():
        # Check if we have any data for this section
        section_data = {k: metadata[k] for k in keys if k in metadata}

        if section_data:
            # Add a section row (acts as a sub-header)
            table.add_row(Text(section_name, style = "bold yellow"), "")

            for key, val in section_data.items():
                table.add_row(f"  {key}", clean_value(val))
                displayed_keys.add(key)

            # Add a blank row for spacing
            table.add_section()

    # Handle "Other" (Any keys that isn't in the groups)
    leftovers = {
        k: v
        for k, v in metadata.items()
        if k not in displayed_keys and k != "JPEGInterchangeFormat"
    }  # skip binary blobs
    if leftovers:
        table.add_row(Text("📝 Other", style = "bold yellow"), "")
        for key, val in leftovers.items():
            table.add_row(f"  {key}", clean_value(val))

    # Print nicely inside a panel
    console.print(
        Panel(table,
              title = "Metadata Report",
              border_style = "blue",
              expand = False)
    )
```

---

## Performance Analysis

### Performance Analysis

#### Time Complexity (Big O Notation)
The time complexity of `print_metadata_table` is \(O(n \cdot m)\), where \(n\) is the number of metadata keys and \(m\) is the average length of key lists in groups. The primary bottleneck is the nested dictionary comprehension used to filter keys, which iterates over all keys multiple times.

#### Space Complexity
The space complexity is \(O(n + k)\), where \(k\) is the total number of unique keys across all group definitions and leftovers. The main memory usage comes from storing intermediate dictionaries during key filtering and table construction.

#### Bottlenecks or Inefficiencies
1. **Redundant Iterations**: The nested dictionary comprehension iterates over `metadata` multiple times, leading to unnecessary overhead.
2. **Unnecessary Operations**: The function creates a new dictionary for each group and leftover keys, which can be optimized by directly adding keys to the table.

#### Optimization Opportunities
1. **Reduce Redundant Iterations**: Use a single pass through `metadata` to populate the table.
2. **Optimize Dictionary Comprehensions**: Combine operations into fewer steps to reduce overhead.

```python
def print_metadata_table(metadata: dict[str, Any]):
    groups = {
        "📄 Document Info": [
            "Author",
            "author",
            "/Author",
            "/Creator",
        ],
        # ... other group definitions ...
    }

    table = Table(box=box.ROUNDED, show_header=True, header_style="bold magenta")
    table.add_column("Property", style="cyan")
    table.add_column("Value", style="green")

    displayed_keys = set()
    for section_name, keys in groups.items():
        section_data = {k: metadata[k] for k in keys if k in metadata}
        if section_data:
            table.add_row(Text(section_name, style="bold yellow"), "")
            for key, val in section_data.items():
                table.add_row(f"  {key}", clean_value(val))
                displayed_keys.add(key)

    leftovers = {
        k: v
        for k, v in metadata.items()
        if k not in displayed_keys and k != "JPEGInterchangeFormat"
    }
    if leftovers:
        table.add_row(Text("📝 Other", style="bold yellow"), "")
        for key, val in leftovers.items():
            table.add_row(f"  {key}", clean_value(val))

    console.print(
        Panel(table,
              title="Metadata Report",
              border_style="blue",
              expand=False)
    )
```

#### Resource Usage Concerns
- **Memory Leaks**: Ensure all resources are properly managed, especially if this function is called repeatedly.
- **File Handles**: No file handling issues in the provided code, but ensure no unclosed files elsewhere.

By optimizing the key filtering and reducing redundant operations, you can improve both time and space efficiency.

---

*Generated by CodeWorm on 2026-02-27 20:03*

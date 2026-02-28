# print_metadata_table

**Type:** Security Review
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

## Security Review

### Security Review for `print_metadata_table` Function

#### Vulnerabilities Found:

1. **Info**: The function does not have any apparent injection, authentication, or authorization issues.
2. **Info**: There are no hardcoded secrets or credentials present.
3. **Info**: No race conditions or TOCTOU bugs were found in the provided code snippet.
4. **Info**: Input validation is handled by dictionary keys and list membership checks, which are appropriate for this context.

#### Attack Vectors:

- While there are no obvious security issues, an attacker could potentially manipulate metadata to include unexpected values that might be displayed differently or cause unintended behavior if used elsewhere in the application.

#### Recommended Fixes:

1. **Documentation**: Ensure all input keys and expected values are documented.
2. **Error Handling**: Implement basic error handling for missing keys to avoid potential `KeyError` exceptions, e.g., using `.get()` method instead of direct access.
3. **Code Comments**: Add comments explaining the purpose of each section in the metadata dictionary.

#### Overall Security Posture:

The current implementation is relatively secure given its context. However, maintaining clear documentation and robust error handling can further enhance security posture. The function focuses on displaying metadata in a structured format without any direct interaction with external systems or user inputs that could introduce vulnerabilities.

```python
def clean_value(value):
    """Clean and prepare the value for display."""
    if isinstance(value, bytes):  # Handle binary data
        return "Binary Data"
    elif isinstance(value, datetime.datetime):
        return value.isoformat()
    else:
        return str(value)

# Example usage of get() to handle missing keys
section_data = {k: metadata.get(k) for k in keys if k in metadata}
```

By implementing these minor improvements, the function can be made more robust and secure.

---

*Generated by CodeWorm on 2026-02-27 19:34*

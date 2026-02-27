# PngProcessor.get_metadata

**Type:** Performance Analysis
**Repository:** Cybersecurity-Projects
**File:** PROJECTS/beginner/metadata-scrubber-tool/src/core/png_metadata.py
**Language:** python
**Lines:** 51-113
**Complexity:** 13.0

---

## Source Code

```python
def get_metadata(self, img: Image) -> dict[str, Any]:
        """
        Extract metadata from a PNG image.

        Extracts both EXIF data (if present) and PNG textual chunks (PngInfo).

        Args:
            img: PIL Image object.

        Returns:
            Dict with 'data' (metadata dict), 'tags_to_delete' (EXIF tag IDs),
            and 'text_keys' (PngInfo keys to remove).

        Raises:
            MetadataNotFoundError: If no metadata is found in the image.
        """
        img.load()
        found_metadata = False

        # Extract EXIF data (if present)
        exif = img.getexif()
        if exif:
            found_metadata = True
            # Main IFD
            for tag, value in exif.items():
                tag_name = ExifTags.TAGS.get(tag, f"Tag_{tag}")
                self.tags_to_delete.append(tag)
                self.data[f"EXIF:{tag_name}"] = value

            # GPS IFD
            gps_ifd = exif.get_ifd(ExifTags.IFD.GPSInfo)
            for tag, value in gps_ifd.items():
                tag_name = ExifTags.GPSTAGS.get(tag, f"GPSTag_{tag}")
                self.tags_to_delete.append(tag)
                self.data[f"GPS:{tag_name}"] = value

        # Extract PNG textual metadata (PngInfo chunks)
        if hasattr(img, "info") and img.info:
            for key, value in img.info.items():
                # Skip binary/internal data
                if key in ("icc_profile", "exif", "transparency", "gamma"):
                    continue

                if isinstance(value, str | bytes):
                    found_metadata = True
                    if isinstance(value, bytes):
                        try:
                            value = value.decode("utf-8", errors = "replace")
                        except Exception:
                            value = str(value)

                    self.data[f"PNG:{key}"] = value
                    if isinstance(key, str):  # Only add string keys
                        self.text_keys_to_delete.append(key)

        if not found_metadata:
            raise MetadataNotFoundError("No metadata found in the PNG image.")

        return {
            "data": self.data,
            "tags_to_delete": self.tags_to_delete,
            "text_keys": self.text_keys_to_delete,
        }
```

---

## Performance Analysis

### Performance Analysis

**Time Complexity:** The function has a time complexity of \(O(n + m)\), where \(n\) is the number of EXIF tags, and \(m\) is the number of PNG textual chunks. This is efficient for typical image sizes.

**Space Complexity:** The space complexity is \(O(k + l)\), where \(k\) is the size of the metadata dictionary, and \(l\) is the length of the `tags_to_delete` and `text_keys_to_delete` lists. Memory usage can be optimized by avoiding unnecessary string conversions.

**Bottlenecks or Inefficiencies:**
1. **Redundant Operations:** The function decodes binary values to strings even when they are not necessary, which is inefficient.
2. **Unnecessary Iterations:** The `if isinstance(value, str | bytes):` check and subsequent decoding can be avoided by handling different types separately.

**Optimization Opportunities:**
1. **Conditional Decoding:** Only decode binary values if they need to be treated as strings.
2. **Early Return:** If no metadata is found early in the function, return immediately instead of processing all EXIF and PNG chunks.

**Resource Usage Concerns:**
- The `img.load()` call may not be necessary if the image has already been loaded elsewhere.
- Ensure that `self.data`, `self.tags_to_delete`, and `self.text_keys_to_delete` are properly initialized to avoid potential issues.

### Optimized Code Example

```python
def get_metadata(self, img: Image) -> dict[str, Any]:
    """
    Extract metadata from a PNG image.
    """
    found_metadata = False

    # Extract EXIF data (if present)
    exif = img.getexif()
    if exif:
        found_metadata = True
        for tag, value in exif.items():
            tag_name = ExifTags.TAGS.get(tag, f"Tag_{tag}")
            self.tags_to_delete.append(tag)
            self.data[f"EXIF:{tag_name}"] = value

            # GPS IFD
            gps_ifd = exif.get_ifd(ExifTags.IFD.GPSInfo)
            for tag, value in gps_ifd.items():
                tag_name = ExifTags.GPSTAGS.get(tag, f"GPSTag_{tag}")
                self.tags_to_delete.append(tag)
                self.data[f"GPS:{tag_name}"] = value

    # Extract PNG textual metadata (PngInfo chunks)
    if hasattr(img, "info") and img.info:
        for key, value in img.info.items():
            if key not in ("icc_profile", "exif", "transparency", "gamma"):
                found_metadata = True
                if isinstance(value, bytes):
                    try:
                        value = value.decode("utf-8", errors="replace")
                    except Exception:
                        pass  # Ignore decoding errors

                self.data[f"PNG:{key}"] = value
                if isinstance(key, str):  # Only add string keys
                    self.text_keys_to_delete.append(key)

    if not found_metadata:
        raise MetadataNotFoundError("No metadata found in the PNG image.")

    return {
        "data": self.data,
        "tags_to_delete": self.tags_to_delete,
        "text_keys": self.text_keys_to_delete,
    }
```

This optimized code reduces unnecessary operations and improves readability.

---

*Generated by CodeWorm on 2026-02-27 18:30*

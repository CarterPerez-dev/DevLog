# get_certification_tags

**Repository:** CertGames-Core
**File:** backend/devtools/migrations/migrate_resources.py
**Language:** python
**Lines:** 30-118
**Complexity:** 34.0

---

## Source Code

```python
def get_certification_tags(name, url, description = ""):
    """
    Determine certification tags based on resource name, URL, and description
    """
    name_lower = name.lower()
    url_lower = url.lower()
    desc_lower = description.lower()
    combined = f"{name_lower} {url_lower} {desc_lower}"

    tags = []

    # A+ patterns
    if any(pattern in combined for pattern in
           ['a+', 'a plus', '220-1101', '220-1102', 'core 1', 'core 2']):
        if not any(exclude in combined for exclude in
                   ['network+', 'security+', 'casp', 'pentest+']):
            tags.append(Certification.A_PLUS.value)

    # Network+ patterns
    if any(pattern in combined for pattern in
           ['network+', 'n10-008', 'n10-009', 'networking', 'subnet']):
        if not any(exclude in combined
                   for exclude in ['a+', 'security+', 'casp', 'pentest+']):
            tags.append(Certification.NETWORK_PLUS.value)

    # Security+ patterns
    if any(pattern in combined
           for pattern in ['security+', 'sy0-701', 'sec+']):
        tags.append(Certification.SECURITY_PLUS.value)

    # CySA+ patterns
    if any(pattern in combined
           for pattern in ['cysa+', 'cs0-003', 'cybersecurity analyst']):
        tags.append(Certification.CYSA_PLUS.value)

    # CASP+/SecurityX patterns
    if any(pattern in combined
           for pattern in ['casp', 'cas-004', 'cas-005', 'securityx']):
        tags.append(Certification.CASP_PLUS.value)

    # PenTest+ patterns
    if any(pattern in combined
           for pattern in ['pentest+', 'pt0-002', 'pt0-003', 'penetration',
                           'ethical hack']):
        tags.append(Certification.PENTEST_PLUS.value)

    # Cloud+ patterns
    if any(pattern in combined for pattern in
           ['cloud+', 'cv0-003', 'cloud essentials', 'cl0-002']):
        tags.append(Certification.CLOUD_PLUS.value)

    # Linux+ patterns
    if any(pattern in combined
           for pattern in ['linux+', 'xk0-005', 'kali linux', 'ubuntu']):
        tags.append(Certification.LINUX_PLUS.value)

    # Data+ patterns
    if any(pattern in combined for pattern in ['data+', 'da0-001']):
        tags.append(Certification.DATA_PLUS.value)

    # Server+ patterns
    if any(pattern in combined for pattern in ['server+', 'sk0-005']):
        tags.append(Certification.SERVER_PLUS.value)

    # Project+ patterns
    if any(pattern in combined for pattern in ['project+', 'pk0-005']):
        tags.append(Certification.PROJECT_PLUS.value)

    # ITF/TECH+ patterns
    if any(pattern in combined for pattern in
           ['itf', 'tech+', 'fc0-u61', 'fc0-u71', 'it fundamentals']):
        tags.append(Certification.TECH_PLUS.value)

    # AWS Cloud patterns
    if any(pattern in combined
           for pattern in ['aws', 'amazon web services', 'clf-c02',
                           'cloud practitioner']):
        tags.append(Certification.AWS_CLOUD.value)

    # CISSP patterns
    if any(pattern in combined for pattern in
           ['cissp', 'isc2', 'certified information systems']):
        tags.append(Certification.CISSP.value)

    # If no specific certification found, add General
    if not tags:
        tags.append(Certification.GENERAL.value)

    return tags
```

---

## Documentation

### Documentation for `get_certification_tags` Function

**Purpose and Behavior:**
The function `get_certification_tags` determines certification tags based on the resource name, URL, and description by searching for specific patterns in a combined string of these inputs. It returns a list of certifications that match the input data.

**Key Implementation Details:**
- The function converts all inputs to lowercase for case-insensitive matching.
- It uses list comprehensions with `any()` to check if any pattern is present in the combined string.
- Multiple conditional checks are used to ensure specific certifications are not excluded based on other patterns found.
- A default tag, `Certification.GENERAL`, is added if no specific certification matches.

**When/Why to Use:**
Use this function when you need to categorize resources or content by relevant certifications based on their names and descriptions. It's particularly useful for educational platforms or certification tracking systems where automatic tagging can enhance user experience and resource organization.

**Patterns and Gotchas:**
- The function uses a series of `if` statements with list comprehensions, which can be complex to maintain but are efficient for pattern matching.
- Ensure that the patterns and exclusion terms are up-to-date and comprehensive to avoid false negatives or positives.
- Consider refactoring into smaller functions if the complexity grows, as it currently has 34 cyclomatic complexity.

---

*Generated by CodeWorm on 2026-01-19 12:58*

# capabilities

**Type:** File Overview
**Repository:** docksec
**File:** internal/proc/capabilities.go
**Language:** go
**Lines:** 1-284
**Complexity:** 0.0

---

## Source Code

```go
/*
CarterPerez-dev | 2025
capabilities.go
*/

package proc

import (
	"fmt"

	"github.com/CarterPerez-dev/docksec/internal/finding"
	"github.com/CarterPerez-dev/docksec/internal/rules"
)

type CapabilitySet struct {
	Effective   uint64
	Permitted   uint64
	Inheritable uint64
	Bounding    uint64
	Ambient     uint64
}

var capabilityBits = map[int]string{
	0:  "CAP_CHOWN",
	1:  "CAP_DAC_OVERRIDE",
	2:  "CAP_DAC_READ_SEARCH",
	3:  "CAP_FOWNER",
	4:  "CAP_FSETID",
	5:  "CAP_KILL",
	6:  "CAP_SETGID",
	7:  "CAP_SETUID",
	8:  "CAP_SETPCAP",
	9:  "CAP_LINUX_IMMUTABLE",
	10: "CAP_NET_BIND_SERVICE",
	11: "CAP_NET_BROADCAST",
	12: "CAP_NET_ADMIN",
	13: "CAP_NET_RAW",
	14: "CAP_IPC_LOCK",
	15: "CAP_IPC_OWNER",
	16: "CAP_SYS_MODULE",
	17: "CAP_SYS_RAWIO",
	18: "CAP_SYS_CHROOT",
	19: "CAP_SYS_PTRACE",
	20: "CAP_SYS_PACCT",
	21: "CAP_SYS_ADMIN",
	22: "CAP_SYS_BOOT",
	23: "CAP_SYS_NICE",
	24: "CAP_SYS_RESOURCE",
	25: "CAP_SYS_TIME",
	26: "CAP_SYS_TTY_CONFIG",
	27: "CAP_MKNOD",
	28: "CAP_LEASE",
	29: "CAP_AUDIT_WRITE",
	30: "CAP_AUDIT_CONTROL",
	31: "CAP_SETFCAP",
	32: "CAP_MAC_OVERRIDE",
	33: "CAP_MAC_ADMIN",
	34: "CAP_SYSLOG",
	35: "CAP_WAKE_ALARM",
	36: "CAP_BLOCK_SUSPEND",
	37: "CAP_AUDIT_READ",
	38: "CAP_PERFMON",
	39: "CAP_BPF",
	40: "CAP_CHECKPOINT_RESTORE",
}

var capabilityNames = func() map[string]int {
	m := make(map[string]int, len(capabilityBits))
	for bit, name := range capabilityBits {
		m[name] = bit
	}
	return m
}()

func (c *CapabilitySet) HasCapability(name string) bool {
	bit, ok := capabilityNames[name]
	if !ok {
		return false
	}
	return (c.Effective & (1 << bit)) != 0
}

func (c *CapabilitySet) HasPermitted(name string) bool {
	bit, ok := capabilityNames[name]
	if !ok {
		return false
	}
	return (c.Permitted & (1 << bit)) != 0
}

func (c *CapabilitySet) HasBounding(name string) bool {
	bit, ok := capabilityNames[name]
	if !ok {
		return false
	}
	return (c.Bounding & (1 << bit)) != 0
}

func (c *CapabilitySet) HasAmbient(name string) bool {
	bit, ok := capabilityNames[name]
	if !ok {
		return false
	}
	return (c.Ambient & (1 << bit)) != 0
}

func (c *CapabilitySet) ListEffective() []string {
	return c.listCaps(c.Effective)
}

func (c *CapabilitySet) ListPermitted() []string {
	return c.listCaps(c.Permitted)
}

func (c *CapabilitySet) ListBounding() []string {
	return c.listCaps(c.Bounding)
}

func (c *CapabilitySet) ListAmbient() []string {
	return c.listCaps(c.Ambient)
}

func (c *CapabilitySet) ListInheritable() []string {
	return c.listCaps(c.Inheritable)
}

func (c *CapabilitySet) listCaps(mask uint64) []string {
	var caps []string
	for bit, name := range capabilityBits {
		if (mask & (1 << bit)) != 0 {
			caps = append(caps, name)
		}
	}
	return caps
}

func (c *CapabilitySet) IsFullyPrivileged() bool {
	return c.Effective == 0x1ffffffffff || c.Effective == 0xffffffffffffffff
}

func (c *CapabilitySet) HasDangerousCapabilities() bool {
	for _, cap := range c.ListEffective() {
		if rules.IsDangerousCapability(cap) {
			return true
		}
	
```

---

## File Overview

# capabilities.go

## Purpose and Responsibility
This Go source file defines a `CapabilitySet` struct to manage Linux capability sets within the `proc` package of the `docksec` project. It provides methods for checking, listing, and counting specific capabilities, as well as determining if certain capabilities are dangerous or critical.

## Key Exports and Public Interface
- **Structs:**
  - `CapabilitySet`: Represents a set of Linux capabilities.
  
- **Functions:**
  - `HasCapability`, `HasPermitted`, `HasBounding`, `HasAmbient`: Check if a specific capability is enabled in the respective sets.
  - `ListEffective`, `ListPermitted`, `ListBounding`, `ListAmbient`, `ListInheritable`: List all capabilities in the specified set.
  - `IsFullyPrivileged`: Determine if the effective capabilities are fully privileged.
  - `HasDangerousCapabilities`, `HasCriticalCapabilities`: Check for dangerous or critical capabilities.
  - `GetDangerousCapabilities`, `GetCriticalCapabilities`: Retrieve a list of dangerous or critical capabilities.
  - `GetCapabilitiesBySeverity`: Filter capabilities by severity level.
  - `EffectiveCount`, `PermittedCount`, `BoundingCount`: Count the number of enabled capabilities in each set.
  - `ParseCapabilityMask`: Parse a hexadecimal string into a capability mask.
  - `CapabilityNameToBit`, `CapabilityBitToName`: Convert between capability names and bit positions.
  - `AllCapabilityNames`: List all capability names.

## How It Fits Into the Project
This file is crucial for managing and analyzing container capabilities in the `docksec` project. It integrates with other components such as `finding` and `rules` to assess security risks based on the container's capability set.

## Notable Design Decisions
- **Error Handling:** Functions like `ParseCapabilityMask` return errors, ensuring robustness.
- **Bitwise Operations:** Efficient use of bitwise operations for checking and counting capabilities.
- **Maps and Interfaces:** Utilizes maps for efficient name-to-bit and bit-to-name conversions, enhancing readability and performance.

---

*Generated by CodeWorm on 2026-02-20 19:56*

# container

**Type:** File Overview
**Repository:** docksec
**File:** internal/analyzer/container.go
**Language:** go
**Lines:** 1-369
**Complexity:** 0.0

---

## Source Code

```go
/*
AngelaMos | 2025
container.go
*/

package analyzer

import (
	"context"
	"strings"

	"github.com/CarterPerez-dev/docksec/internal/benchmark"
	"github.com/CarterPerez-dev/docksec/internal/docker"
	"github.com/CarterPerez-dev/docksec/internal/finding"
	"github.com/CarterPerez-dev/docksec/internal/rules"
	"github.com/docker/docker/api/types"
)

// ContainerAnalyzer inspects running containers for security issues including
// privileged mode, dangerous capabilities, sensitive mounts, and missing
// security profiles (seccomp, AppArmor).
type ContainerAnalyzer struct {
	client *docker.Client
}

// NewContainerAnalyzer creates a new analyzer using the provided Docker client.
func NewContainerAnalyzer(client *docker.Client) *ContainerAnalyzer {
	return &ContainerAnalyzer{client: client}
}

// Name returns "container" as the analyzer identifier.
func (a *ContainerAnalyzer) Name() string {
	return "container"
}

// Analyze lists all containers and inspects each for security misconfigurations
// per CIS Docker Benchmark Section 5 controls.
func (a *ContainerAnalyzer) Analyze(
	ctx context.Context,
) (finding.Collection, error) {
	containers, err := a.client.ListContainers(ctx, true)
	if err != nil {
		return nil, err
	}

	var findings finding.Collection
	for _, c := range containers {
		info, err := a.client.InspectContainer(ctx, c.ID)
		if err != nil {
			continue
		}
		findings = append(findings, a.analyzeContainer(info)...)
	}

	return findings, nil
}

func (a *ContainerAnalyzer) analyzeContainer(
	info types.ContainerJSON,
) finding.Collection {
	var findings finding.Collection
	target := finding.Target{
		Type: finding.TargetContainer,
		Name: strings.TrimPrefix(info.Name, "/"),
		ID:   info.ID,
	}

	if info.HostConfig == nil {
		return findings
	}

	findings = append(findings, a.checkPrivileged(target, info)...)
	findings = append(findings, a.checkCapabilities(target, info)...)
	findings = append(findings, a.checkMounts(target, info)...)
	findings = append(findings, a.checkNamespaces(target, info)...)
	findings = append(findings, a.checkSecurityOptions(target, info)...)
	findings = append(findings, a.checkResourceLimits(target, info)...)
	findings = append(findings, a.checkReadonlyRootfs(target, info)...)

	return findings
}

func (a *ContainerAnalyzer) checkPrivileged(
	target finding.Target,
	info types.ContainerJSON,
) finding.Collection {
	var findings finding.Collection

	if info.HostConfig.Privileged {
		control, _ := benchmark.Get("5.4")
		f := finding.New("CIS-5.4", control.Title, finding.SeverityCritical, target).
			WithDescription(control.Description).
			WithCategory(string(CategoryContainerRuntime)).
			WithRemediation(control.Remediation).
			WithReferences(control.References...).
			WithCISControl(control.ToCISControl())
		findings = append(findings, f)
	}

	return findings
}

func (a *ContainerAnalyzer) checkCapabilities(
	target finding.Target,
	info types.ContainerJSON,
) finding.Collection {
	var findings finding.Collection

	fo
```

---

## File Overview

### container.go

**Purpose and Responsibility:**
This Go source file implements the `ContainerAnalyzer` struct, which inspects running Docker containers for security misconfigurations according to the CIS Docker Benchmark Section 5 controls. It leverages the Docker API client to list and inspect containers, identifying potential vulnerabilities such as privileged mode usage, dangerous capabilities, sensitive mounts, and missing security profiles.

**Key Exports and Public Interface:**
- `ContainerAnalyzer`: A struct for analyzing container security.
- `NewContainerAnalyzer`: Factory function to create a new analyzer instance using a Docker client.
- `Analyze`: Method to perform the analysis and return findings as a collection of `finding` objects.
- `Name`: Returns the identifier "container" for this analyzer.

**How It Fits in the Project:**
This file is part of the `analyzer` package, which is responsible for detecting security issues within Docker containers. The `ContainerAnalyzer` integrates with other components like `benchmark`, `docker`, and `finding` to ensure comprehensive coverage of security controls. It fits into the larger project by providing a concrete implementation of container security analysis that can be integrated into the overall security assessment framework.

**Notable Design Decisions:**
- **Error Handling:** The code uses standard Go error handling practices, returning errors from critical operations like listing and inspecting containers.
- **Modular Analysis:** The `analyzeContainer` method is broken down into smaller functions (`checkPrivileged`, `checkCapabilities`, etc.) to handle specific security checks, promoting readability and maintainability.
- **Dependency Injection:** The analyzer uses a Docker client injected via the constructor, allowing for flexibility in how it interacts with the Docker API.
- **CIS Benchmark Compliance:** The analysis is aligned with the CIS Docker Benchmark Section 5 controls, ensuring that findings are relevant and actionable according to industry standards.

---

*Generated by CodeWorm on 2026-02-20 18:52*

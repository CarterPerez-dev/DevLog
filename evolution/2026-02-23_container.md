# container

**Type:** Code Evolution
**Repository:** docksec
**File:** internal/analyzer/container.go
**Language:** go
**Lines:** 1-1
**Complexity:** 0.0

---

## Source Code

```go
Commit: 5a7c48c4
Message: Initial release
Author: CarterPerez-dev
File: internal/analyzer/container.go
Change type: new file

Diff:
@@ -0,0 +1,368 @@
+/*
+AngelaMos | 2025
+container.go
+*/
+
+package analyzer
+
+import (
+	"context"
+	"strings"
+
+	"github.com/CarterPerez-dev/docksec/internal/benchmark"
+	"github.com/CarterPerez-dev/docksec/internal/docker"
+	"github.com/CarterPerez-dev/docksec/internal/finding"
+	"github.com/CarterPerez-dev/docksec/internal/rules"
+	"github.com/docker/docker/api/types"
+)
+
+// ContainerAnalyzer inspects running containers for security issues including
+// privileged mode, dangerous capabilities, sensitive mounts, and missing
+// security profiles (seccomp, AppArmor).
+type ContainerAnalyzer struct {
+	client *docker.Client
+}
+
+// NewContainerAnalyzer creates a new analyzer using the provided Docker client.
+func NewContainerAnalyzer(client *docker.Client) *ContainerAnalyzer {
+	return &ContainerAnalyzer{client: client}
+}
+
+// Name returns "container" as the analyzer identifier.
+func (a *ContainerAnalyzer) Name() string {
+	return "container"
+}
+
+// Analyze lists all containers and inspects each for security misconfigurations
+// per CIS Docker Benchmark Section 5 controls.
+func (a *ContainerAnalyzer) Analyze(
+	ctx context.Context,
+) (finding.Collection, error) {
+	containers, err := a.client.ListContainers(ctx, true)
+	if err != nil {
+		return nil, err
+	}
+
+	var findings finding.Collection
+	for _, c := range containers {
+		info, err := a.client.InspectContainer(ctx, c.ID)
+		if err != nil {
+			continue
+		}
+		findings = append(findings, a.analyzeContainer(info)...)
+	}
+
+	return findings, nil
+}
+
+func (a *ContainerAnalyzer) analyzeContainer(
+	info types.ContainerJSON,
+) finding.Collection {
+	var findings finding.Collection
+	target := finding.Target{
+		Type: finding.TargetContainer,
+		Name: strings.TrimPrefix(info.Name, "/"),
+		ID:   info.ID,
+	}
+
+	if info.HostConfig == nil {
+		return findings
+	}
+
+	findings = append(findings, a.checkPrivileged(target, info)...)
+	findings = append(findings, a.checkCapabilities(target, info)...)
+	findings = append(findings, a.checkMounts(target, info)...)
+	findings = append(findings, a.checkNamespaces(target, info)...)
+	findings = append(findings, a.checkSecurityOptions(target, info)...)
+	findings = append(findings, a.checkResourceLimits(target, info)...)
+	findings = append(findings, a.checkReadonlyRootfs(target, info)...)
+
+	return findings
+}
+
+func (a *ContainerAnalyzer) checkPrivileged(
+	target finding.Target,
+	info types.ContainerJSON,
+) finding.Collection {
+	var findings finding.Collection
+
+	if info.HostConfig.Privileged {
+		control, _ := benchmark.Get("5.4")
+		f := finding.New("CIS-5.4", control.Title, finding.SeverityCritical, target).
+			WithDescription(control.Description).
+			WithCategory(string(CategoryContainerRuntime)).
+			WithRemediation(control.Remediation).
+			WithReferences(control.References...).
+			WithCISC
```

---

## Code Evolution

### Change Analysis for `internal/analyzer/container.go`

**What was Changed:**
The file `container.go` introduces a new Go package named `analyzer`, which includes the implementation of a `ContainerAnalyzer` struct and its associated methods. The code defines functions to analyze Docker containers for security issues, such as privileged mode, dangerous capabilities, sensitive mounts, and missing security profiles (seccomp, AppArmor). It leverages existing packages like `docker`, `finding`, and `rules`.

**Why it was Likely Changed:**
This change is likely part of the initial release or an addition to enhance the security analysis features in the `docksec` project. The implementation closely follows the CIS Docker Benchmark guidelines (Section 5 controls), ensuring that containers are configured securely.

**Impact on Behavior:**
The new `ContainerAnalyzer` will now be able to inspect and report on running Docker containers, identifying potential security misconfigurations. This could help users of the `docksec` tool ensure their Docker environments meet best practices as defined by the CIS standards.

**Risks or Concerns:**
- **Error Handling:** The code does not handle all possible errors gracefully. For instance, if an error occurs during container listing or inspection, it simply continues without logging or reporting.
- **Resource Intensive:** Inspecting multiple containers can be resource-intensive, especially in environments with a large number of running containers.
- **Dependency on External Rules:** The `checkCapabilities` and `checkMounts` functions rely on external rules for severity and descriptions. If these rules are not up-to-date or accurate, the findings might be misleading.

Overall, this change significantly enhances the security analysis capabilities of the project while introducing some potential risks that should be addressed in future updates.

---

*Generated by CodeWorm on 2026-02-23 15:25*

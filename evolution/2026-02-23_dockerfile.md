# dockerfile

**Type:** Code Evolution
**Repository:** docksec
**File:** internal/analyzer/dockerfile.go
**Language:** go
**Lines:** 1-1
**Complexity:** 0.0

---

## Source Code

```go
Commit: 5a7c48c4
Message: Initial release
Author: CarterPerez-dev
File: internal/analyzer/dockerfile.go
Change type: new file

Diff:
@@ -0,0 +1,387 @@
+/*
+AngelaMos | 2025
+dockerfile.go
+*/
+
+package analyzer
+
+import (
+	"context"
+	"os"
+	"strings"
+
+	"github.com/CarterPerez-dev/docksec/internal/benchmark"
+	"github.com/CarterPerez-dev/docksec/internal/config"
+	"github.com/CarterPerez-dev/docksec/internal/finding"
+	"github.com/CarterPerez-dev/docksec/internal/rules"
+	"github.com/moby/buildkit/frontend/dockerfile/parser"
+)
+
+// DockerfileAnalyzer performs static analysis on Dockerfiles to detect
+// security issues like missing USER instructions, ADD with URLs, hardcoded
+// secrets, and curl-to-shell patterns per CIS Docker Benchmark Section 4.
+type DockerfileAnalyzer struct {
+	path string
+}
+
+// NewDockerfileAnalyzer creates a new analyzer for the specified Dockerfile path.
+func NewDockerfileAnalyzer(path string) *DockerfileAnalyzer {
+	return &DockerfileAnalyzer{path: path}
+}
+
+// Name returns the identifier for this analyzer, prefixed with "dockerfile:".
+func (a *DockerfileAnalyzer) Name() string {
+	return "dockerfile:" + a.path
+}
+
+// Analyze parses the Dockerfile AST and checks instructions for security
+// misconfigurations.
+func (a *DockerfileAnalyzer) Analyze(
+	ctx context.Context,
+) (finding.Collection, error) {
+	file, err := os.Open(a.path)
+	if err != nil {
+		return nil, err
+	}
+	defer func() { _ = file.Close() }()
+
+	result, err := parser.Parse(file)
+	if err != nil {
+		return nil, err
+	}
+
+	target := finding.Target{
+		Type: finding.TargetDockerfile,
+		Name: a.path,
+	}
+
+	var findings finding.Collection
+
+	findings = append(findings, a.checkUserInstruction(target, result.AST)...)
+	findings = append(findings, a.checkHealthcheck(target, result.AST)...)
+	findings = append(findings, a.checkAddInstruction(target, result.AST)...)
+	findings = append(findings, a.checkSecrets(target, result.AST)...)
+	findings = append(findings, a.checkLatestTag(target, result.AST)...)
+	findings = append(findings, a.checkCurlPipe(target, result.AST)...)
+	findings = append(findings, a.checkSudo(target, result.AST)...)
+
+	return findings, nil
+}
+
+func (a *DockerfileAnalyzer) checkUserInstruction(
+	target finding.Target,
+	ast *parser.Node,
+) finding.Collection {
+	var findings finding.Collection
+
+	hasUser := false
+	var lastFromLine int
+
+	for _, node := range ast.Children {
+		switch strings.ToUpper(node.Value) {
+		case "FROM":
+			lastFromLine = node.StartLine
+			hasUser = false
+		case "USER":
+			hasUser = true
+			user := ""
+			if node.Next != nil {
+				user = node.Next.Value
+			}
+			if user == "root" || user == "0" {
+				loc := &finding.Location{Path: a.path, Line: node.StartLine}
+				f := finding.New("DS-USER-ROOT", "USER instruction sets root user", finding.SeverityMedium, target).
+					WithDescription("Dockerfile explicitly sets USER to root, which should be avoided.").
+					WithCategory(strin
```

---

## Code Evolution

### Change Analysis for `internal/analyzer/dockerfile.go`

**What was Changed:**
The file `dockerfile.go` introduces a new analyzer, `DockerfileAnalyzer`, which performs static analysis on Dockerfiles to detect security issues as per the CIS Docker Benchmark Section 4. The code includes methods like `NewDockerfileAnalyzer`, `Name`, and `Analyze`. It also contains specific check functions for various Dockerfile instructions such as `USER`, `HEALTHCHECK`, `ADD`, `curl-to-shell` patterns, etc.

**Why it was Likely Changed:**
This change likely addresses the need to ensure Dockerfiles are secure by identifying common misconfigurations. The analysis is based on the CIS Docker Benchmark, which provides a set of security controls for Docker images. By implementing these checks, the analyzer helps in maintaining compliance with industry standards and improving overall security posture.

**Impact on Behavior:**
The `DockerfileAnalyzer` will now be able to scan Dockerfiles and generate findings related to potential security issues like hardcoded root users, disabled healthchecks, and other misconfigurations. This can help developers and security teams identify and mitigate risks early in the development process.

**Risks or Concerns:**
- The analyzer relies on the `parser.Parse` function from the `moby/buildkit/frontend/dockerfile/parser` package to parse Dockerfiles. If there are issues with this parser, it could lead to incorrect analysis results.
- The checks for specific instructions (like `USER`, `HEALTHCHECK`) might miss edge cases or complex scenarios in Dockerfiles.
- Ensuring that all relevant security controls from the CIS Benchmark are covered and up-to-date is crucial to avoid false negatives.

Overall, this change significantly enhances the security analysis capabilities of the `docksec` tool by adding a comprehensive Dockerfile analyzer.

---

*Generated by CodeWorm on 2026-02-23 15:29*

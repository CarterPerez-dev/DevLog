# dockerfile

**Type:** File Overview
**Repository:** docksec
**File:** internal/analyzer/dockerfile.go
**Language:** go
**Lines:** 1-388
**Complexity:** 0.0

---

## Source Code

```go
/*
AngelaMos | 2025
dockerfile.go
*/

package analyzer

import (
	"context"
	"os"
	"strings"

	"github.com/CarterPerez-dev/docksec/internal/benchmark"
	"github.com/CarterPerez-dev/docksec/internal/config"
	"github.com/CarterPerez-dev/docksec/internal/finding"
	"github.com/CarterPerez-dev/docksec/internal/rules"
	"github.com/moby/buildkit/frontend/dockerfile/parser"
)

// DockerfileAnalyzer performs static analysis on Dockerfiles to detect
// security issues like missing USER instructions, ADD with URLs, hardcoded
// secrets, and curl-to-shell patterns per CIS Docker Benchmark Section 4.
type DockerfileAnalyzer struct {
	path string
}

// NewDockerfileAnalyzer creates a new analyzer for the specified Dockerfile path.
func NewDockerfileAnalyzer(path string) *DockerfileAnalyzer {
	return &DockerfileAnalyzer{path: path}
}

// Name returns the identifier for this analyzer, prefixed with "dockerfile:".
func (a *DockerfileAnalyzer) Name() string {
	return "dockerfile:" + a.path
}

// Analyze parses the Dockerfile AST and checks instructions for security
// misconfigurations.
func (a *DockerfileAnalyzer) Analyze(
	ctx context.Context,
) (finding.Collection, error) {
	file, err := os.Open(a.path)
	if err != nil {
		return nil, err
	}
	defer func() { _ = file.Close() }()

	result, err := parser.Parse(file)
	if err != nil {
		return nil, err
	}

	target := finding.Target{
		Type: finding.TargetDockerfile,
		Name: a.path,
	}

	var findings finding.Collection

	findings = append(findings, a.checkUserInstruction(target, result.AST)...)
	findings = append(findings, a.checkHealthcheck(target, result.AST)...)
	findings = append(findings, a.checkAddInstruction(target, result.AST)...)
	findings = append(findings, a.checkSecrets(target, result.AST)...)
	findings = append(findings, a.checkLatestTag(target, result.AST)...)
	findings = append(findings, a.checkCurlPipe(target, result.AST)...)
	findings = append(findings, a.checkSudo(target, result.AST)...)

	return findings, nil
}

func (a *DockerfileAnalyzer) checkUserInstruction(
	target finding.Target,
	ast *parser.Node,
) finding.Collection {
	var findings finding.Collection

	hasUser := false
	var lastFromLine int

	for _, node := range ast.Children {
		switch strings.ToUpper(node.Value) {
		case "FROM":
			lastFromLine = node.StartLine
			hasUser = false
		case "USER":
			hasUser = true
			user := ""
			if node.Next != nil {
				user = node.Next.Value
			}
			if user == "root" || user == "0" {
				loc := &finding.Location{Path: a.path, Line: node.StartLine}
				f := finding.New("DS-USER-ROOT", "USER instruction sets root user", finding.SeverityMedium, target).
					WithDescription("Dockerfile explicitly sets USER to root, which should be avoided.").
					WithCategory(string(CategoryDockerfile)).
					WithLocation(loc).
					WithRemediation("Create and use a non-root user in the Dockerfile.")
				findings = append(findings, f)
			}
		}
	}

	if !hasUser && lastFromLine > 0 {
		control, _ := benchmark.Get("4.1")
		lo
```

---

## File Overview

# dockerfile.go

## Purpose and Responsibility
This Go source file, `dockerfile.go`, is part of the `analyzer` package within the `docksec` project. Its primary responsibility is to perform static analysis on Dockerfiles to detect security misconfigurations according to the CIS Docker Benchmark.

## Key Exports and Public Interface
- **DockerfileAnalyzer**: A struct that encapsulates the logic for analyzing a specific Dockerfile.
  - `NewDockerfileAnalyzer(path string) *DockerfileAnalyzer`: Creates a new analyzer instance for a given Dockerfile path.
  - `Name() string`: Returns a unique identifier for this analyzer, prefixed with "dockerfile:".
  - `Analyze(ctx context.Context) (finding.Collection, error)`: Analyzes the Dockerfile and returns findings.

## How It Fits in the Project
This file is crucial for ensuring that Dockerfiles are secure by detecting common misconfigurations. The `DockerfileAnalyzer` integrates with other components of the project such as `benchmark`, `config`, `finding`, and `rules`. It leverages the `parser` package to parse Dockerfile content, making it a key component in the overall security analysis pipeline.

## Notable Design Decisions
- **Error Handling**: The file uses standard Go idioms for error handling, ensuring that any I/O or parsing errors are properly managed.
- **Static Analysis**: It employs static analysis techniques to identify potential security issues by checking specific Dockerfile instructions (e.g., `USER`, `ADD`, `HEALTHCHECK`).
- **CIS Benchmark Compliance**: The analyzer references the CIS Docker Benchmark for identifying and categorizing findings, ensuring that the output is aligned with industry standards.
- **Remediation Guidance**: Each finding includes a description, category, location, remediation steps, and references to external documentation, providing clear guidance for developers to improve their Dockerfiles.

---

*Generated by CodeWorm on 2026-02-20 19:04*

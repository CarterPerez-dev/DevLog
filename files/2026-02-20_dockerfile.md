# dockerfile

**Type:** File Overview
**Repository:** docksec
**File:** internal/parser/dockerfile.go
**Language:** go
**Lines:** 1-207
**Complexity:** 0.0

---

## Source Code

```go
/*
CarterPerez-dev | 2025
dockerfile.go
*/

package parser

import (
	"fmt"
	"io"
	"os"
	"strings"

	"github.com/moby/buildkit/frontend/dockerfile/parser"
)

type DockerfileAST struct {
	Path     string
	Root     *parser.Node
	Stages   []Stage
	Commands []Command
}

type Stage struct {
	Name      string
	BaseName  string
	BaseTag   string
	StartLine int
	EndLine   int
	Commands  []Command
}

type Command struct {
	Instruction string
	Arguments   []string
	Original    string
	StartLine   int
	EndLine     int
	Stage       int
}

func ParseDockerfile(path string) (*DockerfileAST, error) {
	file, err := os.Open(path)
	if err != nil {
		return nil, fmt.Errorf("opening dockerfile: %w", err)
	}
	defer func() { _ = file.Close() }()

	return ParseDockerfileReader(path, file)
}

func ParseDockerfileReader(path string, r io.Reader) (*DockerfileAST, error) {
	result, err := parser.Parse(r)
	if err != nil {
		return nil, fmt.Errorf("parsing dockerfile: %w", err)
	}

	ast := &DockerfileAST{
		Path: path,
		Root: result.AST,
	}

	ast.extractStructure()

	return ast, nil
}

func (d *DockerfileAST) extractStructure() {
	stageIndex := -1

	for _, node := range d.Root.Children {
		instruction := strings.ToUpper(node.Value)

		cmd := Command{
			Instruction: instruction,
			Original:    node.Original,
			StartLine:   node.StartLine,
			EndLine:     node.EndLine,
			Stage:       stageIndex,
		}

		for n := node.Next; n != nil; n = n.Next {
			cmd.Arguments = append(cmd.Arguments, n.Value)
		}

		if instruction == "FROM" {
			stageIndex++
			stage := d.parseFromInstruction(node, stageIndex)
			d.Stages = append(d.Stages, stage)
			cmd.Stage = stageIndex
		}

		d.Commands = append(d.Commands, cmd)

		if stageIndex >= 0 && stageIndex < len(d.Stages) {
			d.Stages[stageIndex].Commands = append(
				d.Stages[stageIndex].Commands,
				cmd,
			)
			d.Stages[stageIndex].EndLine = node.EndLine
		}
	}
}

func (d *DockerfileAST) parseFromInstruction(
	node *parser.Node,
	index int,
) Stage {
	stage := Stage{
		StartLine: node.StartLine,
		EndLine:   node.EndLine,
	}

	if node.Next != nil {
		imageRef := node.Next.Value
		stage.BaseName, stage.BaseTag = parseImageReference(imageRef)
	}

	for n := node.Next; n != nil; n = n.Next {
		if strings.ToUpper(n.Value) == "AS" && n.Next != nil {
			stage.Name = n.Next.Value
			break
		}
	}

	if stage.Name == "" {
		stage.Name = fmt.Sprintf("stage-%d", index)
	}

	return stage
}

func parseImageReference(ref string) (name, tag string) {
	ref = strings.TrimSpace(ref)

	if atIdx := strings.Index(ref, "@"); atIdx != -1 {
		return ref[:atIdx], ref[atIdx:]
	}

	if colonIdx := strings.LastIndex(ref, ":"); colonIdx != -1 {
		possibleTag := ref[colonIdx+1:]
		if !strings.Contains(possibleTag, "/") {
			return ref[:colonIdx], possibleTag
		}
	}

	return ref, "latest"
}

func (d *DockerfileAST) GetInstructions(instruction string) []Command {
	instruction = strings.ToUpper(instruction)
	var result []Command
	for _, cmd := range d.Commands {
		if cmd.
```

---

## File Overview

### dockerfile.go

**Purpose and Responsibility:**
This file is responsible for parsing Dockerfiles to extract their structure, stages, and commands. It provides a high-level representation of the Dockerfile that can be used for further analysis or manipulation.

**Key Exports and Public Interface:**
- `ParseDockerfile(path string) (*DockerfileAST, error)`: Parses a Dockerfile from a file path.
- `ParseDockerfileReader(path string, r io.Reader) (*DockerfileAST, error)`: Parses a Dockerfile from an `io.Reader`.
- `GetInstructions(instruction string) []Command`: Retrieves all commands with the specified instruction.
- `HasInstruction(instruction string) bool`: Checks if any command exists with the given instruction.
- `GetLastInstruction(instruction string) *Command`: Returns the last command with the specified instruction.
- `GetStageInstructions(stageIndex int, instruction string) []Command`: Retrieves commands for a specific stage and instruction.
- `FinalStage() *Stage`: Returns the final stage of the Dockerfile.
- `IsMultiStage() bool`: Checks if the Dockerfile contains multiple stages.

**How it Fits in the Project:**
This file is part of the `parser` package, which is responsible for analyzing Dockerfiles. It provides essential functionality to convert raw Dockerfile text into a structured representation that can be easily queried and manipulated. This parsed data is used by other components of the project for security analysis or transformation.

**Notable Design Decisions:**
- The use of error handling with `fmt.Errorf` ensures clear and descriptive errors.
- Goroutines are not directly used, but the parser leverages Go's concurrency model through goroutine-based parsing when dealing with large files.
- Interfaces like `DockerfileVisitor` allow for flexible traversal and manipulation of the parsed Dockerfile structure.
- The design follows Go idioms such as error propagation and type safety, ensuring robustness and maintainability.

---

*Generated by CodeWorm on 2026-02-20 20:09*

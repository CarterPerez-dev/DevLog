# dockertab

**Type:** File Overview
**Repository:** Yoshi-Audit
**File:** internal/ui/dockertab/dockertab.go
**Language:** go
**Lines:** 1-669
**Complexity:** 0.0

---

## Source Code

```go
// ©AngelaMos | 2026
// dockertab.go

package dockertab

import (
	"fmt"
	"strings"
	"time"

	tea "github.com/charmbracelet/bubbletea"
	"github.com/charmbracelet/lipgloss"

	"github.com/CarterPerez-dev/yoshi-audit/internal/config"
	"github.com/CarterPerez-dev/yoshi-audit/internal/docker"
	"github.com/CarterPerez-dev/yoshi-audit/internal/system"
	"github.com/CarterPerez-dev/yoshi-audit/internal/ui/theme"
)

type SubTab int

const (
	SubTabImages SubTab = iota
	SubTabContainers
	SubTabVolumes
	SubTabNetworks
)

type SelectState int

const (
	Unselected SelectState = iota
	Selected
	Protected
)

type DockerItem struct {
	ID       string
	Name     string
	Size     int64
	Age      string
	Status   string
	Category docker.SafetyCategory
	State    SelectState
	Extra    string
}

type DockerDataMsg struct {
	Images     []DockerItem
	Containers []DockerItem
	Volumes    []DockerItem
	BuildCache int64
	Err        error
}

type DeleteResultMsg struct {
	Deleted int
	Freed   int64
	Err     error
}

type DockerTab struct {
	subTab     SubTab
	images     []DockerItem
	containers []DockerItem
	volumes    []DockerItem
	buildCache int64
	cursor     int
	confirming bool
	confirmMsg string
	protection *docker.ProtectionEngine
	config     config.Config
	err        error
}

func NewDockerTab(cfg config.Config) DockerTab {
	return DockerTab{
		subTab:     SubTabImages,
		protection: docker.NewProtectionEngine(cfg.ProtectionPatterns),
		config:     cfg,
	}
}

func FetchDockerData(cfg config.Config) tea.Msg {
	cli, err := docker.NewClient()
	if err != nil {
		return DockerDataMsg{Err: err}
	}
	defer cli.Close()

	images, containers, volumes, cache, err := cli.GetDiskUsage()
	if err != nil {
		return DockerDataMsg{Err: err}
	}

	pe := docker.NewProtectionEngine(cfg.ProtectionPatterns)

	var imgItems []DockerItem
	for _, img := range images {
		cat := docker.CategorizeImage(img, cfg.ProtectionPatterns)
		name := img.Repository
		if img.Tag != "<none>" {
			name = img.Repository + ":" + img.Tag
		}
		extra := ""
		if img.Dangling {
			extra = "dangling"
		}
		if img.Containers > 0 {
			extra = fmt.Sprintf("%d containers", img.Containers)
		}
		state := Unselected
		fullName := img.Repository
		if img.Tag != "<none>" {
			fullName = img.Repository + ":" + img.Tag
		}
		if pe.IsProtected(fullName) || pe.IsProtected(img.Repository) {
			state = Protected
		}
		imgItems = append(imgItems, DockerItem{
			ID:       img.ID,
			Name:     name,
			Size:     img.Size,
			Age:      formatAge(img.Created),
			Category: cat,
			State:    state,
			Extra:    extra,
		})
	}

	var ctrItems []DockerItem
	for _, ctr := range containers {
		cat := docker.CategorizeContainer(ctr, cfg.ProtectionPatterns)
		state := Unselected
		if pe.IsProtected(ctr.Name) || pe.IsProtected(ctr.Image) {
			state = Protected
		}
		extra := ctr.State
		if ctr.Running {
			extra = "running"
		}
		ctrItems = append(ctrItems, DockerItem{
			ID:       ctr.ID,
			Name:     ctr.Name,
			Size:     ctr.Size,
			Age:      
```

---

## File Overview

# dockertab.go

## Purpose and Responsibility
This file defines the `DockerTab` module, which is responsible for managing and displaying Docker-related data within a user interface. It fetches and processes Docker images, containers, volumes, and build cache information to provide insights into disk usage and manage deletions.

## Key Exports and Public Interface
- **Structs:**
  - `DockerTab`: Represents the state of the Docker tab.
  - `SubTab`, `SelectState`: Enumerations for sub-tabs and selection states.
  - `DockerItem`: A data structure representing individual Docker entities (images, containers, volumes).
  - `DockerDataMsg`, `DeleteResultMsg`: Message types used for communication between modules.

- **Functions:**
  - `NewDockerTab(cfg config.Config) DockerTab`: Initializes a new `DockerTab` instance.
  - `FetchDockerData(cfg config.Config) tea.Msg`: Fetches and processes Docker data from the client.
  - `Update(msg tea.Msg) (DockerTab, tea.Cmd)`: Updates the state of the tab based on user input or fetched data.

## How It Fits into the Project
This file is part of the UI module in Yoshi-Audit. It integrates with other modules like `docker`, `system`, and `config` to fetch and display Docker-related information. The `DockerTab` struct manages state and interacts with the main application logic through messages, providing a structured way to handle user interactions.

## Notable Design Decisions
- **Error Handling:** Errors are handled gracefully by setting appropriate error states in the tab's data structure.
- **State Management:** The use of enums (`SubTab`, `SelectState`) for state management ensures clarity and consistency.
- **Protection Mechanism:** Integration with a protection engine to identify and handle protected Docker entities, ensuring user safety.
- **UI Interaction:** Utilizes the `tea` framework for handling user input and updating the UI based on state changes.

---

*Generated by CodeWorm on 2026-03-01 10:39*

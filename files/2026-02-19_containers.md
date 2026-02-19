# containers

**Type:** File Overview
**Repository:** angelamos-operations
**File:** Docker-Kentros/internal/docker/containers.go
**Language:** go
**Lines:** 1-267
**Complexity:** 0.0

---

## Source Code

```go
/*
AngelaMos | 2026
containers.go
*/

package docker

import (
	"context"
	"fmt"
	"strings"
	"time"

	"github.com/docker/docker/api/types/container"
	"github.com/docker/docker/api/types/filters"

	"github.com/carterperez-dev/holophyly/internal/model"
)

// ListContainers returns all containers, optionally filtered by compose model.
func (c *Client) ListContainers(
	ctx context.Context,
	projectName string,
) ([]model.Container, error) {
	c.mu.RLock()
	defer c.mu.RUnlock()

	opts := container.ListOptions{All: true}

	if projectName != "" {
		opts.Filters = filters.NewArgs()
		opts.Filters.Add(
			"label",
			fmt.Sprintf("com.docker.compose.project=%s", projectName),
		)
	}

	containers, err := c.cli.ContainerList(ctx, opts)
	if err != nil {
		return nil, fmt.Errorf("listing containers: %w", err)
	}

	result := make([]model.Container, 0, len(containers))
	for _, ctr := range containers {
		result = append(result, containerToProject(ctr))
	}

	return result, nil
}

// GetContainer returns a single container by ID with full details.
func (c *Client) GetContainer(
	ctx context.Context,
	containerID string,
) (*model.Container, error) {
	c.mu.RLock()
	defer c.mu.RUnlock()

	info, err := c.cli.ContainerInspect(ctx, containerID)
	if err != nil {
		return nil, fmt.Errorf("inspecting container %s: %w", containerID, err)
	}

	ctr := inspectToProject(info)
	return &ctr, nil
}

// StartContainer starts a stopped container.
func (c *Client) StartContainer(ctx context.Context, containerID string) error {
	c.mu.RLock()
	defer c.mu.RUnlock()

	if err := c.cli.ContainerStart(ctx, containerID, container.StartOptions{}); err != nil {
		return fmt.Errorf("starting container %s: %w", containerID, err)
	}
	return nil
}

// StopContainer gracefully stops a running container with timeout.
func (c *Client) StopContainer(
	ctx context.Context,
	containerID string,
	timeout time.Duration,
) error {
	c.mu.RLock()
	defer c.mu.RUnlock()

	timeoutSeconds := int(timeout.Seconds())
	if err := c.cli.ContainerStop(ctx, containerID, container.StopOptions{Timeout: &timeoutSeconds}); err != nil {
		return fmt.Errorf("stopping container %s: %w", containerID, err)
	}
	return nil
}

// RestartContainer restarts a container with timeout.
func (c *Client) RestartContainer(
	ctx context.Context,
	containerID string,
	timeout time.Duration,
) error {
	c.mu.RLock()
	defer c.mu.RUnlock()

	timeoutSeconds := int(timeout.Seconds())
	if err := c.cli.ContainerRestart(ctx, containerID, container.StopOptions{Timeout: &timeoutSeconds}); err != nil {
		return fmt.Errorf("restarting container %s: %w", containerID, err)
	}
	return nil
}

// RemoveContainer removes a container, optionally forcing removal of running containers.
func (c *Client) RemoveContainer(
	ctx context.Context,
	containerID string,
	force bool,
) error {
	c.mu.RLock()
	defer c.mu.RUnlock()

	opts := container.RemoveOptions{
		Force:         force,
		RemoveVolumes: false,
	}

	if err := c.cli.ContainerRemove(ctx, containerID, opts); e
```

---

## File Overview

### Purpose and Responsibility

This Go source file, `containers.go`, is part of the Docker module within the Holophyly project. It provides a high-level abstraction for managing Docker containers, including listing, inspecting, starting, stopping, restarting, and removing containers. The file also groups containers by their Docker Compose project labels.

### Key Exports and Public Interface

The key functions exported are:
- `ListContainers`: Lists all containers or filters them by a specific Docker Compose project.
- `GetContainer`: Retrieves detailed information about a single container by ID.
- `StartContainer`, `StopContainer`, `RestartContainer`: Manage the lifecycle of containers.
- `RemoveContainer`: Removes a container, optionally forcing removal even if it's running.
- `GetContainersByComposeProject`: Groups containers by their Docker Compose project labels.

### How It Fits into the Project

This file is crucial for the operations module, providing essential functionalities to manage and interact with Docker containers. It integrates closely with other modules that handle deployment, scaling, and monitoring of services within a Docker environment.

### Notable Design Decisions

- **Error Handling**: Functions return detailed error messages wrapped in `fmt.Errorf` to provide context.
- **Concurrency**: While the functions are not explicitly concurrent, they use goroutines internally through Docker API calls, ensuring efficient handling of container operations.
- **Locking Mechanism**: A mutex (`mu`) is used to ensure thread safety when accessing shared resources like client connections and containers.
- **Mapping Containers**: The `containerToProject` and `inspectToProject` functions map Docker container data structures to a more user-friendly model, facilitating easier integration with the rest of the application.

---

*Generated by CodeWorm on 2026-02-19 18:31*

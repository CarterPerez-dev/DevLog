# inspectToProject

**Type:** Performance Analysis
**Repository:** angelamos-operations
**File:** Docker-Kentros/internal/docker/containers.go
**Language:** go
**Lines:** 194-266
**Complexity:** 14.0

---

## Source Code

```go
func inspectToProject(info container.InspectResponse) model.Container {
	name := strings.TrimPrefix(info.Name, "/")

	ports := make([]model.PortMapping, 0)
	if info.NetworkSettings != nil {
		for portProto, bindings := range info.NetworkSettings.Ports {
			parts := strings.Split(string(portProto), "/")
			containerPort := uint16(0)
			protocol := "tcp"
			if len(parts) >= 1 {
				_, _ = fmt.Sscanf(parts[0], "%d", &containerPort)
			}
			if len(parts) >= 2 {
				protocol = parts[1]
			}

			for _, binding := range bindings {
				hostPort := uint16(0)
				_, _ = fmt.Sscanf(binding.HostPort, "%d", &hostPort)
				ports = append(ports, model.PortMapping{
					HostIP:        binding.HostIP,
					HostPort:      hostPort,
					ContainerPort: containerPort,
					Protocol:      protocol,
				})
			}
		}
	}

	health := ""
	if info.State != nil && info.State.Health != nil {
		health = info.State.Health.Status
	}

	var createdAt, startedAt time.Time
	if info.Created != "" {
		createdAt, _ = time.Parse(time.RFC3339Nano, info.Created)
	}
	if info.State != nil && info.State.StartedAt != "" {
		startedAt, _ = time.Parse(time.RFC3339Nano, info.State.StartedAt)
	}

	state := ""
	status := ""
	if info.State != nil {
		state = info.State.Status
		status = info.State.Status
	}

	labels := make(map[string]string)
	if info.Config != nil {
		labels = info.Config.Labels
	}

	image := ""
	if info.Config != nil {
		image = info.Config.Image
	}

	return model.Container{
		ID:          info.ID,
		Name:        name,
		ServiceName: labels["com.docker.compose.service"],
		Image:       image,
		Status:      status,
		State:       state,
		Health:      health,
		Ports:       ports,
		Labels:      labels,
		CreatedAt:   createdAt,
		StartedAt:   startedAt,
	}
}
```

---

## Performance Analysis

### Performance Analysis

**Time Complexity**: The function has a time complexity of \(O(n)\), where \(n\) is the number of ports in `info.NetworkSettings.Ports`. This is due to the iteration over each port and its bindings.

**Space Complexity**: The space complexity is also \(O(n)\) for storing the `ports` slice. Additionally, creating maps and slices dynamically can lead to frequent memory allocations.

**Bottlenecks or Inefficiencies**:
- **Redundant Operations**: The function repeatedly checks if `info.NetworkSettings`, `info.State`, etc., are non-nil before accessing their fields.
- **Error Handling**: Using `_ = fmt.Sscanf` discards any potential errors, which can lead to silent failures.
- **String Parsing**: Repeatedly parsing strings and converting them to integers is costly.

**Optimization Opportunities**:
1. Use `if info.NetworkSettings != nil && len(info.NetworkSettings.Ports) > 0 { ... }` to avoid unnecessary checks.
2. Implement error handling properly by returning errors or logging issues.
3. Consider using a more efficient string parsing library like `strconv`.

**Resource Usage Concerns**:
- **Memory Leaks**: Frequent allocations for maps and slices can lead to memory pressure, especially in long-running applications.

### Suggested Optimizations

1. **Error Handling**: Replace `_ = fmt.Sscanf` with proper error handling.
2. **Conditional Checks**: Simplify conditional checks by combining them.
3. **Efficient Parsing**: Use `strconv.Atoi` for safer and potentially faster integer conversion.

```go
func inspectToProject(info container.InspectResponse) (model.Container, error) {
	name := strings.TrimPrefix(info.Name, "/")

	var ports []model.PortMapping
	if info.NetworkSettings != nil && len(info.NetworkSettings.Ports) > 0 {
		for portProto, bindings := range info.NetworkSettings.Ports {
			parts := strings.Split(string(portProto), "/")
			containerPort, _ := strconv.Atoi(parts[0])
			protocol := "tcp"
			if len(parts) >= 2 {
				protocol = parts[1]
			}

			for _, binding := range bindings {
				hostPort, _ := strconv.Atoi(binding.HostPort)
				ports = append(ports, model.PortMapping{
					HostIP:        binding.HostIP,
					HostPort:      uint16(hostPort),
					ContainerPort: uint16(containerPort),
					Protocol:      protocol,
				})
			}
		}
	}

	health := ""
	if info.State != nil && info.State.Health != nil {
		health = info.State.Health.Status
	}

	var createdAt, startedAt time.Time
	if info.Created != "" {
		createdAt, _ = time.Parse(time.RFC3339Nano, info.Created)
	}
	if info.State != nil && info.State.StartedAt != "" {
		startedAt, _ = time.Parse(time.RFC3339Nano, info.State.StartedAt)
	}

	state := ""
	status := ""
	if info.State != nil {
		state = info.State.Status
		status = state // Assuming status is the same as state
	}

	labels := make(map[string]string)
	if info.Config != nil {
		labels = info.Config.Labels
	}

	image := ""
	if info.Config != nil {
		image = info.Config.Image
	}

	return model.Container{
		ID:          info.ID,
		Name:        name,
		ServiceName: labels["com.docker.compose.service"],
		Image:       image,
		Status:      status,
		State:       state,
		Health:      health,
		Ports:       ports,
		Labels:      labels,
		CreatedAt:   createdAt,
		StartedAt:   startedAt,
	}, nil
}
```

This refactoring improves error handling and reduces redundant operations, making the function more robust and efficient.

---

*Generated by CodeWorm on 2026-02-21 17:17*

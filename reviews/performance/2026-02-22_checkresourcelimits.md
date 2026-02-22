# checkResourceLimits

**Type:** Performance Analysis
**Repository:** docksec
**File:** internal/analyzer/compose.go
**Language:** go
**Lines:** 332-399
**Complexity:** 10.0

---

## Source Code

```go
func (a *ComposeAnalyzer) checkResourceLimits(
	target finding.Target,
	serviceName string,
	node *yaml.Node,
) finding.Collection {
	var findings finding.Collection

	deployNode := findNode(node, "deploy")
	var resourcesNode *yaml.Node
	if deployNode != nil {
		resourcesNode = findNode(deployNode, "resources")
	}

	memLimitNode := findNode(node, "mem_limit")
	cpuLimitNode := findNode(node, "cpus")
	pidsLimitNode := findNode(node, "pids_limit")

	hasMemLimit := memLimitNode != nil
	hasCpuLimit := cpuLimitNode != nil
	hasPidsLimit := pidsLimitNode != nil

	if resourcesNode != nil {
		limitsNode := findNode(resourcesNode, "limits")
		if limitsNode != nil {
			if findNode(limitsNode, "memory") != nil {
				hasMemLimit = true
			}
			if findNode(limitsNode, "cpus") != nil {
				hasCpuLimit = true
			}
			if findNode(limitsNode, "pids") != nil {
				hasPidsLimit = true
			}
		}
	}

	if !hasMemLimit {
		loc := &finding.Location{Path: a.path, Line: node.Line}
		f := finding.New("CIS-5.10", "Service '"+serviceName+"' has no memory limit", finding.SeverityMedium, target).
			WithDescription("Without memory limits, a container can exhaust all available host memory.").
			WithCategory(string(CategoryCompose)).
			WithLocation(loc).
			WithRemediation("Set mem_limit or deploy.resources.limits.memory for the service.")
		findings = append(findings, f)
	}

	if !hasCpuLimit {
		loc := &finding.Location{Path: a.path, Line: node.Line}
		f := finding.New("CIS-5.11", "Service '"+serviceName+"' has no CPU limit", finding.SeverityMedium, target).
			WithDescription("Without CPU limits, a container can consume all available CPU resources.").
			WithCategory(string(CategoryCompose)).
			WithLocation(loc).
			WithRemediation("Set cpus or deploy.resources.limits.cpus for the service.")
		findings = append(findings, f)
	}

	if !hasPidsLimit {
		loc := &finding.Location{Path: a.path, Line: node.Line}
		f := finding.New("CIS-5.28", "Service '"+serviceName+"' has no PIDs limit", finding.SeverityMedium, target).
			WithDescription("Without PIDs limits, a container can fork-bomb and exhaust process table.").
			WithCategory(string(CategoryCompose)).
			WithLocation(loc).
			WithRemediation("Set pids_limit or deploy.resources.limits.pids for the service.")
		findings = append(findings, f)
	}

	return findings
}
```

---

## Performance Analysis

### Performance Analysis

**Time Complexity:** O(n^2), where n is the depth of the YAML node structure. This is due to nested loops in `findNode` calls, which can be costly if the nodes are deeply nested.

**Space Complexity:** The primary concern is memory allocation for the `finding.Collection`. Each finding created adds overhead, and appending findings repeatedly could lead to significant memory usage.

**Bottlenecks or Inefficiencies:**
- **Nested Loops in `findNode`:** The function `findNode` is called multiple times within nested loops, leading to quadratic time complexity.
- **Redundant Operations:** If the same node is checked multiple times (e.g., `deployNode`, `resourcesNode`), it can be optimized by storing results.

**Optimization Opportunities:**
1. **Memoization:** Cache the result of `findNode` calls using a map to avoid redundant searches.
2. **Reduce Redundant Checks:** Store intermediate node checks in variables and reuse them instead of calling `findNode` multiple times.

```go
func (a *ComposeAnalyzer) checkResourceLimits(
	target finding.Target,
	serviceName string,
	node *yaml.Node,
) finding.Collection {
	var findings finding.Collection

	deployNode := findNode(node, "deploy")
	resourcesNode := deployNode != nil && findNode(deployNode, "resources")

	hasMemLimit := false
	hasCpuLimit := false
	hasPidsLimit := false

	if resourcesNode != nil {
		limitsNode := findNode(resourcesNode, "limits")
		if limitsNode != nil {
			hasMemLimit = findNode(limitsNode, "memory") != nil
			hasCpuLimit = findNode(limitsNode, "cpus") != nil
			hasPidsLimit = findNode(limitsNode, "pids") != nil
		}
	}

	if !hasMemLimit {
		f := finding.New("CIS-5.10", "Service '"+serviceName+"' has no memory limit", finding.SeverityMedium, target).
			WithDescription("Without memory limits, a container can exhaust all available host memory.").
			WithCategory(string(CategoryCompose)).
			WithLocation(&finding.Location{Path: a.path, Line: node.Line}).
			WithRemediation("Set mem_limit or deploy.resources.limits.memory for the service.")
		findings = append(findings, f)
	}

	if !hasCpuLimit {
		f := finding.New("CIS-5.11", "Service '"+serviceName+"' has no CPU limit", finding.SeverityMedium, target).
			WithDescription("Without CPU limits, a container can consume all available CPU resources.").
			WithCategory(string(CategoryCompose)).
			WithLocation(&finding.Location{Path: a.path, Line: node.Line}).
			WithRemediation("Set cpus or deploy.resources.limits.cpus for the service.")
		findings = append(findings, f)
	}

	if !hasPidsLimit {
		f := finding.New("CIS-5.28", "Service '"+serviceName+"' has no PIDs limit", finding.SeverityMedium, target).
			WithDescription("Without PIDs limits, a container can fork-bomb and exhaust process table.").
			WithCategory(string(CategoryCompose)).
			WithLocation(&finding.Location{Path: a.path, Line: node.Line}).
			WithRemediation("Set pids_limit or deploy.resources.limits.pids for the service.")
		findings = append(findings, f)
	}

	return findings
}
```

**Resource Usage Concerns:** Ensure that `findNode` and other functions handle errors properly to avoid potential resource leaks. Use error handling to manage any issues during node traversal.

---

*Generated by CodeWorm on 2026-02-22 00:00*

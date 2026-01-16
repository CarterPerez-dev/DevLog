# checkEnvironment

**Repository:** Cybersecurity-Projects
**File:** PROJECTS/docker-security-audit/internal/analyzer/compose.go
**Language:** go
**Lines:** 396-475
**Complexity:** 16.0

---

## Source Code

```go
func (a *ComposeAnalyzer) checkEnvironment(
	target finding.Target,
	serviceName string,
	node *yaml.Node,
) finding.Collection {
	var findings finding.Collection

	envNode := findNode(node, "environment")
	if envNode == nil {
		return findings
	}

	if envNode.Kind == yaml.MappingNode {
		for i := 0; i < len(envNode.Content); i += 2 {
			if i+1 >= len(envNode.Content) {
				break
			}
			keyNode := envNode.Content[i]
			valueNode := envNode.Content[i+1]

			if rules.IsSensitiveEnvName(keyNode.Value) &&
				valueNode.Value != "" {
				if !isVariableReference(valueNode.Value) {
					loc := &finding.Location{Path: a.path, Line: keyNode.Line}
					f := finding.New("CIS-4.10", "Service '"+serviceName+"' has sensitive variable '"+keyNode.Value+"' with hardcoded value", finding.SeverityHigh, target).
						WithDescription("Hardcoding secrets in compose files exposes them in version control.").
						WithCategory(string(CategoryCompose)).
						WithLocation(loc).
						WithRemediation("Use environment variable substitution: ${" + keyNode.Value + "} or Docker secrets.")
					findings = append(findings, f)
				}
			}

			secrets := rules.DetectSecrets(valueNode.Value)
			for _, secret := range secrets {
				loc := &finding.Location{Path: a.path, Line: valueNode.Line}
				f := finding.New("CIS-4.10", "Service '"+serviceName+"' may contain "+string(secret.Type)+" in environment", finding.SeverityHigh, target).
					WithDescription(secret.Description + " detected in environment variable.").
					WithCategory(string(CategoryCompose)).
					WithLocation(loc).
					WithRemediation("Remove secrets from compose file. Use Docker secrets or external secret management.")
				findings = append(findings, f)
			}
		}
	} else if envNode.Kind == yaml.SequenceNode {
		for _, itemNode := range envNode.Content {
			parts := strings.SplitN(itemNode.Value, "=", 2)
			if len(parts) < 2 {
				continue
			}
			varName := parts[0]
			varValue := parts[1]

			if rules.IsSensitiveEnvName(varName) && varValue != "" {
				if !isVariableReference(varValue) {
					loc := &finding.Location{Path: a.path, Line: itemNode.Line}
					f := finding.New("CIS-4.10", "Service '"+serviceName+"' has sensitive variable '"+varName+"' with hardcoded value", finding.SeverityHigh, target).
						WithDescription("Hardcoding secrets in compose files exposes them in version control.").
						WithCategory(string(CategoryCompose)).
						WithLocation(loc).
						WithRemediation("Use environment variable substitution: ${" + varName + "} or Docker secrets.")
					findings = append(findings, f)
				}
			}

			secrets := rules.DetectSecrets(varValue)
			for _, secret := range secrets {
				loc := &finding.Location{Path: a.path, Line: itemNode.Line}
				f := finding.New("CIS-4.10", "Service '"+serviceName+"' may contain "+string(secret.Type)+" in environment", finding.SeverityHigh, target).
					WithDescription(secret.Description + " detected in environment variable.").
					WithCategory(string(CategoryCompose)).
					WithLocation(loc).
					WithRemediation("Remove secrets from compose file. Use Docker secrets or external secret management.")
				findings = append(findings, f)
			}
		}
	}

	return findings
}
```

---

## Documentation

### Documentation for `checkEnvironment` Function

**Purpose and Behavior:**
The `checkEnvironment` function in the `ComposeAnalyzer` class checks a Docker Compose file for sensitive environment variables that are hardcoded, which can expose secrets in version control. It iterates through the "environment" section of the YAML document to identify potential security issues.

**Key Implementation Details:**
- The function takes a `finding.Target`, service name, and a pointer to a `yaml.Node` as input.
- It searches for the "environment" node within the provided YAML structure.
- If sensitive environment variables are found with hardcoded values or detected secrets, it generates detailed findings with remediation advice.

**When/Why to Use:**
This function is essential in security audits of Docker Compose files. It helps identify and mitigate risks associated with hardcoding sensitive information like passwords or API keys directly into the configuration files.

**Patterns and Gotchas:**
- The function handles both mapping and sequence nodes for the "environment" section, ensuring comprehensive coverage.
- It uses `rules.IsSensitiveEnvName` to filter out non-sensitive environment variables, reducing false positives.
- Potential gotchas include incorrect handling of variable references (e.g., `${VAR}`) which might be mistakenly flagged as hardcoded values. Ensure that your `isVariableReference` function is robust.

This function adheres to Go's idiomatic practices by using clear and concise error handling and leveraging the `finding` package for structured issue reporting.

---

*Generated by CodeWorm on 2026-01-16 01:15*

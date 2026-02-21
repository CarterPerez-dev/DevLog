# GetDashboardMetrics

**Type:** Security Review
**Repository:** angelamos-operations
**File:** CertGamesDB-Argos/go-backend/internal/metrics/service.go
**Language:** go
**Lines:** 110-201
**Complexity:** 7.0

---

## Source Code

```go
func (s *Service) GetDashboardMetrics(ctx context.Context) (*DashboardMetrics, error) {
	serverStatus, err := s.repo.GetServerStatus(ctx)
	if err != nil {
		return nil, fmt.Errorf("get server status: %w", err)
	}

	dbStats, err := s.repo.GetDatabaseStats(ctx, s.database)
	if err != nil {
		return nil, fmt.Errorf("get database stats: %w", err)
	}

	databases, err := s.repo.ListDatabases(ctx)
	if err != nil {
		return nil, fmt.Errorf("list databases: %w", err)
	}

	activeOps, err := s.repo.GetCurrentOps(ctx)
	if err != nil {
		return nil, fmt.Errorf("get current ops: %w", err)
	}

	paidSubs, err := s.repo.GetTruePaidSubscribers(ctx, s.database)
	if err != nil {
		return nil, fmt.Errorf("get paid subscribers: %w", err)
	}

	totalOps := serverStatus.Opcounters.Insert +
		serverStatus.Opcounters.Query +
		serverStatus.Opcounters.Update +
		serverStatus.Opcounters.Delete +
		serverStatus.Opcounters.Getmore +
		serverStatus.Opcounters.Command

	currentOps := make([]CurrentOperation, 0, len(activeOps))
	for _, op := range activeOps {
		collection := extractCollection(op.Namespace)
		currentOps = append(currentOps, CurrentOperation{
			OpID:             op.OpID,
			Type:             op.Op,
			Namespace:        op.Namespace,
			Collection:       collection,
			MicrosecsRunning: op.MicrosecsRunning,
			MillisRunning:    float64(op.MicrosecsRunning) / 1000.0,
			Client:           op.Client,
		})
	}

	return &DashboardMetrics{
		Timestamp: time.Now(),
		Server: ServerMetrics{
			Host:      serverStatus.Host,
			Version:   serverStatus.Version,
			UptimeSec: serverStatus.Uptime,
		},
		Database: DatabaseMetrics{
			Name:           s.database,
			Collections:    dbStats.Collections,
			Documents:      dbStats.Objects,
			DataSizeMB:     bytesToMB(dbStats.DataSize),
			StorageSizeMB:  bytesToMB(dbStats.StorageSize),
			Indexes:        dbStats.Indexes,
			IndexSizeMB:    bytesToMB(dbStats.IndexSize),
			TotalDatabases: len(databases),
		},
		Connections: ConnectionStats{
			Current:      serverStatus.Connections.Current,
			Available:    serverStatus.Connections.Available,
			TotalCreated: serverStatus.Connections.TotalCreated,
		},
		Operations: OpCounters{
			Insert:  serverStatus.Opcounters.Insert,
			Query:   serverStatus.Opcounters.Query,
			Update:  serverStatus.Opcounters.Update,
			Delete:  serverStatus.Opcounters.Delete,
			Getmore: serverStatus.Opcounters.Getmore,
			Command: serverStatus.Opcounters.Command,
			Total:   totalOps,
		},
		Memory: MemoryStats{
			ResidentMB: serverStatus.Mem.Resident,
			VirtualMB:  serverStatus.Mem.Virtual,
		},
		Network: NetworkStats{
			BytesInMB:   bytesToMB(float64(serverStatus.Network.BytesIn)),
			BytesOutMB:  bytesToMB(float64(serverStatus.Network.BytesOut)),
			NumRequests: serverStatus.Network.NumRequests,
		},
		ActiveOps:       len(activeOps),
		CurrentOps:      currentOps,
		PaidSubscribers: paidSubs,
	}, nil
}
```

---

## Security Review

### Security Review for `GetDashboardMetrics` Function

#### Vulnerabilities Found:

**Info: Hardcoded Secrets or Credentials**
- **Line:** N/A
- **Severity:** Info
- **Fix:** Ensure no hardcoded secrets are present in the codebase. Use environment variables or secure vaults to manage sensitive information.

**Info: Insecure Deserialization (Not Applicable)**
- The function does not involve deserializing any data, so this is not applicable here.

#### Attack Vectors:

- If `s.database` contains user input, it could potentially be used for injection attacks. However, the code snippet does not show where `s.database` is set, and there are no direct SQL or command executions.
- The function handles database operations but does not expose any direct vulnerabilities related to SQL or command injection.

#### Recommended Fixes:

1. **Ensure Environment Variables:**
   - Replace hardcoded values like `s.database` with environment variables or configuration files that are securely managed.

2. **Input Validation (Not Applicable):**
   - Since there is no user input directly affecting the function, this check is not necessary for this specific code snippet.

3. **Error Handling:**
   - The error handling is robust and uses `fmt.Errorf` to wrap errors. Ensure that all potential errors are handled appropriately in other parts of your application.

#### Overall Security Posture:

The function appears to be well-structured with proper error handling. However, ensure that any user inputs or configuration values used within the function are securely managed. The current implementation does not introduce critical vulnerabilities but should still follow best practices for security, such as using environment variables for sensitive data and ensuring secure coding practices.

```go
// Example of replacing hardcoded secrets with environment variables
s.database = os.Getenv("DATABASE_NAME")
if s.database == "" {
    return nil, fmt.Errorf("database name is not set")
}
```

By following these recommendations, the overall security posture can be improved.

---

*Generated by CodeWorm on 2026-02-21 09:32*

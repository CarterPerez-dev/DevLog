# pipeline_test

**Type:** Code Evolution
**Repository:** Cybersecurity-Projects
**File:** PROJECTS/intermediate/secrets-scanner/internal/engine/pipeline_test.go
**Language:** go
**Lines:** 1-1
**Complexity:** 0.0

---

## Source Code

```go
Commit: 294169a2
Message: feat: Complete Go secrets scanner - Portia
Author: CarterPerez-dev
File: PROJECTS/intermediate/secrets-scanner/internal/engine/pipeline_test.go
Change type: new file

Diff:
@@ -0,0 +1,147 @@
+// ©AngelaMos | 2026
+// pipeline_test.go
+
+package engine
+
+import (
+	"context"
+	"os"
+	"path/filepath"
+	"testing"
+
+	"github.com/stretchr/testify/assert"
+	"github.com/stretchr/testify/require"
+
+	"github.com/CarterPerez-dev/portia/internal/rules"
+	"github.com/CarterPerez-dev/portia/internal/source"
+)
+
+func TestPipelineDirectoryScan(t *testing.T) {
+	t.Parallel()
+
+	dir := t.TempDir()
+	require.NoError(t, os.WriteFile( //nolint:gosec
+		filepath.Join(dir, "config.py"),
+		[]byte(`api_key = "AKIAIOSFODNN7EXAMPLE"`+"\n"),
+		0o644,
+	))
+	require.NoError(t, os.WriteFile( //nolint:gosec
+		filepath.Join(dir, "clean.go"),
+		[]byte("package main\n\nfunc main() {}\n"),
+		0o644,
+	))
+
+	reg := testRegistry()
+	p := NewPipeline(reg)
+	src := source.NewDirectory(dir, 0, nil)
+
+	result, err := p.Run(context.Background(), src)
+	require.NoError(t, err)
+	require.NotNil(t, result)
+	assert.GreaterOrEqual(t, len(result.Findings), 1)
+
+	found := false
+	for _, f := range result.Findings {
+		if f.RuleID == "test-aws-key" {
+			found = true
+			assert.Equal(t, "AKIAIOSFODNN7EXAMPLE", f.Secret)
+		}
+	}
+	assert.True(t, found, "expected to find AWS key")
+}
+
+func TestPipelineNoFindings(t *testing.T) {
+	t.Parallel()
+
+	dir := t.TempDir()
+	require.NoError(t, os.WriteFile( //nolint:gosec
+		filepath.Join(dir, "main.go"),
+		[]byte("package main\n\nfunc main() {}\n"),
+		0o644,
+	))
+
+	reg := testRegistry()
+	p := NewPipeline(reg)
+	src := source.NewDirectory(dir, 0, nil)
+
+	result, err := p.Run(context.Background(), src)
+	require.NoError(t, err)
+	assert.Empty(t, result.Findings)
+}
+
+func TestPipelineMultipleSecrets(t *testing.T) {
+	t.Parallel()
+
+	dir := t.TempDir()
+	require.NoError(t, os.WriteFile( //nolint:gosec
+		filepath.Join(dir, "secrets.env"),
+		[]byte(
+			`AWS_KEY=AKIAIOSFODNN7EXAMPLE`+"\n"+
+				`STRIPE_KEY=sk_live_4eC39HqLyjWDarjtT1zdp7dc`+"\n"+
+				`password = "xK9mP2vL5nQ8jR3t"`+"\n",
+		),
+		0o644,
+	))
+
+	reg := testRegistry()
+	p := NewPipeline(reg)
+	src := source.NewDirectory(dir, 0, nil)
+
+	result, err := p.Run(context.Background(), src)
+	require.NoError(t, err)
+	assert.GreaterOrEqual(t, len(result.Findings), 2)
+}
+
+func TestPipelineContextCancel(t *testing.T) {
+	t.Parallel()
+
+	dir := t.TempDir()
+	for i := range 20 {
+		name := "file" + string(rune('A'+i)) + ".txt"
+		require.NoError(t, os.WriteFile( //nolint:gosec
+			filepath.Join(dir, name),
+			[]byte(`AKIAIOSFODNN7EXAMPLE`+"\n"),
+			0o644,
+		))
+	}
+
+	reg := testRegistry()
+	p := NewPipeline(reg)
+	src := source.NewDirectory(dir, 0, nil)
+
+	ctx, cancel := context.WithCancel(context.Background())
+	cancel()
+
+	_, err := p.Run(ctx, src)
+	assert.Error(t, err)
+}
+
+func TestPipelineDedup(t *testing.T) {
+	t.Parallel()
+
+	reg :=
```

---

## Code Evolution

### Change Analysis

**What was Changed:**
A new file `pipeline_test.go` was added to the `internal/engine` package, containing four test functions (`TestPipelineDirectoryScan`, `TestPipelineNoFindings`, `TestPipelineMultipleSecrets`, and `TestPipelineContextCancel`) that cover various scenarios for a secrets scanner pipeline. Additionally, a test function `TestPipelineDedup` is included to ensure duplicate findings are handled correctly.

**Why it was Likely Changed:**
This change was likely made to enhance the testing coverage of the secrets scanner engine. The addition of these tests ensures that different use cases, such as directory scanning, handling no findings, multiple secret occurrences, and context cancellation, are properly validated.

**Impact on Behavior:**
These test functions will help ensure that the pipeline behaves correctly under various conditions. For instance, `TestPipelineDirectoryScan` checks if secrets are detected in a file, while `TestPipelineNoFindings` ensures that the pipeline handles directories with no secret-containing files gracefully.

**Risks or Concerns:**
- **Security Risks:** The tests use temporary files with sensitive data (`config.py`, `secrets.env`). While this is necessary for testing, it should be noted that in a real-world scenario, such files could pose security risks if not handled properly.
- **Performance Impact:** The `TestPipelineContextCancel` test creates 20 files, which might impact performance during the test run. This could be optimized by reducing the number of files or using a more efficient method to simulate multiple files.

Overall, this change significantly improves the robustness and reliability of the secrets scanner engine through comprehensive testing.

---

*Generated by CodeWorm on 2026-02-22 21:02*

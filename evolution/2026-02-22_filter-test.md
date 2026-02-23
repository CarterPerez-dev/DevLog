# filter_test

**Type:** Code Evolution
**Repository:** Cybersecurity-Projects
**File:** PROJECTS/intermediate/secrets-scanner/internal/engine/filter_test.go
**Language:** go
**Lines:** 1-1
**Complexity:** 0.0

---

## Source Code

```go
Commit: 294169a2
Message: feat: Complete Go secrets scanner - Portia
Author: CarterPerez-dev
File: PROJECTS/intermediate/secrets-scanner/internal/engine/filter_test.go
Change type: new file

Diff:
@@ -0,0 +1,352 @@
+// ©AngelaMos | 2026
+// filter_test.go
+
+package engine
+
+import (
+	"testing"
+
+	"github.com/stretchr/testify/assert"
+
+	"github.com/CarterPerez-dev/portia/internal/rules"
+	"github.com/CarterPerez-dev/portia/pkg/types"
+)
+
+func TestIsStopword(t *testing.T) {
+	t.Parallel()
+
+	tests := map[string]struct {
+		secret string
+		extra  []string
+		want   bool
+	}{
+		"exact stopword": {
+			secret: "cache",
+			want:   true,
+		},
+		"contains stopword substring": { //nolint:gosec
+			secret: "admin_cache_key_value",
+			want:   true,
+		},
+		"random high entropy secret": { //nolint:gosec
+			secret: "xK9mP2vL5nQ8jR3t",
+			want:   false,
+		},
+		"extra stopwords match": {
+			secret: "myCustomWord123",
+			extra:  []string{"custom"},
+			want:   true,
+		},
+		"short stopword ignored": {
+			secret: "abcXYZ",
+			extra:  []string{"ab"},
+			want:   false,
+		},
+		"real api key not stopped": { //nolint:gosec
+			secret: "AKIAIOSFODNN7EXAMPLE",
+			want:   false,
+		},
+	}
+
+	for name, tc := range tests {
+		t.Run(name, func(t *testing.T) {
+			t.Parallel()
+			got := IsStopword(tc.secret, tc.extra)
+			assert.Equal(t, tc.want, got)
+		})
+	}
+}
+
+func TestIsPlaceholder(t *testing.T) { //nolint:dupl
+	t.Parallel()
+
+	tests := map[string]struct {
+		secret string
+		want   bool
+	}{
+		"example key": {
+			secret: "EXAMPLE_KEY",
+			want:   true,
+		},
+		"your-api-key": {
+			secret: "your-api-key",
+			want:   true,
+		},
+		"placeholder": {
+			secret: "placeholder",
+			want:   true,
+		},
+		"xxxx pattern": {
+			secret: "xxxxxxxxxxxx",
+			want:   true,
+		},
+		"stars pattern": {
+			secret: "****",
+			want:   true,
+		},
+		"template var dollar": { //nolint:gosec
+			secret: "${SECRET_KEY}",
+			want:   true,
+		},
+		"template var braces": { //nolint:gosec
+			secret: "{{api_token}}",
+			want:   true,
+		},
+		"CHANGEME": {
+			secret: "CHANGEME",
+			want:   true,
+		},
+		"none": {
+			secret: "none",
+			want:   true,
+		},
+		"real secret not placeholder": { //nolint:gosec
+			secret: "sk_live_4eC39HqLyjWDarjtT1",
+			want:   false,
+		},
+		"real aws key not placeholder": { //nolint:gosec
+			secret: "AKIAIOSFODNN7ABCDEFG",
+			want:   false,
+		},
+	}
+
+	for name, tc := range tests {
+		t.Run(name, func(t *testing.T) {
+			t.Parallel()
+			got := IsPlaceholder(tc.secret)
+			assert.Equal(t, tc.want, got)
+		})
+	}
+}
+
+func TestIsTemplated(t *testing.T) { //nolint:dupl
+	t.Parallel()
+
+	tests := map[string]struct {
+		secret string
+		want   bool
+	}{
+		"dollar brace": {
+			secret: "${DB_PASSWORD}",
+			want:   true,
+		},
+		"double brace": { //nolint:gosec
+			secret: "{{api_key}}",
+			want:   true,
+		},
+		"os getenv": { //nolint:gosec
+			secret: `os.Getenv("SECRET_KEY")`,
+			want:   true,
+		},
+
```

---

## Code Evolution

### Change Analysis for `filter_test.go`

**What was Changed:**
A new file `filter_test.go` was added to the `internal/engine` package, containing several test functions (`TestIsStopword`, `TestIsPlaceholder`, `TestIsTemplated`, and `TestHasAssignmentOperator`). These tests validate various filtering criteria for secrets in a Go secrets scanner.

**Why it Was Likely Changed:**
This change likely aims to enhance the functionality of the secrets scanner by adding comprehensive testing for different types of secret patterns. The introduction of these test functions ensures that the scanner correctly identifies stopwords, placeholders, templated secrets, and assignments, thereby improving overall security.

**Impact on Behavior:**
These tests will now cover a broader range of scenarios when filtering secrets. For instance, `TestIsStopword` checks for exact matches as well as substrings within longer strings, while `TestIsPlaceholder` ensures that common placeholder patterns are correctly identified. This increases the robustness of the scanner's ability to filter out non-sensitive information.

**Risks or Concerns:**
While these tests improve coverage, there is a risk that overly aggressive filtering might inadvertently remove legitimate secrets. For example, using simple substring matching for `TestIsStopword` could lead to false positives if not carefully managed. Additionally, the use of `nolint:gosec` comments suggests potential security concerns in the test data, which should be reviewed and mitigated.

Overall, this change significantly enhances the testing framework but requires careful review to ensure it does not introduce new vulnerabilities or overly restrictive filtering rules.

---

*Generated by CodeWorm on 2026-02-22 20:52*

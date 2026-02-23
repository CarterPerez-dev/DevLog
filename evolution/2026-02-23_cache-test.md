# cache_test

**Type:** Code Evolution
**Repository:** angela
**File:** internal/pypi/cache_test.go
**Language:** go
**Lines:** 1-1
**Complexity:** 0.0

---

## Source Code

```go
Commit: ea4cc501
Message: Create canonical module source location - release v1.0.0
Author: CarterPerez-dev
File: internal/pypi/cache_test.go
Change type: new file

Diff:
@@ -0,0 +1,200 @@
+// ©AngelaMos | 2026
+// cache_test.go
+
+package pypi
+
+import (
+	"os"
+	"path/filepath"
+	"testing"
+	"time"
+)
+
+func TestCacheGetSetRoundTrip(t *testing.T) {
+	t.Parallel()
+
+	dir := t.TempDir()
+	cache, err := NewCache(dir, time.Hour)
+	if err != nil {
+		t.Fatalf("NewCache: %v", err)
+	}
+
+	entry := &CacheEntry{
+		ETag:     `"abc123"`,
+		Versions: []string{"1.0", "1.1", "2.0"},
+		CachedAt: time.Now(),
+	}
+
+	if err := cache.Set("requests", entry); err != nil {
+		t.Fatalf("Set: %v", err)
+	}
+
+	got, ok := cache.Get("requests")
+	if !ok {
+		t.Fatal("Get returned false after Set")
+	}
+	if got.ETag != entry.ETag {
+		t.Errorf("ETag = %q, want %q", got.ETag, entry.ETag)
+	}
+	if len(got.Versions) != len(entry.Versions) {
+		t.Fatalf(
+			"Versions len = %d, want %d",
+			len(got.Versions), len(entry.Versions),
+		)
+	}
+	for i, v := range got.Versions {
+		if v != entry.Versions[i] {
+			t.Errorf(
+				"Versions[%d] = %q, want %q",
+				i, v, entry.Versions[i],
+			)
+		}
+	}
+}
+
+func TestCacheGetMiss(t *testing.T) {
+	t.Parallel()
+
+	dir := t.TempDir()
+	cache, err := NewCache(dir, time.Hour)
+	if err != nil {
+		t.Fatalf("NewCache: %v", err)
+	}
+
+	_, ok := cache.Get("nonexistent")
+	if ok {
+		t.Fatal("Get returned true for missing key")
+	}
+}
+
+func TestCacheIsFresh(t *testing.T) {
+	t.Parallel()
+
+	dir := t.TempDir()
+	cache, err := NewCache(dir, time.Hour)
+	if err != nil {
+		t.Fatalf("NewCache: %v", err)
+	}
+
+	fresh := &CacheEntry{CachedAt: time.Now()}
+	if !cache.IsFresh(fresh) {
+		t.Error("entry cached just now should be fresh")
+	}
+
+	stale := &CacheEntry{
+		CachedAt: time.Now().Add(-2 * time.Hour),
+	}
+	if cache.IsFresh(stale) {
+		t.Error("entry cached 2 hours ago should be stale")
+	}
+}
+
+func TestCacheTouch(t *testing.T) {
+	t.Parallel()
+
+	dir := t.TempDir()
+	cache, err := NewCache(dir, time.Hour)
+	if err != nil {
+		t.Fatalf("NewCache: %v", err)
+	}
+
+	old := time.Now().Add(-30 * time.Minute)
+	entry := &CacheEntry{
+		ETag:     `"xyz"`,
+		Versions: []string{"1.0"},
+		CachedAt: old,
+	}
+	if err := cache.Set("flask", entry); err != nil {
+		t.Fatalf("Set: %v", err)
+	}
+
+	cache.Touch("flask")
+
+	got, ok := cache.Get("flask")
+	if !ok {
+		t.Fatal("Get returned false after Touch")
+	}
+	if !got.CachedAt.After(old) {
+		t.Error("Touch did not update CachedAt")
+	}
+}
+
+func TestCacheClear(t *testing.T) {
+	t.Parallel()
+
+	dir := t.TempDir()
+	cache, err := NewCache(dir, time.Hour)
+	if err != nil {
+		t.Fatalf("NewCache: %v", err)
+	}
+
+	for _, name := range []string{"a", "b", "c"} {
+		setErr := cache.Set(name, &CacheEntry{
+			Versions: []string{"1.0"},
+			CachedAt: time.Now(),
+		})
+		if setErr != nil {
+			t.Fatalf("Set %s: %v", name, setErr)
+		}
+	}
+
+	if clearErr := cache.Clear(); clearErr != nil 
```

---

## Code Evolution

### Change Analysis

**What was Changed:**
A new file `cache_test.go` was added to the `internal/pypi` package, containing multiple test functions for a cache implementation. The tests cover various scenarios such as setting and getting entries, checking freshness, touching an entry, clearing the cache, and handling path traversal attempts.

**Why it Was Likely Changed:**
This change likely aims to ensure that the cache implementation is robust and behaves correctly under different conditions. By adding comprehensive test cases, the developers can verify that the cache handles various operations as expected, including edge cases like path traversals.

**Impact on Behavior:**
The tests provide a safety net for future changes to the cache implementation, ensuring that it maintains its intended functionality. Specifically:
- `TestCacheGetSetRoundTrip` verifies basic set and get operations.
- `TestCacheGetMiss` ensures that missing keys are handled correctly.
- `TestCacheIsFresh` checks the freshness logic.
- `TestCacheTouch` confirms that touching an entry updates its timestamp.
- `TestCacheClear` ensures that clearing the cache removes all entries properly.
- `TestCachePathTraversal` prevents path traversal attacks by validating safe and unsafe paths.

**Risks or Concerns:**
The primary risk is ensuring that the tests accurately reflect the intended behavior of the cache. For instance, the test for path traversal should strictly prevent any file creation outside the specified directory to avoid security vulnerabilities. Additionally, the tests should be run regularly as part of the CI/CD pipeline to catch regressions early.

Overall, this change significantly enhances the reliability and security of the cache implementation by providing thorough testing coverage.

---

*Generated by CodeWorm on 2026-02-23 13:54*

# backup_repo

**Type:** File Overview
**Repository:** oneIsNun_
**File:** go-backend/internal/sqlite/backup_repo.go
**Language:** go
**Lines:** 1-150
**Complexity:** 0.0

---

## Source Code

```go
/*
AngelaMos | 2026
backup_repo.go
*/

package sqlite

import (
	"context"
	"database/sql"
	"fmt"
	"time"
)

type BackupRepository struct {
	db *sql.DB
}

func NewBackupRepository(client *Client) *BackupRepository {
	return &BackupRepository{db: client.DB()}
}

type Backup struct {
	ID           string
	DatabaseName string
	FilePath     string
	SizeBytes    int64
	StartedAt    time.Time
	CompletedAt  sql.NullTime
	Status       string
	ErrorMessage sql.NullString
	TriggeredBy  string
}

func (r *BackupRepository) Create(ctx context.Context, b *Backup) error {
	query := `
		INSERT INTO backups (id, database_name, file_path, size_bytes, started_at, status, triggered_by)
		VALUES (?, ?, ?, ?, ?, ?, ?)`

	_, err := r.db.ExecContext(ctx, query,
		b.ID,
		b.DatabaseName,
		b.FilePath,
		b.SizeBytes,
		b.StartedAt,
		b.Status,
		b.TriggeredBy,
	)
	if err != nil {
		return fmt.Errorf("insert backup: %w", err)
	}
	return nil
}

func (r *BackupRepository) UpdateStatus(ctx context.Context, id, status, filePath string, sizeBytes int64, errorMsg string) error {
	query := `
		UPDATE backups
		SET status = ?, file_path = ?, size_bytes = ?, completed_at = ?, error_message = ?
		WHERE id = ?`

	completedAt := sql.NullTime{Time: time.Now(), Valid: true}
	errMsgNull := sql.NullString{String: errorMsg, Valid: errorMsg != ""}

	_, err := r.db.ExecContext(ctx, query, status, filePath, sizeBytes, completedAt, errMsgNull, id)
	if err != nil {
		return fmt.Errorf("update backup status: %w", err)
	}
	return nil
}

func (r *BackupRepository) GetByID(ctx context.Context, id string) (*Backup, error) {
	query := `
		SELECT id, database_name, file_path, size_bytes, started_at, completed_at, status, error_message, triggered_by
		FROM backups
		WHERE id = ?`

	var b Backup
	err := r.db.QueryRowContext(ctx, query, id).Scan(
		&b.ID,
		&b.DatabaseName,
		&b.FilePath,
		&b.SizeBytes,
		&b.StartedAt,
		&b.CompletedAt,
		&b.Status,
		&b.ErrorMessage,
		&b.TriggeredBy,
	)
	if err == sql.ErrNoRows {
		return nil, nil
	}
	if err != nil {
		return nil, fmt.Errorf("get backup by id: %w", err)
	}
	return &b, nil
}

func (r *BackupRepository) ListRecent(ctx context.Context, limit int) ([]*Backup, error) {
	query := `
		SELECT id, database_name, file_path, size_bytes, started_at, completed_at, status, error_message, triggered_by
		FROM backups
		ORDER BY started_at DESC
		LIMIT ?`

	rows, err := r.db.QueryContext(ctx, query, limit)
	if err != nil {
		return nil, fmt.Errorf("list backups: %w", err)
	}
	defer rows.Close()

	var backups []*Backup
	for rows.Next() {
		var b Backup
		err := rows.Scan(
			&b.ID,
			&b.DatabaseName,
			&b.FilePath,
			&b.SizeBytes,
			&b.StartedAt,
			&b.CompletedAt,
			&b.Status,
			&b.ErrorMessage,
			&b.TriggeredBy,
		)
		if err != nil {
			return nil, fmt.Errorf("scan backup: %w", err)
		}
		backups = append(backups, &b)
	}
	return backups, nil
}

func (r *BackupRepository) Delete(ctx context.Context, id string) error {
	query := `DELETE FROM backups WHERE id = 
```

---

## File Overview

# backup_repo.go

## Purpose and Responsibility
This Go source file defines a repository for managing database backups using SQLite. It provides methods to create, update, retrieve, list, and delete backup records.

## Key Exports and Public Interface
- `BackupRepository`: Manages backup operations.
  - `Create(ctx context.Context, b *Backup) error`: Inserts a new backup record.
  - `UpdateStatus(ctx context.Context, id, status, filePath string, sizeBytes int64, errorMsg string) error`: Updates the status of an existing backup.
  - `GetByID(ctx context.Context, id string) (*Backup, error)`: Retrieves a backup by its ID.
  - `ListRecent(ctx context.Context, limit int) ([]*Backup, error)`: Lists recent backups.
  - `Delete(ctx context.Context, id string) error`: Deletes a backup by its ID.
  - `DeleteOlderThan(ctx context.Context, days int) (int64, error)`: Deletes backups older than the specified number of days.

## How It Fits in the Project
This file is part of the `sqlite` package within the `oneIsNun_` repository. It interacts with SQLite to manage backup records, which are crucial for maintaining data integrity and recovery capabilities. The methods provided allow other parts of the application to interact with the backup system seamlessly.

## Notable Design Decisions
- **Error Handling**: All database operations return errors wrapped in `fmt.Errorf`, ensuring consistent error handling.
- **Context Usage**: Utilizes `context.Context` for managing lifecycle and cancellation, enhancing robustness.
- **SQL Injection Prevention**: Uses parameterized queries to prevent SQL injection.
- **Null Values**: Properly handles null values using `sql.NullTime` and `sql.NullString`.
- **Goroutines**: While not explicitly used in this file, the design allows for integration with goroutines for asynchronous operations if needed.
```

This documentation provides a high-level overview of the file's purpose, key exports, its role within the project, and notable design decisions.

---

*Generated by CodeWorm on 2026-02-19 23:57*

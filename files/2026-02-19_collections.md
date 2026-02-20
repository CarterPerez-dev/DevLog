# collections

**Type:** File Overview
**Repository:** oneIsNun_
**File:** go-backend/internal/handler/collections.go
**Language:** go
**Lines:** 1-243
**Complexity:** 0.0

---

## Source Code

```go
/*
AngelaMos | 2026
collections.go
*/

package handler

import (
	"context"
	"net/http"
	"strconv"

	"github.com/go-chi/chi/v5"
	"go.mongodb.org/mongo-driver/v2/bson"

	"github.com/carterperez-dev/templates/go-backend/internal/core"
	"github.com/carterperez-dev/templates/go-backend/internal/mongodb"
)

type collectionsRepository interface {
	ListCollections(ctx context.Context, dbName string) ([]mongodb.CollectionInfo, error)
	GetCollectionStats(ctx context.Context, dbName, collName string) (*mongodb.CollectionStats, error)
	AnalyzeSchema(ctx context.Context, dbName, collName string, sampleSize int) (*mongodb.SchemaAnalysis, error)
	GetIndexes(ctx context.Context, dbName, collName string) ([]mongodb.IndexInfo, error)
	SampleDocuments(ctx context.Context, dbName, collName string, limit int) ([]bson.M, error)
	GetFieldStats(ctx context.Context, dbName, collName, fieldName string) (*mongodb.FieldStats, error)
	CountByFieldValue(ctx context.Context, dbName, collName, fieldName string, value any) (int64, error)
}

type CollectionsHandler struct {
	repo     collectionsRepository
	database string
}

func NewCollectionsHandler(repo collectionsRepository, database string) *CollectionsHandler {
	return &CollectionsHandler{
		repo:     repo,
		database: database,
	}
}

func (h *CollectionsHandler) RegisterRoutes(r chi.Router) {
	r.Route("/api/collections", func(r chi.Router) {
		r.Get("/", h.List)
		r.Get("/{name}", h.GetStats)
		r.Get("/{name}/schema", h.GetSchema)
		r.Get("/{name}/indexes", h.GetIndexes)
		r.Get("/{name}/documents", h.SampleDocuments)
		r.Get("/{name}/fields/{field}", h.GetFieldStats)
		r.Get("/{name}/count", h.CountByField)
	})
}

type CollectionsListResponse struct {
	Database    string                    `json:"database"`
	Count       int                       `json:"count"`
	Collections []mongodb.CollectionInfo  `json:"collections"`
}

func (h *CollectionsHandler) List(w http.ResponseWriter, r *http.Request) {
	dbName := r.URL.Query().Get("database")
	if dbName == "" {
		dbName = h.database
	}

	collections, err := h.repo.ListCollections(r.Context(), dbName)
	if err != nil {
		core.InternalServerError(w, err)
		return
	}

	response := CollectionsListResponse{
		Database:    dbName,
		Count:       len(collections),
		Collections: collections,
	}

	core.OK(w, response)
}

func (h *CollectionsHandler) GetStats(w http.ResponseWriter, r *http.Request) {
	name := chi.URLParam(r, "name")
	dbName := r.URL.Query().Get("database")
	if dbName == "" {
		dbName = h.database
	}

	stats, err := h.repo.GetCollectionStats(r.Context(), dbName, name)
	if err != nil {
		core.InternalServerError(w, err)
		return
	}

	core.OK(w, stats)
}

func (h *CollectionsHandler) GetSchema(w http.ResponseWriter, r *http.Request) {
	name := chi.URLParam(r, "name")
	dbName := r.URL.Query().Get("database")
	if dbName == "" {
		dbName = h.database
	}

	sampleSize := 1000
	if s := r.URL.Query().Get("sample_size"); s != "" {
		if parsed, err := strconv.Atoi(s); err == nil &&
```

---

## File Overview

# `collections.go` Documentation

## Purpose and Responsibility
This file defines a handler for managing MongoDB collections within the backend application. It provides endpoints to list collections, retrieve collection statistics, analyze schema, get indexes, sample documents, and count field values.

## Key Exports and Public Interface
- **CollectionsHandler**: A struct that encapsulates the logic for handling requests related to MongoDB collections.
- **RegisterRoutes**: Registers routes for various operations on collections.
- **List**, **GetStats**, **GetSchema**, **GetIndexes**, **SampleDocuments**, and **CountByField**: Functions that handle specific HTTP GET requests.

## How It Fits into the Project
This file is part of the `handler` package, which contains business logic for handling API requests. The `CollectionsHandler` struct interacts with a repository (`collectionsRepository`) to perform operations on MongoDB collections. These operations are exposed via RESTful endpoints, allowing external clients to interact with the database.

## Notable Design Decisions
- **Dependency Injection**: The `CollectionsHandler` constructor accepts an instance of `collectionsRepository`, promoting loose coupling.
- **Error Handling**: All functions use a consistent error handling pattern by calling `core.InternalServerError` for non-nil errors, ensuring proper response codes and messages are returned to the client.
- **Query Parameters**: Functions like `GetStats` and `SampleDocuments` handle query parameters to provide flexible filtering options.
- **Context Usage**: Context is used throughout to propagate request-specific values such as database names and limits.
```

This documentation provides a high-level overview of the file's purpose, key components, integration within the project, and design choices.

---

*Generated by CodeWorm on 2026-02-19 23:27*

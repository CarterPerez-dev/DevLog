# collections

**Type:** File Overview
**Repository:** oneIsNun_
**File:** go-backend/internal/mongodb/collections.go
**Language:** go
**Lines:** 1-582
**Complexity:** 0.0

---

## Source Code

```go
/*
AngelaMos | 2026
collections.go
*/

package mongodb

import (
	"context"
	"fmt"
	"sort"

	"go.mongodb.org/mongo-driver/v2/bson"
	"go.mongodb.org/mongo-driver/v2/mongo"
	"go.mongodb.org/mongo-driver/v2/mongo/options"
)

type CollectionsRepository struct {
	client *Client
}

func NewCollectionsRepository(client *Client) *CollectionsRepository {
	return &CollectionsRepository{client: client}
}

type CollectionInfo struct {
	Name         string `json:"name"`
	Type         string `json:"type"`
	DocumentCount int64  `json:"document_count"`
	SizeBytes    int64  `json:"size_bytes"`
	AvgDocSize   int64  `json:"avg_doc_size"`
	IndexCount   int    `json:"index_count"`
}

type CollectionStats struct {
	Name          string  `json:"name"`
	DocumentCount int64   `json:"document_count"`
	SizeBytes     int64   `json:"size_bytes"`
	AvgDocSize    int64   `json:"avg_doc_size"`
	StorageSize   int64   `json:"storage_size"`
	IndexCount    int     `json:"index_count"`
	TotalIndexSize int64  `json:"total_index_size"`
	Capped        bool    `json:"capped"`
}

type FieldSchema struct {
	Name       string   `json:"name"`
	Types      []string `json:"types"`
	Coverage   float64  `json:"coverage"`
	Count      int64    `json:"count"`
	TotalDocs  int64    `json:"total_docs"`
	SampleValues []any  `json:"sample_values,omitempty"`
}

type SchemaAnalysis struct {
	CollectionName string        `json:"collection_name"`
	TotalDocuments int64         `json:"total_documents"`
	SampleSize     int64         `json:"sample_size"`
	Fields         []FieldSchema `json:"fields"`
}

type IndexInfo struct {
	Name       string         `json:"name"`
	Keys       map[string]int `json:"keys"`
	Unique     bool           `json:"unique"`
	Sparse     bool           `json:"sparse"`
	Background bool           `json:"background"`
	SizeBytes  int64          `json:"size_bytes"`
}

type FieldStats struct {
	FieldName    string         `json:"field_name"`
	TotalDocs    int64          `json:"total_docs"`
	DocsWithField int64         `json:"docs_with_field"`
	Coverage     float64        `json:"coverage"`
	UniqueValues int64          `json:"unique_values"`
	TopValues    []ValueCount   `json:"top_values,omitempty"`
	NumericStats *NumericStats  `json:"numeric_stats,omitempty"`
}

type ValueCount struct {
	Value any   `json:"value"`
	Count int64 `json:"count"`
}

type NumericStats struct {
	Min float64 `json:"min"`
	Max float64 `json:"max"`
	Avg float64 `json:"avg"`
	Sum float64 `json:"sum"`
}

func (r *CollectionsRepository) ListCollections(ctx context.Context, dbName string) ([]CollectionInfo, error) {
	db := r.client.client.Database(dbName)

	cursor, err := db.ListCollections(ctx, bson.D{})
	if err != nil {
		return nil, fmt.Errorf("list collections: %w", err)
	}
	defer cursor.Close(ctx)

	var collections []CollectionInfo
	for cursor.Next(ctx) {
		var result bson.M
		if err := cursor.Decode(&result); err != nil {
			continue
		}

		name, _ := result["name"].(string)
		collType, _ := result["type"].(string)

		if
```

---

## File Overview

# collections.go

## Purpose and Responsibility
This Go source file defines a repository for managing MongoDB collections within the `oneIsNun_` project. It provides methods to list collection information, retrieve detailed statistics about specific collections, and analyze field schemas.

## Key Exports and Public Interface
- **CollectionsRepository**: A struct responsible for interacting with MongoDB collections.
  - `NewCollectionsRepository`: Constructor function that initializes a new instance of `CollectionsRepository`.
- **CollectionInfo**: Struct representing basic information about a collection.
- **CollectionStats**: Struct containing detailed statistics about a specific collection.
- **FieldSchema**: Struct describing the schema and coverage of fields within a collection.
- **SchemaAnalysis**: Struct for analyzing field schemas across multiple documents.
- **IndexInfo**: Struct detailing index information for collections.
- **FieldStats**: Struct providing statistical analysis on individual fields.

## How It Fits in the Project
This file is part of the `mongodb` package, which handles database operations. The `CollectionsRepository` provides essential functionality to manage and analyze MongoDB collections, making it a crucial component for data management within the project.

## Notable Design Decisions
- **Error Handling**: Uses Go's idiomatic error handling with `fmt.Errorf` to provide clear error messages.
- **Goroutines and Contexts**: While not explicitly used in this file, goroutines and context are common patterns in the broader project for asynchronous operations and graceful shutdowns.
- **BSON Decoding**: Utilizes MongoDB's `bson` package for efficient data decoding from JSON-like documents.
- **Sorting**: Implements sorting of collections based on document count to provide a meaningful order in results.

This file ensures robust management and analysis of MongoDB collections, contributing to the overall data handling capabilities of the project.

---

*Generated by CodeWorm on 2026-02-19 23:21*

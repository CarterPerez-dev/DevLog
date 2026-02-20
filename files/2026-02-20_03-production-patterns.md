# 03_production_patterns

**Type:** File Overview
**Repository:** fastapi-rc
**File:** fastapi-rc/examples/03_production_patterns.py
**Language:** python
**Lines:** 1-291
**Complexity:** 0.0

---

## Source Code

```python
"""
ⒸAngelaMos | 2025
03_production_patterns.py

Caching patterns for enterprise applications
"""

import json
import uvicorn
from typing import Annotated
from datetime import datetime
from contextlib import asynccontextmanager

from fastapi import (
    FastAPI,
    Depends,
    Query,
    status,
)
from pydantic import BaseModel

from fastapi_rc import (
    cachemanager,
    RedisClient,
    CacheService,
    get_ttl_with_jitter,
)


class Product(BaseModel):
    """
    Product model
    """
    id: str
    name: str
    category: str
    price: float
    in_stock: bool
    created_at: datetime


class ProductList(BaseModel):
    """
    Paginated product list
    """
    items: list[Product]
    total: int
    page: int
    per_page: int


@asynccontextmanager
async def lifespan(app: FastAPI):
    """
    Production lifespan with full configuration
    """
    cachemanager.init(
        redis_url = "redis://localhost:6379/0",
        max_connections = 100,
        socket_timeout = 5.0,
        socket_connect_timeout = 2.0,
        health_check_interval = 30,
        decode_responses = True,
    )
    yield
    await cachemanager.close()


app = FastAPI(
    title = "Product API with Redis Caching",
    lifespan = lifespan,
)


async def get_product_cache(redis: RedisClient) -> CacheService[Product]:
    """
    Product cache with 15 minute TTL
    """
    return CacheService(
        redis,
        namespace = "products",
        model = Product,
        default_ttl = 900,
        use_jitter = True,
        prefix = "api",
        version = "v1",
    )


ProductCache = Annotated[CacheService[Product], Depends(get_product_cache)]


async def fetch_product_from_db(product_id: str) -> Product:
    """
    Simulate expensive database query
    """
    return Product(
        id = product_id,
        name = f"Product {product_id}",
        category = "electronics",
        price = 299.99,
        in_stock = True,
        created_at = datetime.now(),
    )


async def fetch_products_from_db(
    category: str | None,
    page: int,
    per_page: int,
) -> ProductList:
    """
    Simulate expensive database query with filters
    """
    products = [
        Product(
            id = str(i),
            name = f"{category or 'General'} Product {i}",
            category = category or "general",
            price = 99.99 * i,
            in_stock = i % 2 == 0,
            created_at = datetime.now(),
        ) for i in range((page - 1) * per_page, page * per_page)
    ]

    return ProductList(
        items = products,
        total = 100,
        page = page,
        per_page = per_page,
    )


@app.get("/products/{product_id}", response_model = Product)
async def get_product(product_id: str, product_cache: ProductCache):
    """
    Pattern 1: Simple cache-aside with CacheService
    """
    product = await product_cache.get_or_set(
        identifier = product_id,
        factory = lambda: fetch_product_from_db(product_id),
    )

    return p
```

---

## File Overview

### 03_production_patterns.py

**Purpose and Responsibility:**
This file implements caching patterns for an enterprise application using Redis, focusing on production-ready scenarios such as cache-aside, query parameter caching, batch caching, and granular cache invalidation.

**Key Exports or Public Interface:**
- `Product` and `ProductList` models
- `lifespan` context manager for initializing and closing the cache manager
- Dependency injection for `CacheService[Product]`
- API endpoints:
  - `/products/{product_id}`: Fetches a single product with cache support.
  - `/products`: Lists products with query parameter caching.
  - `/products/batch`: Creates multiple products and caches them.
  - `/products/{product_id}/stock`: Updates stock status for a specific product.

**How it Fits in the Project:**
This file is part of the `fastapi-rc` repository, demonstrating best practices for integrating Redis caching with FastAPI. It integrates seamlessly into the project by providing reusable cache services and patterns that can be applied across different parts of the application.

**Notable Design Decisions:**
1. **CacheManager Initialization**: Uses a context manager to initialize the cache manager with full configuration settings.
2. **Dependency Injection**: Utilizes `Depends` for injecting `CacheService[Product]`, ensuring consistent caching behavior throughout the API.
3. **TTL and Jitter**: Implements Time-To-Live (TTL) with jitter to avoid cache stampedes, enhancing performance and reliability.
4. **Redis Pipeline**: For batch operations, uses Redis pipelines to efficiently handle multiple set commands in a single network round trip.
5. **Granular Cache Invalidation**: Manually invalidates specific cache keys when updating product stock status, ensuring data consistency.
```

This documentation provides an overview of the file's purpose, key components, integration within the project, and significant design choices.

---

*Generated by CodeWorm on 2026-02-20 01:23*

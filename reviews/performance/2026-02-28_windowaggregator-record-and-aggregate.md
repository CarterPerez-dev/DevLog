# WindowAggregator.record_and_aggregate

**Type:** Performance Analysis
**Repository:** Cybersecurity-Projects
**File:** PROJECTS/advanced/ai-threat-detection/backend/app/core/features/aggregator.py
**Language:** python
**Lines:** 33-121
**Complexity:** 3.0

---

## Source Code

```python
async def record_and_aggregate(
        self,
        ip: str,
        request_id: str,
        path: str,
        path_depth: int,
        method: str,
        status_code: int,
        user_agent: str,
        response_size: int,
        timestamp: float,
    ) -> dict[str, float]:
        """
        Record a request into Redis sorted sets and return all 12
        per-IP windowed features.
        """
        prefix = f"ip:{ip}"
        keys = {
            "requests": f"{prefix}:requests",
            "paths": f"{prefix}:paths",
            "statuses": f"{prefix}:statuses",
            "uas": f"{prefix}:uas",
            "sizes": f"{prefix}:sizes",
            "methods": f"{prefix}:methods",
            "depths": f"{prefix}:depths",
        }

        trim_boundary = timestamp - KEY_TTL
        w1m = timestamp - WINDOW_1M
        w5m = timestamp - WINDOW_5M
        w10m = timestamp - WINDOW_10M

        pipe = self._redis.pipeline()

        pipe.zadd(keys["requests"], {request_id: timestamp})
        pipe.zadd(keys["paths"], {_hash_member(path): timestamp})
        pipe.zadd(keys["statuses"], {f"{status_code}:{request_id}": timestamp})
        pipe.zadd(keys["uas"], {_hash_member(user_agent): timestamp})
        pipe.zadd(keys["sizes"], {f"{response_size}:{request_id}": timestamp})
        pipe.zadd(keys["methods"], {f"{method}:{request_id}": timestamp})
        pipe.zadd(keys["depths"], {f"{path_depth}:{request_id}": timestamp})

        for key in keys.values():
            pipe.zremrangebyscore(key, "-inf", trim_boundary)

        pipe.zcount(keys["requests"], w1m, "+inf")
        pipe.zcount(keys["requests"], w5m, "+inf")
        pipe.zcount(keys["requests"], w10m, "+inf")
        pipe.zcount(keys["paths"], w5m, "+inf")
        pipe.zcount(keys["uas"], w10m, "+inf")
        pipe.zrangebyscore(keys["statuses"], w5m, "+inf")
        pipe.zrangebyscore(keys["sizes"], w5m, "+inf")
        pipe.zrangebyscore(keys["methods"], w5m, "+inf")
        pipe.zrangebyscore(keys["depths"], w5m, "+inf")
        pipe.zrangebyscore(keys["requests"], w10m, "+inf", withscores=True)

        for key in keys.values():
            pipe.expire(key, KEY_TTL)

        results = await pipe.execute()

        read_start = len(keys) * 2
        req_count_1m = results[read_start]
        req_count_5m = results[read_start + 1]
        req_count_10m = results[read_start + 2]
        unique_paths_5m = results[read_start + 3]
        unique_uas_10m = results[read_start + 4]
        statuses_5m = results[read_start + 5]
        sizes_5m = results[read_start + 6]
        methods_5m = results[read_start + 7]
        depths_5m = results[read_start + 8]
        requests_with_scores = results[read_start + 9]

        irt_mean, irt_std = _inter_request_time_stats(requests_with_scores)

        return {
            "req_count_1m": float(req_count_1m),
            "req_count_5m": float(req_count_5m),
            "req_count_10m": float(req_count_10m),
            "error_rate_5m":
```

---

## Performance Analysis

### Performance Analysis

**Time Complexity:** The primary bottleneck is the Redis operations, which are executed in a pipeline but still involve multiple `zadd`, `zremrangebyscore`, and `zcount` calls. Each of these operations has a time complexity of \(O(\log N + M)\), where \(N\) is the number of elements in the sorted set and \(M\) is the number of elements to be added or removed. The overall time complexity is dominated by these Redis operations, making it approximately \(O(M \cdot \log N)\).

**Space Complexity:** The space complexity is primarily determined by the size of the data stored in Redis and the intermediate variables used in the function. The use of a pipeline helps minimize network overhead but does not reduce the memory footprint.

**Bottlenecks or Inefficiencies:**
- **Redundant Operations:** The `zadd` operations are repeated for each key, which is unnecessary since they can be batched into one call per key.
- **Multiple ZCOUNT Calls:** There are multiple `zcount` calls with overlapping time windows, leading to redundant computations.

**Optimization Opportunities:**
- **Batch Redis Commands:** Combine the `zadd` commands for each key into a single pipeline command to reduce network overhead and improve efficiency.
- **Reduce Redundant Operations:** Use a single `zcount` call per window instead of multiple calls with overlapping time windows. For example, use `zcount` once for the 10-minute window and infer other counts based on this.

**Resource Usage Concerns:**
- Ensure that Redis connections are properly managed using context managers or connection pools to avoid resource leaks.
- Consider caching intermediate results if they are reused frequently to reduce redundant computations.

---

*Generated by CodeWorm on 2026-02-28 07:32*

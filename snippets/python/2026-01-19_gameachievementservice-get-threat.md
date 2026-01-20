# GameAchievementService.get_threat_achievements

**Repository:** CertGames-Core
**File:** backend/api/domains/progression/services/game_achievement_ops.py
**Language:** python
**Lines:** 93-192
**Complexity:** 12.0

---

## Source Code

```python
def get_threat_achievements(user: User) -> list[dict[str, Any]]:
        """
        Get all threat hunter game achievements earned by user
        """
        achievements: list[dict[str, Any]] = []

        completed_analyses = LogAnalysis.objects(
            userId = user.id,
            completed = True
        )

        if not completed_analyses:
            return achievements

        first_analysis = min(
            completed_analyses,
            key = lambda x: x.completedAt
        )
        achievements.append(
            {
                "id": "first_threat",
                "name": "First Threat",
                "description":
                "Completed your first threat hunting scenario",
                "gameType": "threat",
                "earnedAt": first_analysis.completedAt.isoformat(),
                "category": "milestone"
            }
        )

        for analysis in completed_analyses:
            if analysis.score >= 90:
                achievements.append(
                    {
                        "id":
                        f"master_threat_hunter_{analysis.scenarioId}",
                        "name": "Master Threat Hunter",
                        "description":
                        f"Scored {analysis.score} in threat hunting",
                        "gameType": "threat",
                        "earnedAt": analysis.completedAt.isoformat(),
                        "category": "performance",
                        "scenarioId": analysis.scenarioId
                    }
                )
            elif analysis.score >= 75:
                achievements.append(
                    {
                        "id": f"senior_analyst_{analysis.scenarioId}",
                        "name": "Senior Analyst",
                        "description":
                        f"Scored {analysis.score} in threat hunting",
                        "gameType": "threat",
                        "earnedAt": analysis.completedAt.isoformat(),
                        "category": "performance",
                        "scenarioId": analysis.scenarioId
                    }
                )

        perfect_analyses = [
            a for a in completed_analyses
            if hasattr(a, 'missedThreats') and a.missedThreats == 0
            and hasattr(a, 'falsePositives') and a.falsePositives == 0
        ]

        for analysis in perfect_analyses:
            achievements.append(
                {
                    "id": f"perfect_detection_{analysis.scenarioId}",
                    "name": "Perfect Detection",
                    "description":
                    "Identified all threats with zero false positives",
                    "gameType": "threat",
                    "earnedAt": analysis.completedAt.isoformat(),
                    "category": "mastery",
                    "scenarioId": analysis.scenarioId
                }
            )

        if len(completed_analyses) >= 10:
            achievements.append(
                {
                    "id":
                    "threat_veteran",
                    "name":
                    "Threat Hunting Veteran",
                    "description":
                    f"Completed {len(completed_analyses)} threat hunting scenarios",
                    "gameType":
                    "threat",
                    "earnedAt":
                    max(completed_analyses,
                        key = lambda x: x.completedAt
                        ).completedAt.isoformat(),
                    "category":
                    "mastery"
                }
            )

        return achievements
```

---

## Documentation

### Documentation for `get_threat_achievements`

**Purpose and Behavior**: 
The function retrieves all achievements earned by a user in the threat hunting game. It first filters completed analyses, then appends specific achievements based on various criteria such as the first analysis, high scores, perfect detections, and completing multiple scenarios.

**Key Implementation Details**:
- **Type Hints**: The function uses type hints to specify that `user` is of type `User`, and returns a list of dictionaries representing achievements.
- **Query Filtering**: It filters completed analyses using Django ORM queries.
- **Achievement Logic**: Achievements are added based on conditions like score thresholds, perfect detections, and scenario completions.

**When/Why to Use This Code**:
Use this function in scenarios where you need to dynamically generate a user's achievements in the threat hunting game. It is particularly useful for displaying progress or rewards within the game interface.

**Patterns/Gotchas**:
- The code uses list comprehensions and lambda functions, which can be complex but are efficient.
- Ensure that `LogAnalysis` model fields like `completedAt`, `score`, and `missedThreats` exist to avoid runtime errors.
- The function assumes the presence of `User` and `LogAnalysis` models; ensure these are correctly defined in your project.

---

*Generated by CodeWorm on 2026-01-19 20:40*

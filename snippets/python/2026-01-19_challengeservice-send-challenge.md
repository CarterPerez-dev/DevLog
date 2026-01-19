# ChallengeService.send_challenge

**Repository:** CertGames-Core
**File:** backend/api/domains/social/services/challenge_ops.py
**Language:** python
**Lines:** 30-141
**Complexity:** 9.0

---

## Source Code

```python
def send_challenge(
        challenger_id: str | ObjectId,
        challenged_id: str | ObjectId,
        test_category: str,
        test_id: int | None,
        question_count: int,
        mode: str
    ) -> Challenge:
        """
        Create a new challenge
        If test_id is None, randomly select a test from the category
        Validates test exists in database before creating challenge
        """
        challenger_id = ObjectId(challenger_id) if isinstance(
            challenger_id,
            str
        ) else challenger_id
        challenged_id = ObjectId(challenged_id) if isinstance(
            challenged_id,
            str
        ) else challenged_id

        if test_id is None:
            available_tests = list(
                Test.objects(category = test_category).only('testId')
            )

            if not available_tests:
                raise NotFoundError(
                    "Test",
                    f"No tests found for category '{test_category}'"
                )

            random_test = random.choice(available_tests)
            test_id = random_test.testId
        else:
            test = Test.objects(
                testId = test_id,
                category = test_category
            ).first()

            if not test:
                raise NotFoundError(
                    "Test",
                    f"Test {test_id} not found in category '{test_category}'"
                )

        existing_challenge = Challenge.objects(
            testCategory = test_category,
            testId = test_id,
            status__in = [
                ChallengeStatus.PENDING.value,
                ChallengeStatus.ACCEPTED.value,
                ChallengeStatus.IN_PROGRESS.value
            ]
        ).filter(
            __raw__ = {
                "$or": [
                    {
                        "challengerUserId": challenger_id,
                        "challengedUserId": challenged_id
                    },
                    {
                        "challengerUserId": challenged_id,
                        "challengedUserId": challenger_id
                    }
                ]
            }
        ).first()

        if existing_challenge:
            raise BusinessRuleError(
                f"You already have an active challenge for {test_category} Test #{test_id} with this user. Complete or cancel it first.",
                error_code = "DUPLICATE_CHALLENGE"
            )

        challenge = Challenge(
            challengerUserId = challenger_id,
            challengedUserId = challenged_id,
            testCategory = test_category,
            testId = test_id,
            questionCount = question_count,
            mode = mode,
            status = ChallengeStatus.PENDING.value,
            createdAt = datetime.now(UTC)
        )
        challenge.save()

        challenger = User.objects(
            id = challenger_id
        ).only("username",
               "level",
               "currentAvatar").first()

        if challenger:
            SocialSocketService.emit_challenge_received(
                challenge_id = str(challenge.id),
                challenger_id = str(challenger_id),
                challenger_username = challenger.username,
                challenger_level = challenger.level,
                challenger_avatar = str(challenger.currentAvatar)
                if challenger.currentAvatar else None,
                challenged_id = str(challenged_id),
                test_category = test_category,
                test_id = test_id,
                question_count = question_count,
                mode = mode,
                expires_at = challenge.expiresAt,
                created_at = challenge.createdAt
            )

        return challenge
```

---

## Documentation

### Documentation for `send_challenge`

**Purpose and Behavior**
The `send_challenge` function creates a new challenge between two users based on specified criteria, such as test category and mode. It ensures the test exists in the database before creating the challenge and checks for existing active challenges to prevent duplicates.

**Key Implementation Details**
- **Type Hints**: The function uses type hints for parameters like `challenger_id`, `challenged_id`, etc.
- **ObjectId Conversion**: Converts string representations of IDs to `ObjectId` instances if necessary.
- **Random Test Selection**: If no test ID is provided, a random test from the specified category is selected.
- **Validation and Error Handling**: Validates that tests exist in the database and raises errors for non-existent or conflicting challenges.
- **Challenge Creation**: Creates and saves a new challenge object with appropriate status and timestamp.

**When/Why to Use This Code**
Use this function when setting up a new challenge between users, ensuring all necessary validations are performed. It is particularly useful in scenarios where dynamic test selection based on categories is required.

**Patterns and Gotchas**
- **Error Handling**: The function handles `NotFoundError` and `BusinessRuleError`, which should be caught and managed appropriately.
- **Database Queries**: Efficient use of MongoDB queries to validate the existence of tests and challenges, ensuring minimal database overhead.

---

*Generated by CodeWorm on 2026-01-19 13:37*

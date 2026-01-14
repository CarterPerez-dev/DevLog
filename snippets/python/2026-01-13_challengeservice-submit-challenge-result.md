# ChallengeService.submit_challenge_result

**Repository:** CertGames-Core
**File:** backend/api/domains/social/services/challenge_ops.py
**Language:** python
**Lines:** 267-379
**Complexity:** 17.0

---

## Source Code

```python
def submit_challenge_result(
        challenge_id: str | ObjectId,
        user_id: str | ObjectId,
        test_attempt_id: str | ObjectId,
        score: float,
        total_questions: int
    ) -> Challenge:
        """
        Submit test result for a challenge
        Updates challenger or challenged test attempt and score
        If both completed, determines winner and marks as completed
        """
        challenge_id = ObjectId(challenge_id) if isinstance(
            challenge_id,
            str
        ) else challenge_id
        user_id = ObjectId(user_id
                           ) if isinstance(user_id,
                                           str) else user_id
        test_attempt_id = ObjectId(test_attempt_id) if isinstance(
            test_attempt_id,
            str
        ) else test_attempt_id

        challenge = Challenge.objects(id = challenge_id).first()
        if not challenge:
            raise NotFoundError("Challenge", str(challenge_id))

        if challenge.status not in [ChallengeStatus.ACCEPTED.value,
                                    ChallengeStatus.IN_PROGRESS.value]:
            raise BusinessRuleError(
                "Challenge must be accepted to submit results",
                error_code = "CHALLENGE_NOT_ACTIVE"
            )

        percentage_score = (score / total_questions) * 100

        if user_id == challenge.challengerUserId:
            challenge.update(
                set__challengerTestAttemptId = test_attempt_id,
                set__challengerScore = percentage_score,
                set__status = ChallengeStatus.IN_PROGRESS.value
            )
        elif user_id == challenge.challengedUserId:
            challenge.update(
                set__challengedTestAttemptId = test_attempt_id,
                set__challengedScore = percentage_score,
                set__status = ChallengeStatus.IN_PROGRESS.value
            )
        else:
            raise BusinessRuleError(
                "User is not part of this challenge",
                error_code = "NOT_CHALLENGE_PARTICIPANT"
            )

        challenge.reload()

        opponent_id = (
            challenge.challengedUserId if user_id
            == challenge.challengerUserId else challenge.challengerUserId
        )
        submitter = User.objects(id = user_id).only("username").first()

        both_completed = (
            challenge.challengerScore is not None
            and challenge.challengedScore is not None
        )

        if submitter:
            SocialSocketService.emit_challenge_result_submitted(
                challenge_id = str(challenge.id),
                submitter_id = str(user_id),
                submitter_username = submitter.username,
                opponent_id = str(opponent_id),
                score = percentage_score,
                total_questions = total_questions,
                test_category = challenge.testCategory,
                both_completed = both_completed
            )

        if both_completed:
            if challenge.challengerScore > challenge.challengedScore:
                winner_id = challenge.challengerUserId
            elif challenge.challengedScore > challenge.challengerScore:
                winner_id = challenge.challengedUserId
            else:
                winner_id = None

            challenge.update(
                set__winnerId = winner_id,
                set__status = ChallengeStatus.COMPLETED.value,
                set__completedAt = datetime.now(UTC)
            )
            challenge.reload()

            winner = (
                User.objects(id = winner_id).only("username").first()
                if winner_id else None
            )

            SocialSocketService.emit_challenge_completed(
                challenge_id = str(challenge.id),
                challenger_id = str(challenge.challengerUserId),
                challenged_id = str(challenge.challengedUserId),
                winner_id = str(winner_id) if winner_id else None,
                winner_username = winner.username if winner else None,
                challenger_score = challenge.challengerScore,
                challenged_score = challenge.challengedScore,
                test_category = challenge.testCategory,
                completed_at = challenge.completedAt
            )

        return challenge
```

---

## Documentation

### Documentation for `submit_challenge_result`

**Purpose and Behavior**
The function `submit_challenge_result` is responsible for updating the result of a challenge in the database. It validates the user's eligibility, updates scores, and determines the winner if both participants have completed their attempts.

**Key Implementation Details**
- **Input Validation:** Converts input IDs to `ObjectId` if they are strings.
- **Challenge Status Check:** Ensures the challenge is active before updating results.
- **Score Calculation:** Computes the percentage score based on the total number of questions.
- **Update Logic:** Updates the relevant fields in the `Challenge` document and reloads it for consistency.
- **Emit Events:** Sends notifications to a social socket service when a result or challenge completion occurs.

**When/Why to Use This Code**
Use this function whenever a user needs to submit their test results for an ongoing challenge. It ensures that only valid participants can update the challenge status and provides real-time updates through the social socket service.

**Patterns and Gotchas**
- **Type Hints:** The use of `str | ObjectId` type hints helps in handling both string and object IDs.
- **Error Handling:** Proper error handling is implemented to manage invalid challenges or users not participating in the challenge.
- **Database Updates:** The function updates multiple fields in a single database call, reducing the number of operations and improving performance.

---

*Generated by CodeWorm on 2026-01-13 22:40*

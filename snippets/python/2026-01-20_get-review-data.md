# get_review_data

**Repository:** CertGames-Core
**File:** backend/api/domains/testing/controllers/analysis/review.py
**Language:** python
**Lines:** 14-108
**Complexity:** 15.0

---

## Source Code

```python
def get_review_data() -> dict[str, Any]:
    """
    Get all data needed for test review mode.
    """
    test: Any = g.test
    attempt: Any = g.test_attempt

    if not attempt or not attempt.finished:
        raise ValidationError("Test must be completed before review")

    score_data: dict[
        str,
        Any] = TestAnalysisService.calculate_final_score(attempt)

    questions_with_answers: list[dict[str, Any]] = []
    flagged_count: int = 0
    correct_count: int = 0
    skipped_count: int = 0

    # Use shuffleOrder to show only the questions that were selected (respects selectedLength)
    shuffle_order = attempt.shuffleOrder or []
    selected_length = attempt.selectedLength or len(test.questions)

    # Only review the questions that were actually in the test (shuffle order is already trimmed to selectedLength)
    for i, question_index in enumerate(shuffle_order[: selected_length]):
        question = test.questions[question_index]
        question_id = str(question.get('id', i + 1))
        answer_data = None

        for answer in attempt.answers:
            if isinstance(answer,
                          dict) and str(answer.get("questionId")
                                        ) == question_id:
                answer_data = answer
                break

        if isinstance(answer_data, dict):
            user_answer_index: int | None = answer_data.get(
                "userAnswerIndex"
            )
            is_flagged: bool = answer_data.get("flagged", False)
        elif answer_data is not None:
            user_answer_index = answer_data
            is_flagged = False
        else:
            user_answer_index = None
            is_flagged = False

        is_correct: bool = user_answer_index == question[
            'correctAnswerIndex'
        ] if user_answer_index is not None else False
        is_skipped: bool = user_answer_index is None

        if is_flagged:
            flagged_count += 1
        if is_correct:
            correct_count += 1
        if is_skipped:
            skipped_count += 1

        question_data: dict[
            str,
            Any] = {
                "questionIndex": i,
                "questionId": question.get('id',
                                           i + 1),
                "question": question['question'],
                "choices":
                question.get('options',
                             question.get('choices',
                                          [])),
                "correctAnswer": question['correctAnswerIndex'],
                "userAnswer": user_answer_index,
                "isCorrect": is_correct,
                "isFlagged": is_flagged,
                "isSkipped": is_skipped,
                "explanation": question.get('explanation',
                                            ''),
                "examTip": question.get('examTip',
                                        '')
            }

        questions_with_answers.append(question_data)

    return {
        "testId": test.testId,
        "testTitle": test.title,
        "attemptId": str(attempt.id),
        "scoreData": score_data,
        "totalQuestions": selected_length,  # Use selected length, not all questions
        "flaggedCount": flagged_count,
        "correctCount": correct_count,
        "skippedCount": skipped_count,
        "questionsWithAnswers": questions_with_answers
    }
```

---

## Documentation

### Documentation for `get_review_data`

**Purpose and Behavior:**
The function `get_review_data` retrieves all necessary data for a test review mode in an educational application. It processes the user's answers, calculates scores, and prepares detailed question-by-question feedback.

**Key Implementation Details:**
- The function validates that the test attempt is completed before proceeding.
- It uses shuffle order to filter questions based on selected length.
- A nested loop iterates through each shuffled question index, checking for corresponding answers and updating flags (correct, flagged, skipped).
- The final output includes a detailed breakdown of each question with its status.

**When/Why to Use:**
Use this function when implementing the review feature in an educational application. It ensures that users can see their performance on specific questions, helping them understand where they excelled or need improvement.

**Patterns and Gotchas:**
- The use of `Any` type hints suggests potential type safety issues; consider refining types for better maintainability.
- The function handles various edge cases (e.g., missing question IDs) by providing default values, which can be a double-edged sword in terms of robustness vs. flexibility.
- The nested loop and conditional checks make the code complex but necessary for accurate data processing.

---

*Generated by CodeWorm on 2026-01-20 23:17*

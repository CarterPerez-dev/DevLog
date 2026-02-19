# ReferralService

**Type:** Class Documentation
**Repository:** stripe-referral
**File:** src/stripe_referral/services/referral_service.py
**Language:** python
**Lines:** 35-219
**Complexity:** 0.0

---

## Source Code

```python
class ReferralService:
    """
    Service for referral business logic
    """
    @staticmethod
    def create_code(
        db: Session,
        user_id: str,
        program_key: str
    ) -> CreateCodeResult:
        """
        Generate unique referral code for a user
        """
        program_repo = ReferralProgramRepository(db)
        code_repo = ReferralCodeRepository(db)

        program = program_repo.get_by_key(program_key)
        if not program or not program.is_active:
            raise ProgramNotFoundError(
                f"Program '{program_key}' not found or inactive",
                program_key = program_key,
            )

        def check_collision(code_string: str) -> bool:
            """
            Check if code already exists in database
            """
            existing = code_repo.get_by_code(code_string)
            return existing is not None

        unique_code = generate_unique_code(
            user_id,
            program_key,
            check_collision
        )

        code = code_repo.create(
            code = unique_code,
            user_id = user_id,
            program_id = program.id,
            status = ReferralCodeStatus.ACTIVE.value,
        )

        return CreateCodeResult(
            code = code.code,
            program_id = code.program_id,
            user_id = code.user_id,
            created_at = code.created_at.isoformat(),
        )

    @staticmethod
    def validate_code(db: Session, code: str) -> CodeValidation:
        """
        Validate if a code is active and usable
        """
        code_repo = ReferralCodeRepository(db)

        code_obj = code_repo.get_by_code(code)
        if not code_obj:
            raise CodeNotFoundError(
                f"Code '{code}' not found",
                code = code
            )

        if code_obj.status != ReferralCodeStatus.ACTIVE.value:
            raise CodeInactiveError(
                f"Code '{code}' is inactive (status: {code_obj.status})",
                code = code,
                status = code_obj.status,
            )

        if code_obj.expires_at and code_obj.expires_at < datetime.now(UTC
                                                                      ):
            raise CodeExpiredError(
                f"Code '{code}' expired at {code_obj.expires_at.isoformat()}",
                code = code,
                expires_at = code_obj.expires_at.isoformat(),
            )

        if code_obj.max_uses and code_obj.uses_count >= code_obj.max_uses:
            raise CodeMaxUsesReachedError(
                f"Code '{code}' has reached maximum uses ({code_obj.max_uses})",
                code = code,
                max_uses = code_obj.max_uses,
                uses_count = code_obj.uses_count,
            )

        return CodeValidation(
            valid = True,
            code_id = code_obj.id,
            program_id = code_obj.program_id,
            referrer_user_id = code_obj.user_id,
        )

    @stat
```

---

## Class Documentation

### ReferralService Documentation

**Class Responsibility and Purpose:**
The `ReferralService` class is responsible for managing referral business logic, including generating unique referral codes, validating these codes, tracking successful referrals, and handling related errors.

**Public Interface (Key Methods):**
- **create_code(db: Session, user_id: str, program_key: str) -> CreateCodeResult**: Generates a unique referral code.
- **validate_code(db: Session, code: str) -> CodeValidation**: Validates if a given referral code is active and usable.
- **track_referral(db: Session, code: str, referred_user_id: str, transaction_id: str | None = None, transaction_amount: float | None = None) -> TrackReferralResult**: Tracks a successful referral conversion.

**Design Patterns Used:**
The class employs the **Factory Method** pattern for generating unique codes through the `create_code` method. The validation and tracking methods ensure that only valid operations are performed on the database, adhering to the Single Responsibility Principle by focusing on specific tasks.

**Relationship to Other Classes:**
- **ReferralProgramRepository**: Manages program-related data.
- **ReferralCodeRepository**: Manages referral code-related data.
- **ReferralTrackingRepository**: Tracks successful referrals and their details.

**State Management Approach:**
The class manages state through database repositories, ensuring that all operations are transactional and consistent. The use of static methods ensures that the service is stateless and can be easily tested in isolation.

This structure allows for a modular and maintainable architecture, where each method handles a specific aspect of referral management, making the codebase easier to understand and extend.

---

*Generated by CodeWorm on 2026-02-18 22:43*

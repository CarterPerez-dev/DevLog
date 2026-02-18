# EmailSender

**Type:** Class Documentation
**Repository:** CertGames-Core
**File:** backend/api/core/services/senders/email.py
**Language:** python
**Lines:** 11-304
**Complexity:** 0.0

---

## Source Code

```python
class EmailSender:
    """
    A utility class for sending emails through Resend
    with different sender addresses and templates.
    """
    def __init__(
        self,
        resend_api_key: str,
        frontend_url: str,
        admin_url: str | None = None
    ) -> None:
        resend.api_key = resend_api_key

        self.default_addresses: dict[str,
                                     str] = {
                                         "password_reset":
                                         Email.PASSWORD_RESET_SENDER,
                                         "newsletter":
                                         Email.NEWSLETTER_SENDER,
                                         "support": Email.SUPPORT_SENDER,
                                         "admin_invite":
                                         Email.ADMIN_INVITE_SENDER,
                                     }
        self.frontend_url: str = frontend_url
        self.admin_url: str = admin_url or f"{frontend_url}:8080"

    def send_email(
        self,
        to_email: str,
        subject: str,
        html_content: str,
        *,
        email_type: str = "password_reset",
        from_email: str | None = None,
    ) -> bool:
        """
        Send an email using Resend.
        """
        sender: str | None = from_email or self.default_addresses.get(
            email_type
        )

        if not sender:
            sender = Email.DEFAULT_SENDER

        try:
            response = resend.Emails.send(
                {
                    "from":
                    f"CertGames Support <{sender or Email.DEFAULT_SENDER}>",
                    "to": to_email,
                    "subject": subject,
                    "html": html_content,
                    "reply_to": "support@certgames.com",
                    "headers": {
                        "X-Entity-Ref-ID":
                        f"{email_type}-{to_email}",
                        "List-Unsubscribe":
                        "<mailto:support@certgames.com?subject=Unsubscribe>"
                    }
                }
            )

            if response and response.get("id"):
                current_app.logger.info(
                    "Email sent to %s (type: %s, id: %s)",
                    to_email,
                    email_type,
                    response.get('id')
                )
                return True

            current_app.logger.error(
                "Failed to send email: No ID returned"
            )
            return False

        except Exception as e:
            current_app.logger.error(f"Error sending email: {str(e)}")
            return False

    def send_password_reset_email(
        self,
        to_email: str,
        reset_token: str,
        frontend_url: str | None = None
    ) -> bool:
        """
        Send a password reset email with a reset link.
        """
        url: str = frontend_url or self.frontend_url
        reset_link: str = f"{url}/reset-pass
```

---

## Class Documentation

### EmailSender Class Documentation

**Class Responsibility and Purpose:**
The `EmailSender` class is a utility for sending emails through Resend, with support for different sender addresses and templates. It ensures that emails are sent correctly to recipients based on predefined email types.

**Public Interface (Key Methods):**
- **`__init__(self, resend_api_key: str, frontend_url: str, admin_url: str | None = None)`**: Initializes the `EmailSender` with a Resend API key and URLs for the frontend and admin.
- **`send_email(self, to_email: str, subject: str, html_content: str, *, email_type: str = "password_reset", from_email: str | None = None) -> bool`**: Sends an email using Resend with specified parameters. It uses default sender addresses if not provided and logs the result.
- **`send_password_reset_email(self, to_email: str, reset_token: str, frontend_url: str | None = None) -> bool`**: A specialized method for sending a password reset email with a personalized link.

**Design Patterns Used:**
The class leverages the Factory pattern implicitly through the use of default sender addresses and the `send_email` method. It also uses logging to handle error reporting, ensuring that issues are tracked effectively.

**Relationship to Other Classes:**
- **Resend API**: The Resend API is used for sending emails.
- **Email Class**: Assumed to be a utility class providing constants for default sender addresses (`EMAIL.PASSWORD_RESET_SENDER`, etc.).
- **CurrentApp (Flask App)**: For logging purposes, the `current_app` object from Flask is utilized.

**State Management Approach:**
The state of the `EmailSender` instance is managed through its attributes, such as `default_addresses`, `frontend_url`, and `admin_url`. These are initialized in the constructor and used throughout the class methods. The use of type hints ensures clarity on expected input types and method return values.

---

*Generated by CodeWorm on 2026-02-18 13:45*

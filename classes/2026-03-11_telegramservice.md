# TelegramService

**Type:** Class Documentation
**Repository:** CertGames-Core
**File:** backend/app/clients/telegram.py
**Language:** python
**Lines:** 16-166
**Complexity:** 0.0

---

## Source Code

```python
class TelegramService:

    def __init__(
        self,
        bot_token: str,
        admin_chat_id: str,
        webhook_secret: str | None = None
    ):
        self.bot_token = bot_token
        self.admin_chat_id = admin_chat_id
        self.webhook_secret = webhook_secret
        self.enabled = bool(bot_token and admin_chat_id)

    def _api_url(self, method: str) -> str:
        return TELEGRAM_API.format(token=self.bot_token, method=method)

    def send_message(
        self,
        text: str,
        reply_to_message_id: int | None = None
    ) -> int | None:
        if not self.enabled:
            return None

        payload: dict[str, Any] = {
            "chat_id": self.admin_chat_id,
            "text": text,
            "parse_mode": "HTML",
        }

        if reply_to_message_id:
            payload["reply_parameters"] = {
                "message_id": reply_to_message_id
            }

        try:
            response = requests.post(
                self._api_url("sendMessage"),
                json=payload,
                timeout=5
            )
            data = response.json()
            if data.get("ok"):
                return int(data["result"]["message_id"])

            current_app.logger.error(
                "Telegram API error: %s",
                data.get("description", "Unknown error")
            )
            return None
        except Exception as e:  # pylint: disable=broad-exception-caught
            current_app.logger.error(
                "Failed to send Telegram message: %s",
                str(e)
            )
            return None

    def register_webhook(self, webhook_url: str) -> bool:
        if not self.enabled:
            return False

        payload: dict[str, Any] = {
            "url": webhook_url,
            "allowed_updates": ["message"],
        }

        if self.webhook_secret:
            payload["secret_token"] = self.webhook_secret

        try:
            response = requests.post(
                self._api_url("setWebhook"),
                json=payload,
                timeout=10
            )
            data = response.json()
            if data.get("ok"):
                current_app.logger.info(
                    "Telegram webhook registered: %s",
                    webhook_url
                )
                return True

            current_app.logger.error(
                "Failed to register Telegram webhook: %s",
                data.get("description", "Unknown error")
            )
            return False
        except Exception as e:  # pylint: disable=broad-exception-caught
            current_app.logger.error(
                "Failed to register Telegram webhook: %s",
                str(e)
            )
            return False

    def notify_new_thread(
        self,
        thread_id: str,
        username: str,
        subject: str
    ) -> int | None:
        text = (
            "<b>New Support Thread</b>\n\n"
            f"User: {_escape_ht
```

---

## Class Documentation

### TelegramService Documentation

**Class Responsibility and Purpose:**
The `TelegramService` class is responsible for interacting with a Telegram bot API to send messages, register webhooks, and notify users about new threads, user messages, and mentorship payments.

**Public Interface (Key Methods):**
- **send_message(text: str, reply_to_message_id: int | None = None) -> int | None:** Sends a message to the admin chat ID. Returns the message ID if successful.
- **register_webhook(webhook_url: str) -> bool:** Registers a webhook for receiving updates from Telegram. Returns `True` on success and logs any errors.
- **notify_new_thread(thread_id: str, username: str, subject: str) -> int | None:** Sends a notification about a new support thread to the admin chat ID.
- **notify_user_message(thread_id: str, subject: str, username: str, content: str) -> int | None:** Sends a message from a user to the admin chat ID with context.
- **notify_mentorship_payment(application: dict, amount_paid: str) -> int | None:** Sends a notification about a mentorship payment received.

**Design Patterns Used:**
The class uses dependency injection for its constructor parameters and employs the Strategy pattern through the `send_message` method to handle different types of messages. It also uses logging via Flask's `current_app.logger` for error handling and information logging.

**How it Fits in the Architecture:**
This class acts as a client service that interacts with external APIs, specifically Telegram's bot API. It is part of the backend architecture designed to manage notifications and interactions within the CertGames-Core application. The class ensures that all messaging functionalities are centralized, making it easier to maintain and extend the notification system.

---

*Generated by CodeWorm on 2026-03-11 21:27*

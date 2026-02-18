# AdminEmailSender

**Type:** Class Documentation
**Repository:** CertGames-Core
**File:** backend/api/core/services/senders/admin_sender.py
**Language:** python
**Lines:** 10-222
**Complexity:** 0.0

---

## Source Code

```python
class AdminEmailSender:
    """
    Email sender specifically for admin-to-user communications
    """
    def __init__(self, resend_api_key: str, frontend_url: str) -> None:
        resend.api_key = resend_api_key
        self.frontend_url = frontend_url
        self.admin_from_email = "admin@certgames.com"

    def send_admin_message_to_user(
        self,
        to_email: str,
        username: str,
        subject: str,
        *,
        message: str,
        admin_email: str
    ) -> bool:
        """
        Send an admin message to a specific user
        """
        html_content = f'''
        <!DOCTYPE html>
        <html>
        <head>
            <meta charset="utf-8">
            <meta name="viewport" content="width=device-width, initial-scale=1.0">
            <title>{subject}</title>
        </head>
        <body style="font-family: Arial, sans-serif; color: #333333; line-height: 1.6; margin: 0; padding: 0;">
            <table width="100%" cellpadding="0" cellspacing="0" style="max-width: 600px; margin: 0 auto; padding: 20px; border: 1px solid #e0e0e0; border-radius: 5px;">
                <tr>
                    <td style="background-color: #2c3e50; padding: 20px; text-align: center; color: white; border-radius: 5px 5px 0 0;">
                        <h1 style="margin: 0; font-size: 24px;">CertGames Admin Team</h1>
                    </td>
                </tr>
                <tr>
                    <td style="padding: 30px; background-color: white; border: 1px solid #e0e0e0; border-top: none; border-radius: 0 0 5px 5px;">
                        <p>Hello {username},</p>
                        
                        <div style="background-color: #f8f9fa; padding: 20px; border-radius: 5px; margin: 20px 0; border-left: 4px solid #2c3e50;">
                            {message.replace(chr(10), '<br>')}
                        </div>
                        
                        <p>If you have any questions or need further assistance, please don't hesitate to reach out to our support team.</p>
                        
                        <div style="text-align: center; margin-top: 30px;">
                            <a href="{self.frontend_url}/support" style="background-color: #2c3e50; color: white; padding: 12px 30px; text-decoration: none; border-radius: 5px; display: inline-block;">Contact Support</a>
                        </div>
                        
                        <div style="margin-top: 40px; border-top: 1px solid #e0e0e0; padding-top: 20px; font-size: 12px; color: #777777;">
                            <p style="margin: 0;">
                                This message was sent by the CertGames admin team (via {admin_email}).<br>
                                For urgent matters, please visit our support center.
                            </p>
                        </div>
                    </td>
                </tr>
            </table>
        </body>
        </html>
        '''

        
```

---

## Class Documentation

### AdminEmailSender Class Documentation

**Class Responsibility and Purpose:**
The `AdminEmailSender` class is responsible for sending email notifications from admin to user, specifically tailored for administrative communications such as account warnings, suspensions, and reactivations. It leverages Resend API for email delivery and ensures that emails are formatted with a consistent template.

**Public Interface:**
- **send_admin_message_to_user**: Sends an admin message to a specific user.
  - Parameters:
    - `to_email`: Email address of the recipient.
    - `username`: Username of the recipient.
    - `subject`: Subject line for the email.
    - `message`: Body content of the email (HTML format).
    - `admin_email`: Email address of the admin sending the message.

- **send_admin_notification**: Sends an administrative notification to a user based on the type of notification.
  - Parameters:
    - `to_email`: Email address of the recipient.
    - `username`: Username of the recipient.
    - `notification_type`: Type of notification (e.g., "account_warning", "account_suspension").
    - `details`: Additional details about the notification.

**Design Patterns Used:**
- **Factory Method**: The class acts as a factory for creating and sending emails, encapsulating the logic for constructing email content.
- **Strategy Pattern**: Different types of notifications are handled through a strategy pattern by using predefined templates.

**Relationship to Other Classes:**
This class is part of the `CertGames-Core` backend API and interacts with the Resend API for email delivery. It is integrated into the application's core services layer, providing a standardized way to handle administrative communications.

**State Management Approach:**
The class maintains state through its attributes such as `resend.api_key`, `frontend_url`, and `admin_from_email`. These are initialized during instantiation and used throughout the lifecycle of the object. The methods do not modify these states but rather use them to construct and send emails.

---

*Generated by CodeWorm on 2026-02-18 14:31*

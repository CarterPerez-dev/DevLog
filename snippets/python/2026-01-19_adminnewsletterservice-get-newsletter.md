# AdminNewsletterService.get_newsletter_stats

**Repository:** CertGames-Core
**File:** backend/api/admin/domains/newsletter/services.py
**Language:** python
**Lines:** 379-489
**Complexity:** 12.0

---

## Source Code

```python
def get_newsletter_stats() -> dict[str, Any]:
        """
        Get overall newsletter statistics
        """
        total_subscribers = NewsletterSubscriber.objects().count()
        active_subscribers = NewsletterSubscriber.objects(
            unsubscribed = False
        ).count()
        unsubscribed_count = NewsletterSubscriber.objects(
            unsubscribed = True
        ).count()

        total_campaigns = NewsletterCampaign.objects().count()
        sent_campaigns = NewsletterCampaign.objects(status = 'sent'
                                                    ).count()
        draft_campaigns = NewsletterCampaign.objects(status = 'draft'
                                                     ).count()

        total_emails_sent = 0
        campaigns = NewsletterCampaign.objects(status = 'sent')
        for campaign in campaigns:
            total_emails_sent += campaign.recipientCount

        campaigns_with_stats = NewsletterCampaign.objects(
            status = 'sent',
            recipientCount__gt = 0
        )

        total_open_rate = 0
        total_click_rate = 0
        campaign_count = 0

        for campaign in campaigns_with_stats:
            if campaign.recipientCount > 0:
                open_rate = (
                    campaign.openCount / campaign.recipientCount
                ) * 100
                click_rate = (
                    campaign.clickCount / campaign.recipientCount
                ) * 100
                total_open_rate += open_rate
                total_click_rate += click_rate
                campaign_count += 1

        avg_open_rate = round(
            total_open_rate / campaign_count,
            2
        ) if campaign_count > 0 else 0
        avg_click_rate = round(
            total_click_rate / campaign_count,
            2
        ) if campaign_count > 0 else 0

        recent_campaigns = NewsletterCampaign.objects(
            status = 'sent'
        ).order_by('-sentAt').limit(5)

        recent_activity = []
        for campaign in recent_campaigns:
            recent_activity.append(
                {
                    "campaignId":
                    campaign.campaignId,
                    "subject":
                    campaign.subject,
                    "sentAt":
                    campaign.sentAt.isoformat()
                    if campaign.sentAt else None,
                    "recipientCount":
                    campaign.recipientCount,
                    "openRate":
                    f"{(campaign.openCount / campaign.recipientCount * 100):.1f}%"
                    if campaign.recipientCount > 0 else "0%"
                }
            )

        thirty_days_ago = datetime.now(UTC).replace(day = 1)
        new_subscribers = NewsletterSubscriber.objects(
            subscribedAt__gte = thirty_days_ago,
            unsubscribed = False
        ).count()

        growth_rate = 0
        if active_subscribers > 0:
            growth_rate = round(
                (new_subscribers / active_subscribers) * 100,
                2
            )

        last_campaign = NewsletterCampaign.objects(
            status = 'sent'
        ).order_by('-sentAt').first()

        last_campaign_date = None
        if last_campaign and last_campaign.sentAt:
            last_campaign_date = last_campaign.sentAt.isoformat()

        return {
            "total_subscribers": total_subscribers,
            "active_subscribers": active_subscribers,
            "unsubscribed_count": unsubscribed_count,
            "total_campaigns": total_campaigns,
            "sent_campaigns": sent_campaigns,
            "draft_campaigns": draft_campaigns,
            "total_emails_sent": total_emails_sent,
            "avg_open_rate": avg_open_rate,
            "avg_click_rate": avg_click_rate,
            "recent_activity": recent_activity,
            "growth_rate": growth_rate,
            "last_campaign_date": last_campaign_date
        }
```

---

## Documentation

### Documentation for `get_newsletter_stats`

**Purpose and Behavior**
The function `get_newsletter_stats` retrieves comprehensive statistics about newsletters, including subscriber counts, campaign metrics, recent activities, and growth rates. It returns a dictionary with various key performance indicators (KPIs) to provide an overview of newsletter performance.

**Key Implementation Details**
- **Database Queries**: The function performs multiple queries on the `NewsletterSubscriber` and `NewsletterCampaign` models using PyMongo.
- **Counting Subscribers and Campaigns**: It calculates active, unsubscribed subscribers, total campaigns, sent campaigns, and draft campaigns.
- **Email Sent Metrics**: It iterates over sent campaigns to calculate total emails sent, average open rate, and click rate. Recent activities are tracked by the last five sent campaigns.
- **Growth Rate Calculation**: The function computes the growth rate of active subscribers over the past 30 days.

**When/Why to Use This Code**
Use this code in an administrative dashboard or reporting tool where detailed newsletter performance metrics are required. It helps in monitoring and optimizing email marketing strategies by providing actionable insights into subscriber engagement and campaign effectiveness.

**Patterns and Gotchas**
- **Performance**: Multiple database queries can impact performance; consider caching results if used frequently.
- **Date Handling**: Ensure that date calculations, especially for recent activities and growth rates, are accurate to avoid misleading statistics.

---

*Generated by CodeWorm on 2026-01-19 21:17*

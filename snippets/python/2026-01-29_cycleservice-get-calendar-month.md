# CycleService.get_calendar_month

**Repository:** ios-test
**File:** fastapi/app/cycle/service.py
**Language:** python
**Lines:** 174-265
**Complexity:** 18.0

---

## Source Code

```python
async def get_calendar_month(
        self,
        user_id: UUID,
        year: int,
        month: int,
    ) -> CalendarMonth:
        """
        Get calendar data for a specific month
        """
        partner = await PartnerRepository.get_by_user_id(self.session, user_id)
        if not partner:
            raise PartnerNotFound(str(user_id))

        first_day = date(year, month, 1)
        if month == 12:
            last_day = date(year + 1, 1, 1) - timedelta(days = 1)
        else:
            last_day = date(year, month + 1, 1) - timedelta(days = 1)

        period_logs = await PeriodLogRepository.get_by_partner_id(
            self.session,
            partner.id,
            limit = 12,
        )

        daily_logs = await DailyLogRepository.get_date_range(
            self.session,
            partner.id,
            first_day,
            last_day,
        )
        daily_log_map = {log.log_date: log for log in daily_logs}

        period_dates: set[date] = set()
        predicted_dates: set[date] = set()

        for log in period_logs:
            period_length = partner.average_period_length
            if log.end_date:
                period_length = (log.end_date - log.start_date).days + 1

            for i in range(period_length):
                d = log.start_date + timedelta(days = i)
                if log.is_predicted:
                    predicted_dates.add(d)
                else:
                    period_dates.add(d)

        if partner.last_period_start:
            predicted_start = partner.last_period_start + timedelta(
                days = partner.average_cycle_length
            )
            while predicted_start <= last_day:
                if predicted_start >= first_day:
                    for i in range(partner.average_period_length):
                        d = predicted_start + timedelta(days = i)
                        if first_day <= d <= last_day and d not in period_dates:
                            predicted_dates.add(d)
                predicted_start += timedelta(days = partner.average_cycle_length)

        days: list[CalendarDay] = []
        current_date = first_day

        while current_date <= last_day:
            cycle_day = None
            phase = CyclePhase.UNKNOWN

            if partner.last_period_start:
                days_since = (current_date - partner.last_period_start).days + 1
                if days_since > 0:
                    cycle_day = ((days_since - 1) % partner.average_cycle_length) + 1
                    phase = self._get_phase(cycle_day, partner.average_cycle_length)

            daily_log = daily_log_map.get(current_date)

            days.append(CalendarDay(
                date = current_date,
                cycle_day = cycle_day,
                phase = phase,
                is_period = current_date in period_dates,
                is_predicted_period = current_date in predicted_dates,
                has_daily_log = daily_log is not None,
                mood = daily_log.mood if daily_log else None,
            ))

            current_date += timedelta(days = 1)

        return CalendarMonth(
            year = year,
            month = month,
            days = days,
        )
```

---

## Documentation

### Documentation for `get_calendar_month`

**Purpose and Behavior:**
The function `get_calendar_month` retrieves calendar data for a specific month, including period logs, daily logs, and predicted dates based on the user's partner information. It constructs a list of `CalendarDay` objects representing each day in the specified month, with attributes indicating whether it is part of a period, a predicted period, or has a daily log.

**Key Implementation Details:**
- **Asynchronous Operations:** The function uses async/await to handle database queries efficiently.
- **Date Calculations:** It calculates the first and last days of the month and iterates through each day to determine its attributes based on logs and cycle predictions.
- **Predicted Periods:** Predictions are made using average period length, cycle length, and previous period start date.

**When/Why to Use:**
This function is useful for generating detailed monthly calendars that include health-related data. It can be used in applications needing to display or analyze menstrual cycles over time, such as fertility tracking apps or health monitoring systems.

**Patterns/Gotchas:**
- **Date Handling:** The code carefully handles date calculations and comparisons, ensuring accurate period and predicted period identification.
- **Predictive Logic:** Predicted periods are calculated based on historical data, which may not always be accurate. Users should consider this when interpreting the results.
- **Asynchronous Calls:** Ensure that database connections and queries are managed properly to avoid performance bottlenecks.

This function provides a comprehensive view of a user's menstrual cycle by integrating period logs, daily logs, and predicted periods into a single calendar month representation.

---

*Generated by CodeWorm on 2026-01-29 13:56*

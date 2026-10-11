# Customer Feedback Call System

> **Every morning, every store gets its own call list. The team only calls and types what the customer said — everything else is automatic, including escalating complaints until someone fixes them.**

<p>
  <img src="https://img.shields.io/badge/role-Sole%20Builder-orange" alt="Sole Builder">
  <img src="https://img.shields.io/badge/Google_Apps_Script-4285F4?style=flat&logo=google&logoColor=white" alt="Apps Script">
  <img src="https://img.shields.io/badge/Google_Sheets-34A853?style=flat&logo=googlesheets&logoColor=white" alt="Google Sheets">
</p>

A daily customer feedback-call system for a 25+-outlet family entertainment chain. Each morning it picks yesterday's walk-in customers for every store straight from the POS, hands the calling team a ready list, tracks which stores actually made their calls, and turns negative feedback into tracked escalations. Since October it also follows up every completed birthday party.

---

## The problem

Feedback calls were run from a manual workbook: one tab per store, date blocks built by hand, customer numbers copied in by someone each day. It worked as a record, but it couldn't answer the questions that mattered:

- Did each store actually make its calls today?
- Did the unhappy customer from Tuesday ever hear back?
- When a new store opens, is it getting the extra attention it needs in its first weeks?

A complaint typed into a remarks cell had nowhere to go.

---

## What the team sees

One Google Sheet. Every morning, 5 customers per store from the previous day are already listed. The team does two things only: call, and type what the customer said. Everything else is filled in for them.

---

## The rules that make it work

**Store targets depend on store age.** A store in its first 45 days after opening must complete **5 calls a day**; an established store needs **2**. New stores join the list automatically when they first appear in the POS, and their 45-day clock starts from their opening date (a store that closed and reopened restarts from the reopening).

**Walk-in customers only.** The list takes paid walk-in sessions and skips voucher redemptions, voucher purchases, free entries, and extension-only sales. The question being asked is about the visit experience, so the list is built from the customers who had one.

**"Call done" means a remark was typed — not that the customer picked up.** This was a deliberate metric choice. If "done" meant "connected," a new store could never hit 5/5 on a day when customers don't answer, and the target would punish the team for something outside its control. Reach rate is still tracked, just separately, so neither number hides the other.

**The first remark stamps the time automatically.** Calls made late in the day show as *Done late* rather than simply *Done*.

---

## Escalations: complaints can't quietly disappear

Each call row has **Escalate** and **Resolved** checkboxes and a resolution note:

- Ticking **Escalate** stamps the time; the row turns red.
- Ticking **Resolved** stamps the time; the row turns green. *Resolved* only counts if the row was escalated first, so a stray tick can't close a complaint that was never raised.
- The monitoring dashboard has an Escalations view with the age of every open complaint, and a banner once anything has been open longer than 3 days.

---

## Birthday party follow-ups

Every birthday party that genuinely completed gets its own follow-up call on a separate tab, with the same escalation workflow.

"Completed" needed a precise definition, because the booking data includes cancellations, future parties and unpaid bookings:

> **status confirmed + party date already passed + actually paid** (settled amount greater than zero)

Two data problems had to be handled rather than ignored:

- **Party dates arrive in several formats**, plus a few values that aren't dates at all. Recognised formats are parsed; anything unreadable is skipped and counted in a sync log — never guessed into a date.
- **Bookings come from two separate POS databases**, so a bill number alone isn't unique. Every party is keyed on *database + bill number* to stop two different parties overwriting each other.

A party with no call remark after 2 days is flagged overdue.

---

## Privacy decisions

- The chain's analytics database stores customer phone numbers **only as hashes**. The calling team needs real numbers to dial, so instead of weakening that, the sheet pulls each day's list directly from the POS into the team's own sheet.
- The monitoring dashboard is built for managers, not callers, so it receives **counts, first names and remarks only** — never a phone number, address, or parent's name.
- The dashboard's data endpoint requires a key; a wrong key returns *unauthorized*.

---

## Architecture

```
POS booking/visit APIs
        │  hourly trigger: make sure today's list exists
        ▼
Google Apps Script ──► Google Sheet (Calls · Birthday Calls · Stores · Monitor · Config)
        │                    ▲
        │                    └── team types remarks, ticks Escalate / Resolved
        │                        (onEdit stamps call / escalation / resolution times)
        ▼
JSON web endpoint (keyed) ──► monitoring dashboard
                               per store, per day: calls done vs target,
                               open escalations by age, birthday follow-ups
```

---

## Testing

**104 unit tests, 50 end-to-end tests and 37 escalation tests**, plus a cross-check of the sheet's own monitor formulas against an independent calculation. The full loop — POS fetch, remark → timestamp → dashboard status, escalate → resolve — was verified in the real Google environment before handover, and the test data removed afterwards.

---

## Status

Call lists, automation, escalations and birthday follow-ups are live (October 2026). The store-monitoring dashboard is built and being rolled out to managers.

---

## Tech stack

`Google Apps Script` · `Google Sheets` · `POS REST APIs` · `JavaScript` · `HTML/CSS`

---

### Author

**Souvik Kundu** — Business Intelligence & Automation Engineer.

📫 [LinkedIn](https://linkedin.com/in/souvik-kundu-bi) · [GitHub](https://github.com/Souvikkundu369)

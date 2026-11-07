# Tempo Café

A full-stack booking and operations platform for a pay-for-time coworking café. It covers the public website, online booking and payment, the reception desk and the owner's back office, in French and English.

> **Portfolio demo.** Tempo Café is a fictional business. Names, prices, opening hours, photos and contact details are all made up. Payments are simulated, and in demo mode no email leaves the app.

> **Status: work in progress.** The core product works end to end. I'm still polishing the UI and building a few features, such as networking events, so some screens may change.

## Contents

1. [Why I built it](#why-i-built-it)
2. [What it does](#what-it-does)
3. [Try the demo](#try-the-demo)
4. [Tech stack](#tech-stack)
5. [How it's built](#how-its-built)
6. [Deployment](#deployment)
7. [Hard problems](#hard-problems)

## Why I built it

This is a self-initiated project. I was a regular at a local coworking café I really liked, and saw how much of its day-to-day ran on the staff's own effort. I wanted to see how far one well-fitted system could take that off their hands.

I did it on my own, end to end:

- **Research.** Talked with the staff about their day: confirming bookings, keeping track of who is in and for how long, working out each bill and handling private-event requests.
- **Audit.** Studied how the café's current setup fits together: its website, booking tool, hosting and the services behind them, what each one costs and why it was likely chosen. This showed what already worked and where a custom system would add the most, so the right ideas carried over into a full rebuild from scratch on a new stack.
- **Market and pricing.** Compared competitors, booking SaaS and transport operators' fare rules (fixed versus flexible tickets, changes, cancellations, refunds), and used them to shape the booking options, cancellation rules and credit.
- **Product.** Wrote the specification and the business rules. Kept the café's own pay-for-time pricing and designed everything around it for online use: fixed and flexible bookings, extensions, rescheduling, cancellations, refunds and credit, and paying with plans. Kept a decision log and a development record throughout.
- **Design.** Built a design system and wrote all the copy, French first, then English.
- **Build and ship.** Built the whole stack: the public site, booking and payment flows, the reception desk, the back office, accounts and roles, background jobs and emails. Tested it with automated tests and browser checks in both languages on desktop and mobile, ran code, security, UI and accessibility audits, then packaged and deployed it on low-cost infrastructure.

The result is one app that runs the whole café.

## What it does

### Customers

- **Browse** the spaces, what's included and every price, and see whether the café is open and how busy the open space is.
- **Walk in** to the open space, no booking needed, and pay for the time you stay, with a daily cap.
- **Book a private room or the meeting room** online without an account. Only free times are offered, and your slot is held while you pay.
- **Choose how you book**: set your departure and pay in full, or book a flexible slot, pay the first hour and settle the rest when you leave. The meeting room also comes by the half day or full day.
- **Pay** by card, wallet credit or plan minutes, and get a ticket with an arrival code, a confirmation email and a reminder.
- **Check in on your own** at reception with the QR code and your arrival code, and check out the same way when nothing is owed.
- **Manage everything from your account**: reschedule, cancel or extend a booking within the booking terms, and see your visits, plans, credit, payments and refunds.
- **Buy plans**: day packs, a monthly subscription and student subscriptions, online or at the counter. Students unlock theirs with proof, uploaded from their account or shown at the counter.
- **Hire the whole venue** by requesting a quote from your account, then paying it and picking the date online.

### Reception staff

- A **touch-screen desk** to start walk-ins, check people in and out, extend visits, and mark or reverse no-shows.
- **Checkout** that shows the exact amount owed after the amount already paid, plan minutes and credit, then takes cash, card or wallet credit, or leaves a balance to collect later.
- **Counter sales** of any plan, plus drinks added to a student's bill and plan prints and drinks recorded.
- A **check-in screen** for customers on a second display at reception.
- A **room calendar**, today's arrivals, the open-space headcount and closures of the café or a single space.
- **Live notifications**, for example when a flexible visit reaches its time limit.

### Owner

- **Dashboards** for today's figures, revenue by source and period, and every booking.
- **Booking control**: cancel any booking with a reason, after which the customer picks a new slot or a refund, and waive balances.
- **Customer tools**: look up accounts, manage roles, issue credit, extend or revoke plans, and review student proofs.
- **Private-hire quotes**: set the price, length and allowed dates, or decline.
- **Closures** that list the bookings they affect before anything is cancelled.
- **Problem alerts** for undelivered emails and refunds that keep failing, plus an **audit log** of every action that touches money, access or inventory.

### Runs on its own

- Unpaid holds expire and free their slot. A payment that arrives late still gets its booking if the slot is free, or the customer's choice of a new slot or a refund if it isn't.
- Missed bookings become no-shows and free the room. Visits nobody checked out close at closing time, and any balance stays open for staff.
- A failed refund leaves the booking as it was, is retried, then flagged to the owner.
- Emails and reminders are queued with the change they report, retried on failure and never sent twice.

## Try the demo

**Live demo:** [tempocafe.chinople.com](https://tempocafe.chinople.com)

- Pick a role, **customer**, **barista** or **admin**, and the login form fills in that demo account. Sign-up is off.
- Card payments go through a mock provider, so nothing is charged.
- Data resets every night.

## Tech stack

| Area | Choice |
| --- | --- |
| Framework | Next.js 16 (App Router), React 19, TypeScript |
| Runtime | Bun |
| Database | SQLite (WAL mode) with Drizzle ORM |
| Auth | Better Auth: email and password, optional Google sign-in, customer, barista and admin roles |
| UI | Tailwind CSS 4 and Radix primitives, with a custom design system |
| i18n | next-intl, French by default, English under `/en` |
| Email | Queued outbox, sent through Resend, or a mock mailbox in demo mode |
| Charts | Recharts |
| Real time | Server-Sent Events for in-app notifications |
| Validation | Zod |
| Tests | Bun test, Playwright for browser checks |
| Hosting | Docker Compose on a VPS, behind Cloudflare Tunnel |

## How it's built

- **One app, one database.** Deliberately simple: a single Next.js app, SQLite on one server, and background jobs that run every minute.
- **The server decides.** Prices, availability, permissions and status changes are all checked on the server.
- **Safe money handling.** Payments, refunds, credit and room inventory change inside locked database transactions, with idempotency keys so a retry or a double click never charges twice.
- **One pricing engine.** All billing rules live in one place, so the booking flow, the desk, the customer's account and the dashboards always show the same amount.
- **Business-time aware.** Opening hours, billing and deadlines follow the café's time zone, not the server's.
- **Audited.** Every staff, admin or system action that moves money, access or inventory is written to a permanent log.
- **Tested where it matters.** Tests focus on pricing, booking states, refunds and edge cases such as retries, concurrent requests and failures halfway through.

## Deployment

- **One Docker image**, so a release is a pull and a restart, and a rollback is the previous image.
- **Docker Compose on a single VPS** runs the app plus small worker containers: one runs the per-minute jobs (holds, no-shows, late payments, refunds, reminders), one sends queued emails.
- **Cloudflare Tunnel** brings traffic in, so no port is open on the server.
- **Hardened containers** run with no Linux capabilities.
- **Backups** of the SQLite database are taken while the app keeps running.
- **Nightly demo reset** rebuilds the data from seeds with no downtime.
- **Low running cost.** Everything runs on one small server with a file-based database, with no paid booking, managed hosting or database services. The one outside service is Resend for email, which the expected volume should keep on its free tier.
- **Protecting the email quota.** Emails anyone can trigger are rate-limited per visitor or per address, an address that bounces or reports spam gets nothing more, nothing is ever sent twice, a reminder that would arrive too late is dropped, and sending follows the provider's rate. The public demo sends no email at all.

## Hard problems

- **Double bookings.** Two people can go for the same slot at the same moment. Availability takes in bookings, live holds, private hires, closures and the turnaround gap between bookings, and every hold is taken in a locked transaction, so only one person gets it.
- **Billing by time.** Visits are billed by the hour and then in time blocks, with a short grace period, a daily cap in the open space, and plan minutes, deposits and credit applied before anything is collected. The customer, the desk and the dashboards always see the same amount.
- **Payments that go wrong.** A reply can get lost, a customer can click twice, a payment can land after the hold has expired, or a refund can be refused. Each case ends in one clear state: no duplicate charge, and the booking or the money back.
- **Cancellations.** The rules depend on the booking type, how early you cancel and who cancels. Each part of the payment, whether card, credit or plan minutes, goes back to the right place.
- **Guests becoming customers.** People can book without an account. Their bookings join their account later, but only after they prove they own the email address.
- **Check-in without staff.** A QR code at reception that changes every minute, plus a personal arrival code, lets customers check themselves in. A photo of the code stops working within a couple of minutes.
- **Emails that always match reality.** An email is queued in the same transaction as the change it reports, so a change that fails sends nothing, and one that succeeds always gets its email, in the right order.
- **Releases with open tabs.** A browser tab left open during a deploy reloads cleanly instead of breaking.

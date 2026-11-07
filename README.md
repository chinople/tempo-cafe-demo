# Tempo Café

**The website and day-to-day system of a pay-for-time coworking café.** Customers book and pay online, staff run the front desk, and the owner sees the whole business, in French and English.

**[Try the live demo →](https://tempocafe.chinople.com)**

> [!NOTE]
> Tempo Café is a fictional business made for this portfolio demo. Names, prices, hours, photos and contacts are made up, payments are simulated and no email is sent.
>
> The project is a work in progress. The core works end to end, and I'm still polishing the UI and finishing a few features, such as networking events.

## Contents

1. [Why I built it](#why-i-built-it)
2. [What it does](#what-it-does)
3. [Try the demo](#try-the-demo)
4. [Tech stack](#tech-stack)
5. [How it's built](#how-its-built)
6. [Deployment](#deployment)
7. [Hard problems](#hard-problems)

## Why I built it

I was a regular at a local coworking café I really liked, and saw how much of its day-to-day ran on the staff's own effort. So I set out to build one system that could take that off their hands.

I did it on my own, from first conversation to deployment:

- **Research:** talked with the staff about their day: bookings, tracking who's in and for how long, working out bills and handling private events.
- **Audit:** studied the café's current website, booking tool and hosting, what each costs and why it was chosen, to know what to keep and what to rebuild.
- **Market:** compared competitors, booking SaaS and transport fare rules (fixed vs. flexible tickets, changes, refunds) to shape the booking options and policies.
- **Product:** kept the café's own pay-for-time pricing and designed the rest around it: flexible bookings, extensions, rescheduling, cancellations, refunds, credit and plans.
- **Design:** built a design system and wrote all the copy, French first.
- **Build and ship:** rebuilt everything from scratch on a new stack, then tested, audited and deployed it.

## What it does

### For customers

- See the spaces, prices, opening status and how busy it is right now.
- Walk in to the open space and pay only for the time you stay.
- Book a private room or the meeting room in a short step-by-step booking flow, no account needed: pick a space, a time, then pay.
- Pick a set departure, or a flexible slot and pay the rest when you leave. The meeting room also books by the half or full day.
- Pay by card, credit or plan minutes, and get a ticket with your arrival code, a confirmation email and a reminder.
- Check in at reception by scanning the QR code and entering your arrival code, and check out the same way when nothing is owed.
- Reschedule, cancel or extend from your account, and see your visits, plans, credit and payments.
- Buy day packs and subscriptions online or at the counter, with student plans unlocked by proof.
- Request a quote to hire the whole venue, then pay and pick the date online.

### For reception staff

- A touch-screen desk for walk-ins, check-ins, check-outs, extensions and no-shows.
- Checkout that shows exactly what's owed after plans and credit, then takes cash, card or credit, or leaves a balance for later.
- Plan sales at the counter, plus student drinks and the prints and drinks included in plans.
- A room calendar, today's arrivals, the open-space headcount and closures.
- A check-in screen for customers on a second display.
- Live alerts, like a visit reaching its time limit.

### For the owner

- Dashboards for daily figures, revenue and bookings.
- Cancel any booking with a reason, and the customer picks a new slot or a refund.
- Issue credit, waive balances, manage plans and roles, and review student proofs.
- Set the price, length and dates of private-hire requests, or decline them.
- Close the café or a room, and see who's affected first.
- Alerts when an email or a refund fails, and a log of every action on money, access or rooms.

### On its own

- Frees slots when a payment isn't finished in time. A late payment still gets its booking, or a new slot or a refund.
- Marks no-shows and closes forgotten visits at closing time, leaving any balance for staff.
- Sends reminders, retries failed refunds and emails, and never sends the same email twice.

## Try the demo

**[Open the live demo →](https://tempocafe.chinople.com)**

- Pick **customer**, **barista** or **admin** on the login page to sign in as that role. Sign-up is off.
- Payments are fake, so nothing is charged.
- The data resets every night.

## Tech stack

| | |
| --- | --- |
| **App** | Next.js 16, React 19, TypeScript, Bun |
| **Data** | SQLite with Drizzle ORM |
| **Auth** | Better Auth, with Google sign-in and customer, barista and admin roles |
| **UI** | Tailwind CSS 4, Radix, Recharts, custom design system |
| **Languages** | next-intl (French and English) |
| **Live updates** | Server-Sent Events for notifications |
| **Email** | Outbox queue sent through Resend, with a fake mailbox in the demo |
| **Tests** | Bun test, Playwright |
| **Hosting** | Docker on a VPS, Cloudflare Tunnel |

## How it's built

- **Simple on purpose:** one app, one database, one server.
- **The server has the final say** on prices, availability, permissions and booking status.
- **Money is handled carefully:** a retry or a double click never charges twice.
- **One set of pricing rules,** so the website, the desk and the dashboards always agree.
- **Everything is logged:** every staff, admin or system action on money, access or rooms is kept in a permanent audit log.
- **The café's own clock:** hours and deadlines follow its time zone, not the server's.
- **Emails go through an outbox:** each one is saved with the change it reports, then sent by a background job, so no action waits on the email service.
- **Tests focus on what matters most:** pricing, bookings and refunds, including retries, simultaneous requests and failures halfway through.

## Deployment

- Runs as Docker containers on one small server, behind Cloudflare Tunnel, with no open ports.
- Background jobs run every minute, and a separate one sends the outbox.
- Releases and rollbacks just swap the image, and containers run with minimal permissions.
- Backups are taken without stopping the app, and the demo resets nightly with no downtime.
- **Low cost:** no paid database, hosting or booking service. Resend is the only outside service, and should stay on its free tier.
- **Email quota protected:** rate limits, no emails to addresses that bounce or report spam, no duplicates or late reminders, and sending paced to the provider's limit.

## Hard problems

- **Double bookings:** two people clicking the same slot at once. Availability accounts for bookings, open holds, private hires and closures, and only one person gets it.
- **Billing by time:** first hour, then time blocks, with a grace period, a daily cap in the open space, plan minutes and credit.
- **Payments that go wrong:** lost replies, double clicks, late payments and refused refunds still end with one charge, and either the booking or the money back.
- **Cancellations:** different rules by booking type, timing and who cancels, with each part of the payment refunded to the right place.
- **A booking flow that feels simple:** only free times are offered, your slot is held with a visible countdown, and your credit or plan minutes are suggested when they cover the booking.
- **Guests becoming customers:** guest bookings join an account only after its email is verified.
- **Self check-in:** a QR code that changes every minute plus a personal code, so it only works at the café.
- **Emails in sync:** thanks to the outbox, an email goes out only if its change saved, always goes out if it did, and arrives in the right order.
- **Smooth releases:** tabs left open during a deploy reload cleanly instead of breaking.

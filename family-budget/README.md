# Hearth: Family Budget App Plan

A shared family budget app for iOS, Android and the web, built to replace the family's Google Sheet. "Hearth" is a working name.

`dashboard-prototype.html` is a clickable dashboard mockup that uses sample data. Open it in any browser.

---

## 1. What the best tools do, and where Hearth wins

| App | Strong at | Weak for families |
|---|---|---|
| **YNAB** | Zero-based "give every dollar a job" budgeting | Steep learning curve, $109/yr, little per-person context |
| **Monarch Money** | Clean dashboard, bank sync, shared household access | No kid/teen roles, generic categories |
| **Copilot Money** | Beautiful UI, smart categorization | iPhone/Mac only, so Android family members can't use it |
| **Goodbudget** | Envelope budgeting synced across a household | Dated UI, no bank sync on the free tier |
| **Rocket Money** | Finding and cancelling subscriptions | Weak budgeting depth |
| **Google Sheets** (current) | Flexible and free | Manual entry, poor on phones, no alerts, formulas break |

**Hearth's edge is that it's built for the whole family, not one power user:**
- **Roles:** Parents (full access), Partner, Teen (sees own allowance and goals), Viewer.
- **"Who spent it"** on every transaction, with a per-member filter on every screen.
- **Safe-to-spend number:** remaining budget minus bills still due, shown as a daily amount.
- **Pace markers:** each category shows where spending *should* be on today's date.
- **Shared goals** with progress rings (trip, emergency fund, college).
- **Kids' allowance and chores** wallet, which no competitor does well.
- **"Needs review" queue:** a parent approves categories the app wasn't sure about.
- **One-click Google Sheets import** so no history is lost.

## 2. Recommended tech stack

| Layer | Choice | Why |
|---|---|---|
| App framework | **Expo (React Native) + Expo Router, TypeScript** | One codebase ships to iOS, Android **and** the web. |
| UI | **Tamagui** (or NativeWind) | Shared design tokens, light and dark themes, fast on all three platforms |
| Charts | **Victory Native XL** (Skia) on mobile, the same components on web | Smooth, animated charts |
| Backend | **Supabase** (Postgres, Auth, Realtime, Storage, Edge Functions) | Row-Level Security makes household sharing safe. Realtime keeps every phone in sync instantly. |
| Offline | **PowerSync** (Supabase-native) or TanStack Query + MMKV cache | Expenses can be added with no signal and sync later |
| Bank sync | **Plaid** (US/CA); **GoCardless Bank Account Data / TrueLayer** (UK/EU) | Automatic transaction import |
| AI | **Claude API**: Haiku 4.5 for auto-categorizing, Sonnet for receipt scanning and monthly "money check-in" summaries | Fewer manual edits than a spreadsheet |
| Notifications | Expo Notifications + Supabase cron | Bill reminders, over-budget alerts, weekly digest |
| Auth | Supabase Auth: Sign in with Apple, Google, email magic link; Face ID / fingerprint lock | Easy for every family member |
| Builds & stores | **EAS Build / Submit / Update** | App Store + Play Store releases and over-the-air fixes |
| Web hosting | Vercel (Expo web export) | Free tier is enough for a family |
| Monitoring | Sentry + PostHog | Crash reports and usage |

**Rough running cost for one family:** Supabase free–$25/mo, Plaid about $0.30–$1.50 per connected account per month, Apple developer $99/yr, Google Play $25 one-time, Claude API a few dollars a month.

**Alternative:** Flutter + Firebase is equally capable. Choose it only if you prefer Dart. Expo + Supabase is the better fit here because a SQL database suits financial data and the web build is stronger.

### Core data model (Postgres)
```
households(id, name, currency, budget_method)
members(id, household_id, user_id, display_name, role, color)       -- role: owner|partner|teen|viewer
accounts(id, household_id, name, type, institution, balance, owner_member_id)
categories(id, household_id, group_name, name, icon, is_fixed)
budgets(id, household_id, month, category_id, assigned_amount)
transactions(id, household_id, account_id, date, merchant, amount,
             category_id, member_id, status, notes, receipt_url)     -- status: cleared|pending|needs_review
recurring_bills(id, household_id, name, amount, due_day, autopay, account_id)
goals(id, household_id, name, target, saved, target_date, color)
allowances(id, member_id, weekly_amount, balance)
```
Every table carries `household_id`, and Row-Level Security allows a user to see only rows for households they belong to. Teen visibility is limited further by role.

## 3. Brand and visual design

The palette is warm and calm, close to Claude's look: brownish, clay, teal and white.

| Token | Light | Dark | Use |
|---|---|---|---|
| Ivory background | `#FAF9F5` | `#171411` | Page |
| Surface | `#FFFFFF` | `#221E1A` | Cards |
| Ink | `#221C16` | `#F2ECE4` | Text |
| Clay (primary) | `#C4643F` | `#E3896A` | Buttons, spending, highlights |
| Deep brown | `#5B3F2C` | `#120F0D` | Sidebar, hero card |
| Teal | `#1C7A72` | `#55BCB1` | Income, savings, "on track" |
| Amber | `#A86A12` | `#E2A94E` | Warning / ahead of pace |
| Red | `#A3322A` | `#E7766A` | Over budget only |

**Type:** Newsreader (serif) for headings and big numbers gives an editorial, trustworthy feel. Instrument Sans is used for UI text and JetBrains Mono for amounts in lists. Money always uses tabular numbers so digits line up.

**Rules:**
- Teal = money in or good. Clay = money out. Red appears only when something is actually over budget.
- Status is shown in words *and* color ("$62 over", "Ahead of pace") for colorblind users.
- 44 px minimum touch targets, plus a big floating **+** button to add an expense in 2 taps.
- Full dark mode from day one.

## 4. UX and screens

**Navigation:** a bottom tab bar on phones (Home · Budget · **+** · Goals · Reports) and a sidebar on tablet/web.

1. **Dashboard:** greeting with month status, Safe-to-spend hero with pace bar, KPI tiles (income, spent, saved, net worth), 6-month cash-flow chart, upcoming bills, budget categories with pace ticks, goals, recent transactions, and an Everyone / per-member filter.
2. **Budget:** zero-based monthly plan. Drag money between categories. Roll leftovers into next month. Templates copy last month's plan.
3. **Add expense** (the + button): amount keypad first, then the app suggests a category and the member. Snap a receipt photo and AI fills in the details.
4. **Transactions:** search, filters, split a transaction, "needs review" queue, comments ("what was this?").
5. **Bills calendar:** month view of recurring bills, autopay badges, and a reminder 3 days before each due date.
6. **Goals:** shared savings goals with a contribution plan ("$310/mo to hit it by June").
7. **Kids:** allowance wallet, chores to earn, savings jar; a simplified view for teens.
8. **Reports:** spending trends, category breakdown, per-member breakdown, year-over-year, CSV/PDF export.
9. **Monthly check-in:** an AI-written summary on the 1st ("Dining was 18% higher; groceries saved $120").

**Onboarding (under 3 minutes):** create household → invite family by link/QR → import Google Sheet or connect bank → pick a starter budget template.

## 5. Build roadmap

| Phase | Weeks | Deliverables |
|---|---|---|
| 1. MVP | 1–6 | Auth, household + invites, manual transactions, categories, monthly budget, dashboard, Google Sheets CSV import, web + TestFlight/internal Android |
| 2. Automation | 7–10 | Plaid bank sync, auto-categorize (Claude Haiku), recurring bills + reminders, offline mode |
| 3. Family | 11–14 | Roles & teen view, allowances, goals, receipt scanning, needs-review queue |
| 4. Polish & launch | 15–18 | Reports, monthly AI check-in, widgets (iOS/Android home screen), App Store / Play Store release |

## 6. Security and privacy
- Bank credentials never touch our servers (Plaid/aggregator token exchange only).
- Postgres Row-Level Security on every table; encrypted at rest (Supabase) and TLS in transit.
- Biometric app lock, session timeouts, and an audit log of who changed what.
- Data export and account deletion for every member.

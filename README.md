# COMO - Early Task In Order With Timer & Batcher Dashboard

A Tampermonkey userscript for the Amazon Fresh COMO Operations Dashboard that helps dispatchers manage batching tasks, associates, deadlines, cart actions, and staffing more efficiently.

**Current documented release: v23.9.187**

---

## What It Does

### ⏰ Early Task In Order

Automatically keeps normal tasks organized by **Batch Target time**.

The earliest deadline always gets priority so dispatchers can focus on the carts that need attention first.

Tasks that are overdue or approaching their Batch Target are visually highlighted.

- 🟢 Enough time remaining
- 🟡 Getting close to Batch Target
- 🔴 Overdue

---

### ⏱ Time Left Column

Adds a live **Time Left** column next to Batch Target on normal task rows.

The countdown updates automatically so you can immediately see how much time remains before a task's deadline.

- 🟢 Green → more than 10 minutes left
- 🟡 Yellow → under 10 minutes
- 🔴 Red → overdue

Problem Solve, Partially Batched, and Staged for Pickup are excluded from the normal Time Left task logic.

The **Time Left** title and countdown values remain centered in the same column at different browser sizes.

---

### 🟨 P-1-H Last Known Location Highlight

Normal task rows are automatically highlighted in yellow when a package inside the task has a **Last Known Location beginning with `P-1-H`**.

The highlight:

- updates automatically when task data changes
- stays visible when you open a task and return to the dashboard
- is restored from cache during dashboard navigation
- uses paced background checks to avoid slowing down the website

Cached results appear immediately. New tasks begin checking shortly after the dashboard loads and are processed in a limited two-request queue.

---

## 🦺 Batcher Timer Dashboard

Adds a live **Batcher Timers** dashboard above the Utilization section.

The panel tracks associate batching activity and performance.

### Dashboard Tabs

- **Live** — currently active batchers with elapsed time and bags/min
- **Today** — today's batcher performance
- **Weekly** — the current Sunday-through-Saturday store week
- **Fastest** — all qualifying associates ranked by their fastest recorded performance
- **Names** — associate name lookup

The panel also includes associate search and associate performance profiles.

---

## 📊 Live Dashboard Stats

The top of the Batcher Timer panel shows:

### 🦺 Batchers

Number of associates currently batching normal tasks.

### 👥 Recommended

Recommended minimum number of batchers based on the current task workload, Batch Targets, cart count, and time remaining.

The recommendation planning cycle begins at **:57 each hour**. The normal **:55–:57** release window is handled separately so newly loading carts do not create an unnecessary staffing spike.

The recommendation is deadline-based and does not depend on individual associate speed.

### 👤 Unassigned

Number of normal tasks that do not currently have an associate assigned.

### 📋 Tasks

Current number of normal tasks on the dashboard.

### 📦 Remaining

Total remaining package workload.

---

## ⚡ Fast Stat Loading

**Batchers, Recommended, Unassigned, Tasks, and Remaining** are optimized to appear without blocking the website during a page reload.

The script can immediately show a recent store-specific snapshot while fresh dashboard data finishes loading.

Fresh data automatically replaces the temporary value as soon as the normal Tasks section is authoritative.

Performance protections include:

- store-specific cache
- limited storage writes
- reuse of already-fetched data
- no extra permanent polling loop
- no extra MutationObserver
- dashboard-only stats requests

The `+/-` staffing difference stays hidden until fresh authoritative task data is available.

---

# ▶ Run - Cart Actions

Press **▶ Run** to access dispatcher cart tools.

Each action is independent and performs only the action selected.

---

## ▶ Assign Cart

Allows a dispatcher to select one or multiple associates and automatically assign tasks to them.

Associates are assigned in the exact order selected.

Example:

1. Associate #1
2. Associate #2
3. Associate #3

The first associate receives the highest-priority eligible task, then the second associate receives the next one.

### Assignment Priority

Tasks are assigned using this order:

1. **Earliest Batch Target**
2. If multiple tasks have the same Batch Target → **most packages first**
3. If Batch Target and package count are both the same → **task displayed higher on the dashboard first**

Example:

| Batch Target | Packages | Priority |
|---|---:|---:|
| 9:50 PM | 30 | 1 |
| 10:00 PM | 59 | 2 |
| 10:00 PM | 41 | 3 |
| 10:00 PM | 25 | 4 |

The **9:50 PM task always comes first**, even though the 10:00 PM task has more packages.

Package count is only used when Batch Targets are identical.

### Eligible Task Rules

| Associate Name | Cart/s | Action |
|---|---|---|
| Blank / ASSIGNABLE | Blank | ✅ Try |
| Associate name | Blank | ✅ Try |
| Blank / ASSIGNABLE | Cart present | ✅ Try |
| Associate name | Cart present | ⛔ Skip |

If one task rejects an assignment, the script keeps the **same associate** and tries the next eligible task.

It does not advance to the next selected associate until the current associate has been assigned or there are no eligible tasks remaining.

Assignments happen directly without opening each task page.

---

## 👤 Assign Cart Name Suggestions

When **▶ Assign Cart** opens, the picker can immediately show associate suggestions before you type anything.

Suggestions are based only on associates you recently selected with **Assign Cart at that specific warehouse**.

### How suggestions work

- Shows up to **10 recent associates**
- Most recently used names appear first
- Each warehouse keeps its own recent list
- The list builds automatically as Assign Cart is used
- If there is no recent history yet, the picker asks you to search
- Typing **2 or more characters** still uses the normal full associate search
- Suggestions are **not** ranked by bags/min, speed, or performance

This keeps the suggestion system useful without guessing who should be assigned.

---

## 🚫 Assign Cart Disabled When There Are No Tasks

Assign Cart cannot be used when the normal **Tasks** section has no tasks.

The button stays disabled when:

- **Tasks = 0**
- the Tasks section is still loading and is not authoritative yet

The script also re-checks before opening the picker and again before the final Assign starts.

If tasks disappear while the picker is open, the assignment will not begin.

---

## ▶ Force Assign

Processes eligible **UNASSIGNABLE** normal carts.

The action only affects carts eligible for Force Assignment.

Problem Solve is not modified.

---

## ▶ Partially Batched

Handles eligible carts inside the **Partially Batched** section independently from normal Force Assign.

---

## ▶ Auto Complete

Attempts the site's normal **Complete Task** action on eligible tasks.

Auto Complete does not automatically Force Assign a task when completion is unavailable.

---

## ▲ Missing Package QR

Checks jobs for packages marked:

- **MISSING**
- **DAMAGED**

When found, the tool generates QR codes for:

1. The package's **Scannable ID**
2. The package's associated **CART_...** location when available

If multiple missing or damaged packages are found, the QR window supports moving between them.

This feature is read-only and does not modify the job.

Missing Package QR checks only:

- normal **Tasks**
- **Problem Solve**

It does not scan the **Partially Batched** section.

Clicking a QR tile copies that tile's ID.

---

# ⚡ Faster Cart Actions

The Run actions are optimized to move quickly while still keeping API requests sequential.

### Current pacing

- **Force Assign / Partially Batched / Auto Complete** — short safety pause between carts
- **Assign Cart** — very short pause between associates and retry steps

The script does **not** send large parallel request bursts.

One API operation completes before the next begins.

This keeps actions fast while reducing unnecessary load on the site.

---

# 🪟 Background Tab Processing

Cart actions continue processing when you switch to another browser tab.

Supported background processing:

- **Assign Cart**
- **Force Assign**
- **Partially Batched**
- **Auto Complete**

Browsers normally throttle timers in hidden tabs. The script avoids depending on those throttled timers for the action sequence.

API writes still remain sequential.

**Missing Package QR** is read-only/on-demand and does not require a background write process.

---

# 🔎 Associate Search & Autocomplete

Provides associate username/name suggestions in supported assignment fields.

Autocomplete is limited to relevant associate assignment/search fields so normal search boxes are not affected.

The older floating Associate Search panel is retired. Suggestions now appear directly inside the website's supported associate fields.

---

# 👤 Associate Performance Profiles

Click an associate's name in **Live, Today, Weekly, Fastest, Names, or search results** to open their profile.

The profile supports **Today** and **This week** and shows:

- total carts
- total packages
- average rate
- fastest rate
- current live rate
- latest cart rate
- carts, packages, and rate grouped by completion hour

Click an hour to view each cart's:

- task/cart reference
- start time
- finish time
- total time taken
- packages
- rate

All rate values use **Packages/Minute**. Duplicate or unverified records are excluded from verified totals and rates.

---

# 📅 Data Periods & Reset Behavior

- **Today** resets automatically at store midnight.
- **Weekly** uses a Sunday-through-Saturday store week and resets when the new week begins.
- **Fastest** does not reset with Today or Weekly. It changes only when qualifying performance data changes.
- Repeated API responses, sync events, and dashboard refreshes are deduplicated so the same completed cart does not inflate totals.

---

# 📐 Responsive Sizing

The dashboard, Run menu, highlighted-text QR window, Missing Package QR window, associate autocomplete, and profile window automatically fit the available browser viewport.

Scaling is recalculated on browser resize without a permanent sizing interval. Text, buttons, rows, and QR codes keep their layout and avoid wrapping or overlapping.

The dashboard can also be resized vertically by dragging its bottom handle.

---

# 📱 QR Code Tools

The script includes QR functionality for operational values.

Selected text can be displayed as a QR code for quick scanner use. The popup supports dragging and automatically fits inside the current browser viewport.

The selected-text QR closes automatically after **60 seconds**, or immediately when dismissed manually.

---

# ⚡ Performance

The script is designed to minimize impact on the COMO dashboard.

Performance optimizations include:

- dashboard UI mounts after the website's first paint
- no continuous high-frequency whole-page scanning
- targeted DOM updates
- coalesced UI rendering
- guarded network requests
- hidden-tab work reduction
- dashboard-only stats requests
- reuse of already-fetched data
- no unnecessary assignment polling
- no background automatic task-assignment monitor
- no extra permanent polling loop for Assign suggestions
- no extra permanent MutationObserver for Assign suggestions
- sequential cart-action API writes
- reduced artificial delays between cart actions
- recent-name suggestions stored locally without extra network requests
- P-1-H results cached by task
- a maximum of two paced P-1-H detail requests at once
- direct task-to-row highlight updates instead of rescanning every row after each response
- longer retry backoff after failed detail requests

The goal is to add dispatcher tools without making the normal COMO website feel slow.

---

## Features At A Glance

| Feature | Description |
|---|---|
| Early task sorting | Earliest Batch Target gets priority |
| Same-time package priority | Most packages first when Batch Targets match |
| Time Left | Live deadline countdown |
| P-1-H highlight | Yellow task highlight when Last Known Location starts with P-1-H |
| Live tab | Current batchers and elapsed time |
| Today tab | Daily associate performance |
| Weekly tab | Current Sunday-through-Saturday associate performance |
| Fastest tab | All qualifying associates ranked by fastest performance |
| Names tab | Associate lookup |
| Associate profiles | Hourly carts, packages, rates, and cart timing details |
| Batchers | Current active batcher count |
| Recommended | Deadline-based staffing recommendation |
| Unassigned | Current unassigned normal-task count |
| Tasks | Current normal-task count |
| Remaining | Remaining package workload |
| Fast stats loading | Shows recent store-specific stats while fresh data loads |
| Assign Cart | Assign selected associates to prioritized tasks |
| Assign suggestions | Recently used associates shown before typing |
| No-task safety | Assign Cart is disabled when Tasks = 0 |
| Force Assign | Process eligible UNASSIGNABLE carts |
| Partially Batched | Separate handling for partially batched carts |
| Auto Complete | Complete eligible tasks |
| Faster actions | Reduced unnecessary between-action delays |
| Background actions | Run actions continue while another browser tab is active |
| Missing/Damaged QR | QR codes for problem packages and cart locations |
| Associate autocomplete | Suggestions in supported associate fields |
| QR Generator | Create scanner-friendly QR codes |
| Click QR to copy | Click a Missing Package QR tile to copy its ID |
| Resizable panel | Resize the Batcher Timer panel |

---

# How To Install

## Step 1 — Install Tampermonkey

If you don't already have Tampermonkey, install the browser extension:

- [Firefox](https://addons.mozilla.org/en-US/firefox/addon/tampermonkey/)
- [Chrome](https://chrome.google.com/webstore/detail/tampermonkey/dhdgffkkebhmkfjojejmpbldmpobfkfo)

---

## Step 2 — Install the Script

Click the link below.

Tampermonkey should automatically open the userscript installation screen.

Click **Install**.

👉 **[Click here to install the script](https://github.com/IbrahimaSy11/-como-scripts/raw/refs/heads/main/como-early-task-in-order.user.js)**

---

## Step 3 — Open COMO

Open the Amazon Fresh COMO Operations Dashboard.

The script starts automatically.

---

# How To Use

## Task Sorting

No action is required.

Tasks are automatically kept in Batch Target order so earlier deadlines receive priority.

---

## Time Left

The Time Left column appears automatically beside Batch Target.

It updates continuously while you are viewing the dashboard.

---

## Batcher Timer Dashboard

The panel is located above the Utilization section.

You can:

- switch between **Live, Today, Weekly, Fastest, and Names**
- search for an associate
- view elapsed batching time
- view bags/min
- view Batchers, Recommended, Unassigned, Tasks, and Remaining
- click an associate name to open their performance profile
- resize the dashboard vertically
- collapse or expand the dashboard

---

## Assigning Carts

Press:

**▶ Run → ▶ Assign Cart**

Then:

1. Choose a recent suggested associate or search for a name
2. Select the associate
3. Add additional associates if needed
4. Verify their numbered order
5. Press **Assign**

The script handles task priority automatically.

If normal Tasks = 0, Assign Cart remains disabled.

---

# Compatible Pages

Primary COMO support:

- `https://como-operations-dashboard-iad.iad.proxy.amazon.com/*`

Some associate autocomplete functionality is also supported on relevant:

- `https://na.store-management.f3.amazon.dev/*`

Features activate only on pages where they are relevant.

---

# Updates

When a new version is published, Tampermonkey can detect the updated userscript.

Use Tampermonkey's update feature to install the latest version.

Current script version documented here:

**v23.9.187**

---

# Author

Built by **Ibrahim** — Amazon Fresh Dispatcher, UNY2

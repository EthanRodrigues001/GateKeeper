# Build Prompt — Apartment Visitor Security System (Dummy / No DB)

## What You Are Building

Build a complete, fully functional **dummy/prototype** for an Apartment Visitor Security & Fake Entry Detection System called **"GateKeeper"**. This is a hackathon demo — no real database, no real backend, no real notifications. All data lives in a Zustand in-memory store initialized from mock JSON. Every screen must be fully built. No placeholder screens. Every button must do something.

The system has three separate surfaces:

- **Admin Dashboard** — Next.js 16 web app (the society secretary uses this)
- **Resident App** — React Native (Expo)
- **Guard App** — React Native (Expo)

The core idea: residents share QR codes with expected visitors before arrival. For delivery agents or unknown visitors, the guard logs them at the gate and the resident gets a notification to accept or decline. All entries are logged. A risk engine flags suspicious patterns. The admin manages the society structure, assigns flats to residents, assigns guards to gates, and monitors everything from a dashboard.

---

## Tech Stack

| Layer | Technology | Version |
|---|---|---|
| Admin web app | Next.js (App Router) | 16 (latest) |
| Mobile apps | React Native + Expo | Expo SDK 52 (latest) |
| Web styling | Tailwind CSS | v4 (latest) |
| UI components | shadcn/ui | latest |
| Mobile styling | NativeWind | v4 (latest) |
| State management | Zustand | v5 (latest) |
| Charts | shadcn/ui charts (Recharts wrapper) | latest |
| QR rendering | react-native-qrcode-svg (mobile) + qrcode.react (web) | latest |
| QR scanning | expo-camera + expo-barcode-scanner (simulated) | latest SDK 52 compatible |
| Icons | @expo/vector-icons (mobile), lucide-react (web) | latest |
| Notifications | Simulated — Zustand state change + in-app banner/badge | — |
| No auth library | Just a role-select login screen that sets active user from mock data | — |

---

---

## Project Setup — Use CLI Commands, Not Manual package.json

**Do NOT write or edit `package.json` from scratch for any of the three projects.** Always scaffold using the official CLI commands below, then install additional packages on top. This ensures correct peer dependencies, config files, and project structure are generated automatically.

### 1. Admin Dashboard — Next.js 16

```bash
npx create-next-app@latest gatekeeper-admin \
  --typescript \
  --tailwind \
  --eslint \
  --app \
  --src-dir \
  --import-alias "@/*"
cd gatekeeper-admin
```

After scaffolding, install additional dependencies:

```bash
# shadcn/ui setup (run the init command — do not manually add shadcn config)
npx shadcn@latest init

# Add the specific shadcn components used in this project
npx shadcn@latest add button card badge dialog sheet table tabs select separator toast sonner

# Charts (shadcn chart components)
npx shadcn@latest add chart

# Additional packages
npm install zustand qrcode.react lucide-react recharts
```

### 2. Resident App — React Native + Expo

```bash
npx create-expo-app@latest gatekeeper-resident --template blank-typescript
cd gatekeeper-resident
```

After scaffolding, install dependencies:

```bash
# Navigation
npx expo install @react-navigation/native @react-navigation/bottom-tabs @react-navigation/stack
npx expo install react-native-screens react-native-safe-area-context

# NativeWind v4
npm install nativewind
npm install --save-dev tailwindcss@3
npx tailwindcss init

# Camera and barcode scanner
npx expo install expo-camera expo-barcode-scanner

# QR code rendering
npm install react-native-qrcode-svg react-native-svg

# State management
npm install zustand

# Icons (included in Expo SDK)
npx expo install @expo/vector-icons

# Bottom sheet
npm install @gorhom/bottom-sheet
npx expo install react-native-reanimated react-native-gesture-handler
```

### 3. Guard App — React Native + Expo

```bash
npx create-expo-app@latest gatekeeper-guard --template blank-typescript
cd gatekeeper-guard
```

Install the exact same dependencies as the Resident App above — the guard app uses the same stack.

### Shared mock data

Create a `shared/` folder at the monorepo root (or duplicate into both Expo apps if keeping them separate). Place `mockData.ts` and `store.ts` (Zustand) here. Import from this shared location in all three apps.

### Version and setup notes

- Always run `npx expo install <package>` for Expo-compatible packages — this automatically picks the version compatible with your installed Expo SDK. Do not guess version numbers for Expo packages.
- For Next.js packages, use `npm install <package>@latest` unless a specific version is already noted above.
- After installing `react-native-reanimated`, add `"react-native-reanimated/plugin"` to the `plugins` array in `babel.config.js` — this is required and frequently missed.
- NativeWind v4 requires adding the preset to `tailwind.config.js` and wrapping the app root with `<NativeWindStyleSheet.setOutput>`. Follow the official NativeWind v4 docs exactly.
- Run `npx expo start --clear` after any major dependency install to clear the Metro bundler cache.
- Do not manually create or modify `package-lock.json`. Let npm manage it.

---

## Mock Data

Create a `mockData.ts` file. Import it to initialize the Zustand store on app load.

```ts
export const mockSociety = {
  name: "Sunshine Residency",
  city: "Mumbai",
  totalWings: 3,
  gates: [
    { id: "gate_main", name: "Main Gate" },
    { id: "gate_back", name: "Back Gate" }
  ]
}

export const mockWings = [
  { id: "wing_a", name: "Wing A", floors: 4, flatsPerFloor: 4 },
  { id: "wing_b", name: "Wing B", floors: 3, flatsPerFloor: 4 },
  { id: "wing_c", name: "Wing C", floors: 5, flatsPerFloor: 4 },
]

// Auto-generate flats from wings above. Each flat ID = "A-101", "A-102", "B-201" etc.
// Status: "assigned" | "pending" | "unassigned" | "vacant"
export const mockFlats = [
  { id: "A-101", wingId: "wing_a", floor: 1, number: "A-101", status: "assigned", residentId: "r001" },
  { id: "A-102", wingId: "wing_a", floor: 1, number: "A-102", status: "assigned", residentId: "r002" },
  { id: "A-103", wingId: "wing_a", floor: 1, number: "A-103", status: "pending",  residentId: "r007" },
  { id: "A-104", wingId: "wing_a", floor: 1, number: "A-104", status: "unassigned", residentId: null },
  { id: "A-201", wingId: "wing_a", floor: 2, number: "A-201", status: "assigned", residentId: "r003" },
  { id: "A-202", wingId: "wing_a", floor: 2, number: "A-202", status: "vacant",   residentId: null },
  { id: "A-203", wingId: "wing_a", floor: 2, number: "A-203", status: "assigned", residentId: "r004" },
  { id: "A-204", wingId: "wing_a", floor: 2, number: "A-204", status: "assigned", residentId: "r005" },
  // Add remaining flats for A floor 3, 4 and all of Wing B and C following same pattern
]

export const mockResidents = [
  { id: "r001", name: "Priya Sharma",   flat: "A-101", wingId: "wing_a", phone: "+91 98765 43210", email: "priya@email.com",   status: "assigned" },
  { id: "r002", name: "Arjun Mehta",    flat: "A-102", wingId: "wing_a", phone: "+91 91234 56789", email: "arjun@email.com",   status: "assigned" },
  { id: "r003", name: "Sunita Rao",     flat: "A-201", wingId: "wing_a", phone: "+91 99887 76655", email: "sunita@email.com",  status: "assigned" },
  { id: "r004", name: "Vikram Nair",    flat: "A-203", wingId: "wing_a", phone: "+91 87654 32100", email: "vikram@email.com",  status: "assigned" },
  { id: "r005", name: "Ananya Iyer",    flat: "A-204", wingId: "wing_a", phone: "+91 93322 11000", email: "ananya@email.com",  status: "assigned" },
  { id: "r006", name: "Rohit Desai",    flat: "B-101", wingId: "wing_b", phone: "+91 90000 55555", email: "rohit@email.com",   status: "assigned" },
  { id: "r007", name: "Meena Pillai",   flat: "A-103", wingId: "wing_a", phone: "+91 99111 22333", email: "meena@email.com",   status: "pending"  },
  { id: "r008", name: "Karan Joshi",    flat: null,    wingId: null,     phone: "+91 98001 23456", email: "karan@email.com",   status: "pending"  },
]

export const mockGuards = [
  { id: "g001", name: "Ramesh Kumar",  gate: "Main Gate", gateId: "gate_main", shift: "morning", phone: "+91 91234 00000", status: "assigned", onDuty: true  },
  { id: "g002", name: "Suresh Patil",  gate: "Back Gate", gateId: "gate_back", shift: "evening", phone: "+91 90000 11111", status: "assigned", onDuty: false },
  { id: "g003", name: "Dinesh Yadav",  gate: null,        gateId: null,        shift: null,      phone: "+91 88800 99900", status: "pending",  onDuty: false },
]

export const mockVisitorPasses = [
  { id: "p001", residentId: "r001", visitorName: "Ankit Verma",    passType: "one_time",   validFrom: "2024-03-15T14:00", validUntil: "2024-03-15T20:00", used: false, revoked: false, qrValue: "GATEKEEPER|p001|r001|one_time|1710511200" },
  { id: "p002", residentId: "r002", visitorName: "Maid - Sunita",  passType: "permanent",  validFrom: "2024-01-01",       validUntil: null,               used: false, revoked: false, qrValue: "GATEKEEPER|p002|r002|permanent|0" },
  { id: "p003", residentId: "r003", visitorName: "Ravi Electrician",passType: "time_bound", validFrom: "2024-03-16T09:00", validUntil: "2024-03-16T13:00", used: false, revoked: false, qrValue: "GATEKEEPER|p003|r003|time_bound|1710586800" },
  { id: "p004", residentId: "r001", visitorName: "Old Friend Nikhil",passType:"one_time",  validFrom: "2024-03-10T10:00", validUntil: "2024-03-10T18:00", used: true,  revoked: false, qrValue: "GATEKEEPER|p004|r001|one_time|1710064800" },
  { id: "p005", residentId: "r005", visitorName: "Plumber",         passType: "one_time",  validFrom: "2024-03-14T11:00", validUntil: "2024-03-14T15:00", used: false, revoked: true,  qrValue: "GATEKEEPER|p005|r005|one_time|1710417600" },
]

export const mockVisitLogs = [
  { id: "l001", guardId: "g001", flatId: "A-101", residentId: "r001", visitorName: "Ankit Verma",      visitType: "expected", status: "allowed", entryTime: "2024-03-15T15:32", exitTime: "2024-03-15T17:10", riskScore: 5,  passId: "p001", visitorPhoto: null },
  { id: "l002", guardId: "g001", flatId: "A-102", residentId: "r002", visitorName: "Zomato Delivery",  visitType: "delivery", status: "allowed", entryTime: "2024-03-15T13:10", exitTime: null,               riskScore: 10, passId: null,   visitorPhoto: "https://i.pravatar.cc/150?img=12" },
  { id: "l003", guardId: "g001", flatId: "A-204", residentId: "r005", visitorName: "Unknown Male",     visitType: "unknown",  status: "denied",  entryTime: "2024-03-15T23:45", exitTime: null,               riskScore: 88, passId: null,   visitorPhoto: "https://i.pravatar.cc/150?img=3" },
  { id: "l004", guardId: "g002", flatId: "B-101", residentId: "r006", visitorName: "Amazon Delivery",  visitType: "delivery", status: "allowed", entryTime: "2024-03-15T11:20", exitTime: null,               riskScore: 8,  passId: null,   visitorPhoto: "https://i.pravatar.cc/150?img=15" },
  { id: "l005", guardId: "g001", flatId: "A-201", residentId: "r003", visitorName: "Maid - Sunita",    visitType: "expected", status: "allowed", entryTime: "2024-03-15T08:05", exitTime: "2024-03-15T10:30", riskScore: 2,  passId: "p002", visitorPhoto: null },
  { id: "l006", guardId: "g001", flatId: "A-203", residentId: "r004", visitorName: "Suspicious Person",visitType: "unknown",  status: "flagged", entryTime: "2024-03-14T22:50", exitTime: null,               riskScore: 75, passId: null,   visitorPhoto: "https://i.pravatar.cc/150?img=8",  flagReason: "Refused to show ID" },
  { id: "l007", guardId: "g001", flatId: "A-204", residentId: "r005", visitorName: "Unknown Woman",    visitType: "unknown",  status: "allowed", entryTime: "2024-03-14T18:20", exitTime: "2024-03-14T19:45", riskScore: 45, passId: null,   visitorPhoto: "https://i.pravatar.cc/150?img=5" },
  { id: "l008", guardId: "g001", flatId: "A-204", residentId: "r005", visitorName: "Swiggy Delivery",  visitType: "delivery", status: "allowed", entryTime: "2024-03-14T20:10", exitTime: null,               riskScore: 12, passId: null,   visitorPhoto: "https://i.pravatar.cc/150?img=20" },
]

export const mockPendingApprovals = [
  { id: "pa001", guardId: "g001", flatId: "A-101", residentId: "r001", visitorName: "Flipkart Delivery", visitType: "delivery", status: "waiting", arrivedAt: "2024-03-16T10:02", visitorPhoto: "https://i.pravatar.cc/150?img=33" },
  { id: "pa002", guardId: "g001", flatId: "A-204", residentId: "r005", visitorName: "Rahul Singh",       visitType: "unknown",  status: "waiting", arrivedAt: "2024-03-16T10:05", visitorPhoto: "https://i.pravatar.cc/150?img=7"  },
]

export const mockBlacklist = [
  { id: "b001", name: "Unknown Male",       photoUrl: "https://i.pravatar.cc/150?img=3",  scope: "society",  reason: "Attempted entry twice after denial", addedBy: "admin",  createdAt: "2024-03-15T23:50" },
  { id: "b002", name: "Suspicious Person",  photoUrl: "https://i.pravatar.cc/150?img=8",  scope: "society",  reason: "Refused to show ID at gate",         addedBy: "admin",  createdAt: "2024-03-14T23:05" },
]

export const mockRiskEvents = [
  { id: "re001", visitLogId: "l003", ruleTriggered: "odd_hours",   riskScore: 88, alertedAdmin: true,  createdAt: "2024-03-15T23:45", description: "Entry attempt at 11:45 PM" },
  { id: "re002", visitLogId: "l006", ruleTriggered: "guard_flag",  riskScore: 75, alertedAdmin: true,  createdAt: "2024-03-14T22:50", description: "Guard flagged — refused ID" },
  { id: "re003", visitLogId: "l007", ruleTriggered: "flat_spike",  riskScore: 45, alertedAdmin: false, createdAt: "2024-03-14T18:20", description: "3rd unknown visitor to A-204 this week" },
  { id: "re004", visitLogId: "l008", ruleTriggered: "odd_hours",   riskScore: 30, alertedAdmin: false, createdAt: "2024-03-14T20:10", description: "Entry after 8 PM" },
]

export const mockAnalytics = {
  dailyVolume: [
    { day: "Mon", visitors: 12 }, { day: "Tue", visitors: 18 },
    { day: "Wed", visitors: 9  }, { day: "Thu", visitors: 22 },
    { day: "Fri", visitors: 31 }, { day: "Sat", visitors: 27 },
    { day: "Sun", visitors: 14 },
  ],
  entryTypeBreakdown: [
    { type: "Expected", count: 48, color: "#3B82F6" },
    { type: "Delivery", count: 35, color: "#F59E0B" },
    { type: "Unknown",  count: 17, color: "#EF4444" },
  ],
  topFlats: [
    { flat: "A-204", count: 14 }, { flat: "B-101", count: 11 },
    { flat: "A-101", count: 9  }, { flat: "A-203", count: 8  },
    { flat: "C-302", count: 7  },
  ],
  peakHours: [9, 11, 13, 14, 18, 19, 20], // hours with highest traffic
}
```

---

## Zustand Store

Create a single `useStore.ts` Zustand store. Initialize all state from mockData. This store is shared across both mobile apps and the web app.

```ts
interface StoreState {
  // Auth
  activeUser: { role: 'admin' | 'resident' | 'guard', id: string } | null
  setActiveUser: (user) => void
  logout: () => void

  // Society structure
  society: typeof mockSociety
  wings: Wing[]
  flats: Flat[]
  addWing: (wing: Wing) => void
  updateFlat: (flatId, updates) => void
  assignFlatToResident: (flatId, residentId) => void
  markFlatVacant: (flatId) => void

  // Residents
  residents: Resident[]
  assignResident: (residentId, flatId) => void

  // Guards
  guards: Guard[]
  assignGuard: (guardId, gateId, shift) => void
  setOnDuty: (guardId, isOnDuty) => void

  // Visitor passes
  visitorPasses: VisitorPass[]
  createPass: (pass) => void
  revokePass: (passId) => void

  // Visit logs
  visitLogs: VisitLog[]
  addVisitLog: (log) => void
  flagVisitLog: (logId, reason) => void

  // Pending approvals
  pendingApprovals: PendingApproval[]
  addPendingApproval: (approval) => void
  resolveApproval: (approvalId, response: 'accepted' | 'declined') => void

  // Blacklist
  blacklist: BlacklistEntry[]
  addToBlacklist: (entry) => void
  removeFromBlacklist: (id) => void

  // Risk events
  riskEvents: RiskEvent[]
  addRiskEvent: (event) => void

  // Settings
  settings: { timeoutMinutes: number, defaultTimeoutAction: string, allowedHoursFrom: string, allowedHoursTo: string }
  updateSettings: (s) => void

  // Messages (guard ↔ resident)
  messages: Message[]
  sendMessage: (fromId, toId, text) => void
}
```

---

## Feature 1 — Notification System (Resident App)

The notification system is the core of the resident experience. When a guard logs a delivery or unknown visitor for a flat, a pending approval entry is added to Zustand. The resident's app polls Zustand and reacts immediately.

### How it appears to the resident

**Banner on home screen**: If `pendingApprovals` in Zustand contains any entry matching the logged-in resident's flat, show a persistent amber banner at the top of the home screen: `"1 visitor waiting at your gate"`. Tapping the banner navigates to the Approvals screen.

**Badge on Approvals tab**: The bottom tab "Approvals" shows a red badge with the count of pending approvals for this resident.

**In-app notification card** (shown on the Approvals screen): Each pending approval is a card showing:
- Visitor photo (from Zustand — pravatar URL for demo)
- Visitor name and type badge (Delivery = amber, Unknown = red)
- Guard name and gate
- Time ("2 min ago")
- Four action buttons:

  **Accept** (green button) — calls `resolveApproval(id, 'accepted')` in Zustand. The approval's status changes to "accepted". A new visit log entry is created with status "allowed". A green toast appears: "Entry approved". The guard's Log Visitor screen updates live (the pending card shows "Resident Accepted ✓" in green).

  **Decline** (red button) — calls `resolveApproval(id, 'declined')`. Status becomes "declined". Visit log entry created with status "denied". Red toast: "Entry declined". Guard screen shows "Resident Declined ✗".

  **Classify** (blue button, shown only for unknown visitors) — opens a bottom sheet with:
  - Text input: visitor's actual name
  - Label picker: Friend / Family / Service Worker / Other
  - Option toggle: "Create a pass for this person for future visits"
  - If toggle on: shows pass type selector (one-time / time-bound / permanent) with date/time inputs if time-bound
  - "Save & Accept" button — saves the classification to the visit log, optionally creates a new pass, then resolves the approval as accepted

  **Contact Guard** (gray button with phone icon) — opens a bottom sheet showing the on-duty guard's name and two options: "Call Guard" (shows toast "Calling Ramesh Kumar…") and "Message Guard" (opens a simple in-app message input — messages saved to Zustand `messages` array, shown as a simple chat thread)

---

## Feature 2 — Admin Society Setup with Wing & Flat Grid

This is the most complex screen in the admin dashboard. Build it carefully.

### Route: `/society-setup`

The page has two sections stacked vertically: Wings Management at the top, and the Flat Grid below it.

### Wings Management section

At the top: a row of wing cards. Each card shows the wing name, number of floors, and flats per floor. An "Add Wing" button opens a shadcn Dialog with a form:
- Wing name (text input)
- Number of floors (number input, 1–20)
- Flats per floor (number input, 1–8)
- "Save Wing" button — adds the wing to Zustand, auto-generates all flat documents for it (e.g., 4 floors × 4 flats = 16 flat documents created with IDs like "C-101" through "C-404"), closes the dialog, and the wing card appears in the row

### Flat Grid section

Below the wings row: a tab row or segmented control to select which wing to view. When a wing tab is selected, the flat grid for that wing renders.

**Grid layout**: CSS Grid where rows = floors (highest floor at the top) and columns = flat positions (1 to flatsPerFloor). Each cell is a flat card.

**Flat card appearance** (based on `flat.status` from Zustand):
- `unassigned` → light gray background, gray border, flat ID centered, "Unassigned" label in small text
- `pending` → amber background, amber border, flat ID, resident name below in small text, "Pending" badge
- `assigned` → green background, green border, flat ID, resident name below, "Assigned" badge
- `vacant` → white background with dashed border, flat ID, "Vacant" label

**Clicking a flat card** opens a shadcn Sheet (side panel) from the right with the following content:

*For unassigned flats*:
- Flat ID and wing name at the top
- "Assign Resident" section: a dropdown/select populated with all residents whose `status === "pending"` or `status === "assigned"` but whose flat is not yet set
- "Save" button → calls `assignFlatToResident(flatId, residentId)` in Zustand, updates flat status to "assigned", closes the sheet, grid cell updates to green

*For pending flats*:
- Flat ID, wing, floor info
- Resident's name, phone, email
- "Confirm Assignment" button → updates flat status to "assigned" and resident status to "assigned", shows success toast
- "Assign Different Resident" option

*For assigned flats*:
- Flat ID, wing info
- Assigned resident: name, phone, email, registration date
- Visit count for this flat (count from `visitLogs` filtered by flatId)
- Last visitor entry (most recent log entry for this flat)
- "View Full Visit History" link → navigates to `/visit-logs?flat=A-101`
- "Mark Vacant" button → confirmation dialog → calls `markFlatVacant(flatId)`, clears residentId, updates status to "vacant"

*For vacant flats*:
- "Assign Resident" form same as unassigned

**Save behavior**: All changes update Zustand immediately and reflect live in the grid without page refresh.

### Gates section (below the grid)

A simple list of gate cards. Each shows the gate name and the guards assigned to it. "Add Gate" button opens a small form. Clicking a gate card opens a sheet to view/edit the gate and reassign guards.

---

## Feature 3 — QR Code Generation (Resident App)

### Create Pass screen

Accessible from the home screen "Create Pass" button and from My Passes screen "+ New Pass" button.

This is a multi-step bottom sheet or full-screen modal:

**Step 1 — Visitor details**
- "Who is this for?" — text input for visitor name (optional, can be left as "Guest")
- Purpose label chips: Guest / Family / Delivery / Service Worker / Other (tap to select, used only for display in visit log)

**Step 2 — Pass type**
Three option cards with icons and descriptions:

- **One-time** — "Visitor can enter once. Pass expires after the first scan." Tap to select.
- **Time-bound** — "Valid within a specific date and time window you set." Shows date/time pickers for start and end when selected. Use a date picker component for the from/to fields.
- **Permanent** — "No expiry. Use for regular visitors like domestic help." Shows a warning note: "This pass will remain valid until you revoke it."

**Step 3 — Generate**
- Tap "Generate Pass" button
- The QR code renders immediately using `react-native-qrcode-svg` (or `qrcode.react` on web)
- QR value format: `GATEKEEPER|{passId}|{residentId}|{passType}|{validUntilUnixTimestamp}`
- Below the QR: pass summary card — visitor name, pass type badge, validity text ("Valid today 3 PM – 8 PM" / "Permanent" / "One-time use")
- Two buttons:
  - **Share** — simulates sharing (show a toast: "Pass link copied to clipboard")
  - **Save to My Passes** — saves the pass to Zustand and navigates to My Passes

### My Passes screen

List of all passes created by the logged-in resident from Zustand, sorted by creation date.

Each pass card shows:
- QR code thumbnail (small, rendered inline using react-native-qrcode-svg)
- Visitor name
- Pass type badge (color-coded: blue = one-time, green = permanent, purple = time-bound)
- Validity text
- Status badge: Active (green) / Used (gray) / Expired (amber) / Revoked (red strikethrough)
- "Revoke" button (only if status is Active) — confirmation dialog → calls `revokePass(passId)` in Zustand → card updates to Revoked

---

## Feature 4 — Visit Logs

### Admin route: `/visit-logs`

Full-width page showing all entries from `visitLogs` in Zustand.

**Filter bar at top (using shadcn Select and DatePicker components)**:
- Filter by wing (dropdown)
- Filter by flat (text input or dropdown)
- Filter by visit type: All / Expected / Delivery / Unknown
- Filter by status: All / Allowed / Denied / Flagged
- Date range picker: from/to
- "Clear Filters" button

**Table** (using shadcn Table component):
Columns: Date & Time | Visitor Name | Photo | Flat | Visit Type | Guard | Entry Method | Status | Risk Score | Actions

- Photo column: small circular avatar (img tag with pravatar URL, or placeholder icon if null)
- Visit Type column: colored badge — blue for Expected, amber for Delivery, red for Unknown
- Status column: colored badge — green for Allowed, red for Denied, orange for Flagged
- Risk Score column: numeric badge — green for 0–30, amber for 31–60, red for 61–100
- Actions column: "View" button (opens a shadcn Dialog with full entry details), "Flag" button (if not already flagged), "Add to Blacklist" button

**Log detail dialog** (opens on "View"):
- All fields from the visit log entry
- Visitor photo (full size if available)
- Pass details if it was a QR entry (pass type, validity)
- Risk events linked to this entry (if any)
- Timeline: entry time, approval time, exit time

**Resident route**: Residents can access their own flat's visit history from the History screen in the mobile app. Same data from Zustand, filtered to the resident's flat only. Each entry has the same detail view. "Flag Visitor" button adds to personal blocklist. "Classify" button for unknown visitors opens the classification bottom sheet.

---

## Feature 5 — Flag Alerts

### Admin route: `/alerts`

Displays all `riskEvents` from Zustand plus any `visitLogs` with `status === "flagged"`.

**Alert severity levels**:
- CRITICAL (risk score 80–100): red badge, shown at the very top
- HIGH (60–79): orange badge
- MEDIUM (30–59): amber badge
- LOW (0–29): gray badge

**Alert cards** — each card contains:
- Severity badge (CRITICAL / HIGH / MEDIUM / LOW)
- Risk score (large number on the right side of the card)
- Rule triggered label: "Odd Hours Entry" / "Guard Flagged" / "Flat Spike" / "Failed QR" / "Denied Reappearance"
- Visitor name and photo thumbnail
- Flat number and guard name
- Time ("Yesterday 11:45 PM")
- Description text (e.g., "Entry attempt at 11:45 PM, unknown visitor, denied by resident")
- Two action buttons:
  - **View Entry** — opens the visit log detail dialog
  - **Add to Blacklist** — opens a small form pre-filled with visitor name and photo, scope selector (Personal / Society), reason text input, "Confirm" button → adds to blacklist in Zustand, shows success toast, button changes to "Blacklisted ✓" (disabled)

**Filter bar**: filter by severity, date range, rule triggered.

**Guard App — Alerts tab**:
Shows only risk events related to this guard's gate. Each alert card has a "Flag Entry" button that opens a reason picker (Suspicious Behaviour / Refused ID / Repeated Appearance / Other) and calls `flagVisitLog(logId, reason)` in Zustand.

---

## Feature 6 — Admin Analytics Dashboard

### Route: `/analytics`

Built entirely with **shadcn/ui chart components** (which wrap Recharts). All data comes from `mockAnalytics` in mockData — no computation from visit logs needed.

### Layout

4 metric cards in a row at the top:
- Total Visitors This Week: 133
- Unknown Visitors: 17 (with red trend arrow)
- QR Pass Entries: 48 (with green trend arrow)
- Active Passes: 3

Below the metric cards, 4 chart sections:

**Chart 1 — Daily Visitor Volume (Bar Chart)**
- shadcn BarChart
- X-axis: days of week (Mon–Sun)
- Y-axis: visitor count
- Bars colored blue, hover tooltip showing count
- Title: "Visitor Volume — Last 7 Days"

**Chart 2 — Entry Type Breakdown (Pie / Donut Chart)**
- shadcn PieChart or use a Recharts PieChart with custom label
- Three segments: Expected (blue), Delivery (amber), Unknown (red)
- Legend below showing counts and percentages
- Title: "Entry Types This Week"

**Chart 3 — Top Flats by Visitor Count (Horizontal Bar Chart)**
- shadcn BarChart with `layout="vertical"`
- Y-axis: flat numbers (A-204, B-101, etc.)
- X-axis: visitor count
- Title: "Most Active Flats"

**Chart 4 — Peak Hours Heatmap**
- Build a simple custom grid: 1 row × 24 columns (one cell per hour, 12 AM to 11 PM)
- Cell background opacity scales with traffic volume (use mockAnalytics.peakHours to determine which hours are heavy)
- Hours 9, 11, 13, 14, 18, 19, 20 are highlighted in blue with varying intensity
- X-axis labels: "6 AM", "9 AM", "12 PM", "3 PM", "6 PM", "9 PM"
- Title: "Peak Entry Hours"

**Below charts — Recent Alerts section**:
- Last 3 risk events from Zustand shown as compact alert rows with severity badge, description, and time
- "View All Alerts" link → navigates to `/alerts`

---

## Feature 7 — Guard App

### Login screen
- Society name and "GateKeeper" logo at top
- "Select Guard" — list of guards from mockData as selectable cards (name, gate, shift badge)
- Tapping one sets `activeUser` in Zustand and navigates to the guard home

### Bottom tab navigation
- **Home** — overview and shift controls
- **Scan QR** — QR scanner with simulate buttons
- **Log Visitor** — delivery and unknown visitor entry flows
- **Today's Log** — entries processed this shift
- **Alerts** — flagged entries and risk events for this gate

### Home screen

Top section:
- Guard name, gate assigned, shift badge
- "Start Shift" / "End Shift" large toggle button — updates `onDuty` status in Zustand
- When on duty: green dot next to name, shift timer showing "On duty for 1h 23m"

Stats row (4 small cards):
- Entries Today
- Pending Approvals (with amber highlight if > 0)
- Denials Today
- Flags Raised

Pending approvals section (if any exist in Zustand for this guard's gate):
- Amber banner: "2 visitors waiting for resident approval"
- Each pending approval shown as a compact card: visitor name, flat, type badge, time, live status chip that updates when the resident responds in Zustand
- Status chip states: "Waiting…" (pulsing) → "Accepted ✓" (green) → "Declined ✗" (red)

### Scan QR screen

Top: camera viewfinder area (use expo-camera for a real live preview — no actual QR decode needed).

Below the viewfinder: three "Simulate Scan" buttons:

**"Scan: Valid Pass"** → Immediately shows a full-screen green result overlay:
```
✓ Entry Granted
Flat A-101 — Priya Sharma
Visitor: Ankit Verma
One-time Pass | Valid until 8:00 PM
```
"Log This Entry" button → adds entry to `visitLogs` in Zustand with status "allowed" and visitType "expected", shows a success toast, result overlay dismisses.

**"Scan: Expired Pass"** → Full-screen amber overlay:
```
⚠ Pass Expired
This pass was valid until March 10, 8:00 PM
Ask the visitor to contact their host for a new pass.
```
"Log Attempt" button → adds a denied log entry to Zustand.

**"Scan: Revoked / Invalid"** → Full-screen red overlay:
```
✗ Access Denied
This pass has been revoked or is invalid.
Do not allow entry.
```
"Log Attempt" button → adds a denied log entry. If the scanned pass ID is in the blacklist (simulated), also show a black warning banner: "This visitor is on the society blacklist."

### Log Visitor screen

Two large option cards with icons:

---

**Option 1 — Delivery**

Tapping "Delivery" expands a form:
1. "Flat Number" — a flat selector that opens a **mini flat grid** (same grid as admin but compact, read-only). The guard taps a flat cell to select it. Selected flat highlights in blue. Shows the resident name below once selected.
2. "Service Name" — text input with quick-select chips: Zomato / Swiggy / Amazon / Flipkart / Other
3. "Take Photo" button — opens expo-camera. Guard captures or confirms a photo. Photo shown as thumbnail. (Store as pravatar URL for demo.)
4. "Notify Resident" button (primary, full-width)

On tapping "Notify Resident":
- A pending approval entry is added to Zustand (which the resident's app will immediately show)
- The form collapses and a **waiting card** appears showing:
  - Visitor name/type, flat, resident name
  - Pulsing indicator: "Waiting for Priya Sharma (A-101)…"
  - Timer counting up from 0:00
  - Two simulation buttons: **"Simulate: Resident Accepts"** and **"Simulate: Resident Declines"**

On "Simulate: Resident Accepts":
- Waiting card turns green: "Resident Approved ✓ — Entry Allowed"
- `resolveApproval` called in Zustand
- Visit log entry added with status "allowed"
- "Done" button resets the form

On "Simulate: Resident Declines":
- Waiting card turns red: "Resident Declined ✗ — Do Not Allow Entry"
- Visit log entry added with status "denied"
- "Done" button resets the form

If the Zustand timeout (3 minutes — simulated with a 30-second demo timer) fires before a response:
- Card shows: "No Response — Apply Society Policy"
- Two buttons: "Allow Anyway" and "Turn Away"

---

**Option 2 — Unknown Visitor**

Tapping "Unknown Visitor" expands a form:
1. "Visitor Name" — text input
2. "Take Photo" — **required for unknown visitors**. Opens expo-camera. Photo captured and shown as thumbnail. A red label "Photo Required" appears if guard tries to proceed without taking a photo. (For demo: store a pravatar URL from `https://i.pravatar.cc/150?img={random}`)
3. "Flat Number" — same mini flat grid selector as delivery
4. **Blacklist check**: as soon as the visitor name is typed, do a case-insensitive fuzzy match against `blacklist` names in Zustand. If a match is found, show a red warning banner immediately below the name input: "⚠ Warning: This name matches a blacklisted individual. Proceed with caution." Include the blacklisted person's photo thumbnail and reason.
5. "Notify Resident" button — same flow as Delivery (waiting card → simulate accept/decline)

### Today's Log screen

List of all visit log entries where `guardId === activeUser.id` from Zustand, filtered to today.

Each entry row: time | visitor name | flat | type badge | status badge | "Flag" icon button

Tapping "Flag" on any non-flagged entry opens a bottom sheet:
- "Reason for flagging" — option chips: Suspicious Behaviour / Refused to Show ID / Repeated Appearance / Other
- "Other" shows a text input
- "Submit Flag" button → calls `flagVisitLog(logId, reason)` in Zustand → adds a risk event → shows toast "Entry flagged and reported to admin"

If no entries today: empty state with an icon and "No entries logged yet for this shift."

### Alerts screen

List of `riskEvents` filtered to this guard's gateId. Same card layout as admin alerts but simpler — no "Add to Blacklist" button, only "View Entry" and "Flag Entry". Shows last 20 events sorted by time descending.

---

## UI & Design Guidelines

### Color system (consistent across all three apps)

| State | Color | Use for |
|---|---|---|
| Success / Allowed | Green `#22C55E` | Entry allowed, on-duty, active pass, assigned flat |
| Warning / Pending | Amber `#F59E0B` | Pending approval, pending assignment, time-bound |
| Danger / Denied | Red `#EF4444` | Entry denied, flagged, revoked, blacklisted |
| Info / Expected | Blue `#3B82F6` | QR expected visitors, passes, information |
| Neutral | Gray `#9CA3AF` | Unassigned, off-duty, vacant |

### Admin dashboard (Next.js + shadcn)
- Use shadcn Card, Badge, Button, Dialog, Sheet, Table, Select, Tabs, Separator throughout
- Sidebar is fixed, 240px wide. Main content area is scrollable
- All tables use shadcn Table with sortable column headers
- All modals use shadcn Dialog or Sheet (side panel for flat details)
- Charts use shadcn chart components (Bar, Pie, horizontal Bar)
- Use shadcn Toast (Sonner) for all success/error messages
- Dark mode support using shadcn's built-in theming

### Mobile apps (React Native + NativeWind)
- Bottom tab navigator with 5 tabs
- Use bottom sheets (react-native-bottom-sheet or a simple animated View) for detail panels
- Use @expo/vector-icons for all icons
- All list items should have a right chevron or action icon
- All destructive actions (Decline, Revoke, Mark Vacant) require a confirmation alert before executing
- Show a proper empty state (icon + message) for every list that can be empty
- All timestamps displayed as relative: "just now", "5 min ago", "Today 3:45 PM", "Yesterday"

---

## Screens Checklist

### Admin Dashboard (Next.js) — 10 routes

- [ ] `/login` — role select, "Enter as Admin" button
- [ ] `/dashboard` — metrics, recent activity, quick actions
- [ ] `/society-setup` — wings management + flat grid + gates section
- [ ] `/residents` — assigned + pending tabs, assign flat modal
- [ ] `/guards` — assigned + pending tabs, assign gate modal
- [ ] `/visit-logs` — filterable table with full log detail dialog
- [ ] `/alerts` — risk event cards with flag + blacklist actions
- [ ] `/analytics` — 4 shadcn charts + metric cards + recent alerts
- [ ] `/blacklist` — blacklisted person cards + add/remove
- [ ] `/settings` — timeout policy, allowed hours, save

### Resident App (React Native) — 8 screens

- [ ] Login / role select
- [ ] Home (pending approval banner, recent visits)
- [ ] Create Pass (multi-step: visitor name → pass type → QR display)
- [ ] My Passes (list with QR thumbnails, revoke)
- [ ] Approvals (cards with Accept / Decline / Classify / Contact Guard)
- [ ] History (filterable log, entry detail, classify unknown, flag visitor)
- [ ] Contact Security (guard details, call/message)
- [ ] Profile (flat info, blocklist, notification prefs, sign out)

### Guard App (React Native) — 7 screens

- [ ] Login / role select
- [ ] Home (shift toggle, stats, pending approvals with live status)
- [ ] Scan QR (camera preview + 3 simulate buttons + result overlay)
- [ ] Log Visitor (delivery flow + unknown visitor flow with flat grid picker)
- [ ] Today's Log (filterable list, flag entry)
- [ ] Alerts (risk events for this gate)

---

## Critical Implementation Notes

1. **No database, no backend, no API calls.** Everything is Zustand. When the guard taps "Notify Resident", a `pendingApprovals` entry is added to Zustand. The resident's app reads from the same Zustand store. When the resident taps Accept, `resolveApproval` updates Zustand and the guard's screen reacts. This simulates real-time communication.

2. **The flat grid is the most complex component.** Build it as a CSS Grid in Next.js (`grid-template-columns: repeat(flatsPerFloor, 1fr)`). Each cell is a div with conditional `className` based on `flat.status`. Rows go from highest floor to lowest (reverse the floors array before rendering). Clicking a cell opens a shadcn Sheet sliding in from the right. The Sheet reads the flat's data from Zustand and renders the correct content (unassigned / pending / assigned / vacant states). "Save" in the Sheet calls the appropriate Zustand action and the grid updates without a page reload.

3. **QR codes are visual only.** Use `react-native-qrcode-svg` on mobile and `qrcode.react` on web to render a QR SVG from the `qrValue` string stored in the pass. The QR is never actually decoded — scanning is simulated by preset buttons. The QR is a visual prop for the demo.

4. **Photos are pravatar.cc URLs.** When the guard taps "Take Photo", open `expo-camera` for the real camera experience, but on capture, store `https://i.pravatar.cc/150?img=${Math.floor(Math.random()*70)}` as the photo URL. This is sufficient for the demo.

5. **Pending approval live status on guard screen.** After the guard taps "Notify Resident" and the waiting card appears, the status chip updates automatically because it reads `pendingApprovals[id].status` from Zustand. When the resident taps Accept (updating Zustand), the guard's chip changes from "Waiting…" to "Accepted ✓" without any manual refresh.

6. **Charts must all render with data.** Use the `mockAnalytics` object directly as chart data. Do not attempt to compute analytics from `visitLogs` — the mock analytics data is pre-built. Pass it directly to the shadcn chart components.

7. **The blacklist name check in the guard app** is a simple `blacklist.some(b => logName.toLowerCase().includes(b.name.toLowerCase()))` check run on every keystroke in the visitor name input. Show the warning banner immediately if matched.

8. **Every button must do something.** If a feature isn't fully wired, at minimum show a toast: "Coming soon" or a mock confirmation. No silent buttons.

9. **Build all screens fully.** Do not create placeholder screens. The hackathon demo will navigate through every screen — every route must render real UI with real mock data.

10. **Never write `package.json` from scratch.** Always scaffold with `npx create-next-app@latest` for the web app and `npx create-expo-app@latest` for mobile apps, then install extra packages on top using `npm install` or `npx expo install`. Writing `package.json` manually causes peer dependency conflicts and broken configs. If a dependency already exists in the scaffolded project (e.g., React, TypeScript), do not re-add it.
# Comprehensive Survey & Architectural Analysis: Fast POS & Live Orders Kanban

**Author**: Explorer 3 (Sushi Dark Kitchen Order Control System Survey Team)  
**Date**: 2026-08-26  
**Scope**: 
1. **R3**: Fast POS Order Creation Screen (`/orders/new`)
2. **R4**: Live Orders Board & Kitchen Kanban with Prep Timers (`/orders`)

---

## 1. Executive Summary

This investigation analyzed the frontend implementation, state management, API routes, database schemas, and state machine transitions governing order creation (`/orders/new`) and live order tracking (`/orders`).

### Key Findings Overview
1. **Fast POS Screen (`/orders/new`)**:
   - The current `OrderForm` (`src/components/orders/order-form.tsx`, 1,095 lines) is functional but operates as a generic software cart rather than a high-speed POS system.
   - The receipt/cart panel lacks dedicated thermal-ticket visual metaphors, monospace tabular column locks, and quick-clear ergonomics.
   - Payment method selectors are small (`h-8`, text-xs) buttons with low tactile feedback, failing fast touch-target requirements (min 44×44px).
   - Customer lookup is debounced (300ms) and computes lifetime orders, but displays only a basic badge rather than tiered dark-kitchen loyalty badging (Regular, VIP, Gold, Platinum).
2. **Live Orders Board & Kitchen Kanban (`/orders`)**:
   - The current `OrdersTable` (`src/components/orders/orders-table.tsx`, 1,067 lines) only provides a tabular view and card list.
   - There is no View Switcher or Kanban Board mode.
   - The backend schema (`prisma/schema.prisma`: `Order.preparingAt`) and service (`src/services/orders.ts`: `transitionOrderStatus`) record the exact timestamp when status enters `PREPARING`, but the frontend `OrderRowItem` interface does not expose `preparingAt`, preventing live elapsed preparation timer calculation and pulsing delay warnings (>15m).
   - State machine rules in `src/lib/orderStateMachine.ts` (`ALLOWED_TRANSITIONS`) strictly enforce `NEW -> CONFIRMED -> PREPARING -> READY -> OUT_FOR_DELIVERY -> DELIVERED`. The Kanban board must strictly adhere to these transition sequences and intercept driver assignments when moving to `OUT_FOR_DELIVERY`.

---

## 2. Fast POS Order Creation Screen (`/orders/new` — R3)

### 2.1 Current File & Component Structure
- **Server Entry**: `src/app/[locale]/(dashboard)/orders/new/page.tsx` (Lines 1–40)
  - Enforces `requirePageUser()`.
  - Concurrently fetches `listBrands()`, `listPlatforms()`, `listDeliveryZones()`, and `listDeliveryDrivers()` via `Promise.all`.
  - Renders `<OrderForm>`.
- **Client Container**: `src/components/orders/order-form.tsx` (Lines 1–1095)
  - Manages monolithic state: brand selection, menu fetching, customer debouncing, cart array, delivery zone, driver, payment method, discounts, and order notes.

### 2.2 Customer Lookup & Loyalty Engine
- **Lookup Mechanism**:
  - `useEffect` on `customerPhone` (Lines 211–241) triggers a debounced call to `/api/customers/search?q=${encodeURIComponent(cleanPhone)}` after 300ms when phone length $\ge 3$.
  - Handled by `searchCustomersByPhone` in `src/services/customers.ts` (Lines 58–72), returning up to 10 customers ordered by `lastOrderAt desc` with fields `{ id, name, phone, address, totalOrders }`.
- **Current Loyalty Display**:
  - `order-form.tsx` (Lines 591–603) displays:
    ```tsx
    {selectedCustomerMeta ? (
      <Badge variant="outline" className="text-emerald-600 border-emerald-300 dark:text-emerald-400">
        <User className="size-3 me-1" />
        {t("returningCustomer", { count: selectedCustomerMeta.totalOrders })}
      </Badge>
    ) : customerPhone.trim().length >= 5 ? (
      <Badge variant="outline" className="text-amber-600 border-amber-300 dark:text-amber-400">
        <Plus className="size-3 me-1" />
        {t("newCustomer")}
      </Badge>
    ) : null}
    ```
- **Recommended Enhanced Loyalty Architecture**:
  - Introduce structured loyalty tiers based on `totalOrders`:
    | Tier | Orders Count | Visual Signature | Badge Label (AR/EN) | Perks / Context |
    |---|---|---|---|---|
    | **New** | 0 | Amber subtle border / Sky badge | عميل جديد / New Guest | First-time customer |
    | **Regular** | 1 – 4 | Emerald soft badge (`bg-emerald-500/10 text-emerald-600`) | عميل متكرر / Regular | Repeat sushi buyer |
    | **Gold VIP** | 5 – 9 | Gold metallic gradient (`bg-amber-500/15 text-amber-600 border-amber-500/30`) | عميل مميز (VIP) / Gold VIP | High-frequency buyer |
    | **Platinum** | 10+ | Royal purple glow (`bg-purple-500/15 text-purple-600 border-purple-500/40 shadow-xs`) | عميل بلاتيني / Platinum Legend | Top dark-kitchen patron |
  - Add quick action: "Auto-fill previous address" if customer address exists.

### 2.3 Pricing Calculation Pipeline & Financial Integrity
- **Calculations in `order-form.tsx`** (Lines 312–342):
  - `subtotal = cart.reduce((sum, item) => sum + item.price * item.quantity, 0)`
  - `rawDeliveryFee = selectedZone ? selectedZone.fee : 0`
  - `netDeliveryFee = (driver?.type === "APP" || driver?.type === "PICKUP") ? 0 : rawDeliveryFee`
  - `numericDiscount = parseFloat(discountAmount) || 0`
  - `grandTotal = Math.max(0, subtotal - numericDiscount + netDeliveryFee)`
- **Conformance**:
  - Matches backend `calculateOrderTotals` in `src/lib/orderStateMachine.ts` (Lines 74–91) and `createOrder` in `src/services/orders.ts` (Lines 124–130).
  - Discount reason validation: If `numericDiscount > 0`, `discountReason` is mandatory; automatically tags discount as `APPROVED` if role is `OWNER` or `MANAGER`, and `PENDING` if role is `CASHIER`.

### 2.4 Interactive Receipt Ticket Preview Panel Design (`tabular-nums`)
- **Current Problem**:
  - The right column uses a generic card (`CardContent`, Lines 740–1036).
  - Price alignment lacks fixed-width character columns; numbers jitter as cart items change.
  - Cashiers lack a printed slip preview to verify before pressing "Create Order".
- **Recommended Receipt Ticket Architecture**:
  - **Component**: `src/components/orders/pos/receipt-ticket-preview.tsx`
  - **Visual Design**:
    - Thermal paper aesthetic: crisp container with subtle top/bottom jagged edge styling (CSS clip-path or SVG zig-zag), paper texture background token in light mode (`bg-zinc-50 dark:bg-zinc-900/90`), border-dashed dividers (`border-zinc-300 dark:border-zinc-700`).
    - Monospace Header:
      - Japanese dark kitchen brand signature banner (e.g. `🍣 FLOWER SUSHI • POS RECEIPT`).
      - Dynamic timestamp preview (`HH:mm:ss DD/MM/YYYY`).
      - Cashier identifier & Order type pill (Dine-in / Delivery / Pickup / App).
    - Tabular Itemized Grid:
      - Class `font-mono tabular-nums` on all quantities, unit prices, and line totals.
      - Grid header: `[Qty] [Item Description] [Total]` with right-aligned currency tokens.
      - Quick item modification buttons (increase/decrease/delete) appear on hover/touch without disrupting the receipt layout.
    - Financial Breakdown:
      - Dashed separator (`border-t border-dashed`).
      - Subtotal, Net Delivery fee, Discount (strikethrough & badge), and High-contrast Grand Total in extra-large bold font.
    - Footer Actions:
      - Sticky bottom action bar with `1-Click Submit` showing live total, `Clear Ticket`, and optional `Draft Print`.

### 2.5 Touch-Optimized Payment Selector Architecture
- **Current State** (`order-form.tsx`, Lines 876–912):
  - 3 small buttons (`h-8`, text-xs) in a 3-column row: Cash, Visa, Online.
- **Enhanced Design**:
  - **Component**: `src/components/orders/pos/payment-selector.tsx`
  - Ergonomics: Minimum height 52px (touch target $> 48\times 48\text{px}$), bold icons, subtitle, tactile active state:
    - **Cash (نقدي)**: `Banknote` icon, emerald accent active border (`border-emerald-500 bg-emerald-500/10`).
    - **Visa / Card (فيزا)**: `CreditCard` icon, indigo accent active border (`border-indigo-500 bg-indigo-500/10`).
    - **Online / Prepaid (أونلاين)**: `Globe` / `QrCode` icon, violet accent active border (`border-purple-500 bg-purple-500/10`).
  - Keyboard Shortcuts: Key `1` for Cash, Key `2` for Visa, Key `3` for Online during POS entry.

---

## 3. Live Orders Board with Kitchen Kanban & Prep Timers (`/orders` — R4)

### 3.1 Current Orders Listing & Filter State
- **Server Entry**: `src/app/[locale]/(dashboard)/orders/page.tsx` (Lines 1–32)
  - Fetches `listBrands()` and `listPlatforms()` concurrently.
  - Renders `<OrdersTable>`.
- **Client Container**: `src/components/orders/orders-table.tsx` (Lines 1–1067)
  - Filters: `activeTab` (`all`, `active`, `delivered`, `cancelled`), `selectedBrandId`, `selectedPlatformId`, `selectedDate`, `searchQuery` (debounced 300ms).
  - Polling: Auto-refresh every 15 seconds (`setInterval`) with manual toggle.
  - State transitions: 1-click status advancement via `handleAdvanceStatus` calling `PATCH /api/orders/${order.id}/status`.
  - Intercepts `READY -> OUT_FOR_DELIVERY` to enforce driver assignment if `order.driver` is null.

### 3.2 View Switcher Architecture (Table View vs. Kitchen Kanban)
- **State Definition**:
  ```ts
  type OrdersViewMode = "table" | "kanban";
  const [viewMode, setViewMode] = useState<OrdersViewMode>("table");
  ```
- **Sync & Persistence**:
  - Sync with URL query parameter `?view=table` or `?view=kanban` via `useSearchParams` and `useRouter`.
  - Preserves all filter states (`selectedBrandId`, `selectedPlatformId`, `selectedDate`, `debouncedSearch`) seamlessly when toggling views.
- **UI Control**:
  - Segmented control in the top toolbar:
    - Table Icon + "جدول الطلبات / Table"
    - Kanban / LayoutGrid Icon + "لوحة المطبخ / Kanban"

### 3.3 Kanban Column Workflow & State Machine Conformance
- **Strict Transition Rules (`src/lib/orderStateMachine.ts`, Lines 7–16)**:
  ```ts
  ALLOWED_TRANSITIONS:
    NEW              -> [CONFIRMED, CANCELLED]
    CONFIRMED        -> [PREPARING, CANCELLED]
    PREPARING        -> [READY, CANCELLED]
    READY            -> [OUT_FOR_DELIVERY, CANCELLED]
    OUT_FOR_DELIVERY -> [DELIVERED]
    DELIVERED        -> []
    CANCELLED        -> []
  ```
- **Kanban Column Mapping**:
  To fulfill the user requirement `(NEW -> PREPARING -> READY -> OUT_FOR_DELIVERY -> DELIVERED)` while strictly upholding `orderStateMachine.ts`:
  1. **Column 1: `NEW & CONFIRMED` (الوارد والمؤكد)**:
     - Shows `NEW` orders with action "تأكيد وبدء التحضير" (or 2-step prompt).
     - Moving an order to `PREPARING` automatically completes `NEW -> CONFIRMED -> PREPARING` via sequential API calls or handles `CONFIRMED` orders seamlessly.
     *(Alternatively: 5 discrete workflow stages with `CONFIRMED` grouped in Column 1 or as an explicit mini-stage).*
  2. **Column 2: `PREPARING` (قيد التحضير)**:
     - Active kitchen cooking/assembly station.
     - Houses live prep timers and overdue warning badges.
     - Primary action: "جاهز للتسليم / Ready".
  3. **Column 3: `READY` (جاهز للتسليم)**:
     - Packed orders awaiting driver pickup.
     - If driver assigned: action is "خروج للتوصيل / Dispatch".
     - If driver NOT assigned: clicking action or dragging to Out for Delivery opens `AssignDriverDialog`.
  4. **Column 4: `OUT_FOR_DELIVERY` (في الطريق)**:
     - Orders currently on the road with driver.
     - Primary action: "تم التسليم / Mark Delivered".
  5. **Column 5: `DELIVERED` (تم التسليم)**:
     - Completed orders for the active shift.
- **Cancelled Orders**:
  - Shown in a collapsible drawer or filtered tab so they don't clutter the active kitchen board.

### 3.4 Elapsed Prep Timer Architecture & Pulsing Visual Warnings
- **Backend Timestamp Availability**:
  - In `prisma/schema.prisma` (Line 199), the `Order` model has `preparingAt DateTime?`.
  - In `src/services/orders.ts` (Lines 232–237), `transitionOrderStatus` automatically records `preparingAt = new Date()` when entering `OrderStatus.PREPARING`.
  - `listOrders` (`src/services/orders.ts`, Lines 405–420) already selects all root `Order` fields, meaning `preparingAt` is sent over the wire in `/api/orders`.
- **Frontend DTO Update**:
  - Update `OrderRowItem` in `src/components/orders/orders-table.tsx` to include:
    ```ts
    preparingAt?: string | null;
    confirmedAt?: string | null;
    readyAt?: string | null;
    outForDeliveryAt?: string | null;
    deliveredAt?: string | null;
    ```
- **Timer Hook (`usePrepTimer`)**:
  ```ts
  export function usePrepTimer(preparingAt?: string | null, createdAt?: string) {
    const [now, setNow] = useState<number>(Date.now());

    useEffect(() => {
      const interval = setInterval(() => setNow(Date.now()), 1000);
      return () => clearInterval(interval);
    }, []);

    const startTime = preparingAt ? new Date(preparingAt).getTime() : new Date(createdAt || 0).getTime();
    const elapsedSeconds = Math.max(0, Math.floor((now - startTime) / 1000));
    const elapsedMinutes = Math.floor(elapsedSeconds / 60);
    const secondsRemainder = elapsedSeconds % 60;
    const formatted = `${String(elapsedMinutes).padStart(2, "0")}:${String(secondsRemainder).padStart(2, "0")}`;

    const isWarning = elapsedMinutes >= 10 && elapsedMinutes < 15;
    const isCritical = elapsedMinutes >= 15; // > 15 minutes trigger

    return { elapsedMinutes, formatted, isWarning, isCritical };
  }
  ```
- **Visual Alert Indicators**:
  - **Normal ($< 10$ min)**: Soft emerald/neutral timer badge: `⏱️ 04:32`.
  - **Warning ($10 - 15$ min)**: Amber warning badge: `⏳ 12:45` with amber border.
  - **Critical Alert ($> 15$ min)**:
    - High-contrast pulsing red badge:
      ```tsx
      <div className="flex items-center gap-1.5 rounded-full bg-red-500/15 border border-red-500/40 px-2.5 py-1 text-xs font-bold text-red-600 dark:text-red-400 animate-pulse">
        <span className="relative flex size-2">
          <span className="animate-ping absolute inline-flex h-full w-full rounded-full bg-red-400 opacity-75" />
          <span className="relative inline-flex rounded-full size-2 bg-red-500" />
        </span>
        <Flame className="size-3.5" />
        <span className="tabular-nums font-mono">{formatted}</span>
        <span className="text-[10px] font-semibold uppercase">تأخير (+15m)</span>
      </div>
      ```
    - Pulsing Card Border: `ring-2 ring-red-500/50 bg-red-500/5` for immediate kitchen visibility.

### 3.5 Drag-and-Drop vs Tactile 1-Click Status Transitions
- **Dual-Interaction Strategy**:
  1. **HTML5 Native Drag-and-Drop**:
     - Drag card from one column to the next.
     - Drop target highlights column with dashed colored ring.
     - On drop: validates against `assertTransition` and executes status update.
  2. **Touch-Optimized 1-Click Action Buttons**:
     - Critical for kitchen staff with wet/gloved hands who cannot perform drag gestures.
     - Large tactile advance button on the card footer (e.g. `[ ▶️ بدء التحضير ]`, `[ ✅ جاهز للتسليم ]`, `[ 🚚 إرسال مع الطيار ]`).
     - Includes instant loading spinner (`Loader2`) to prevent double-clicks.

---

## 4. Brand & Platform Visual Signatures

### 4.1 Four Sushi Brands Palette
To ensure cashiers and kitchen chefs can instantly distinguish orders at a glance:

| Brand Name | Japanese Signature | Theme Color | CSS / Tailwind Tokens |
|---|---|---|---|
| **Flower** | 🌸 フラワー (Floral Sushi) | Soft Rose / Cherry Pink | `bg-rose-500/15 text-rose-700 dark:text-rose-300 border-rose-500/30` |
| **Mastery** | 🗡️ 職人 (Sushi Mastery) | Deep Obsidian / Gold | `bg-amber-500/15 text-amber-800 dark:text-amber-300 border-amber-500/40` |
| **Niwa** | 🌿 庭 (Sushi Garden) | Fresh Wasabi / Matcha Emerald | `bg-emerald-500/15 text-emerald-700 dark:text-emerald-300 border-emerald-500/30` |
| **Tobiko** | 🍣 飛子 (Flying Fish Roe) | Vivid Coral / Tobiko Orange | `bg-orange-500/15 text-orange-700 dark:text-orange-300 border-orange-500/30` |

### 4.2 Five Platforms Palette
| Platform Name | Icon / Badge | Brand Color | CSS / Tailwind Tokens |
|---|---|---|---|
| **Talabat** | 🛵 Talabat | Vivid Orange | `bg-orange-600/15 text-orange-600 dark:text-orange-400 border-orange-600/30` |
| **elmenus** | 🍽️ elmenus | Crimson Red | `bg-red-600/15 text-red-600 dark:text-red-400 border-red-600/30` |
| **InstaShop** | 🛒 InstaShop | Fresh Cyan / Teal | `bg-teal-500/15 text-teal-600 dark:text-teal-400 border-teal-500/30` |
| **HarryApp** | ⚡ HarryApp | Indigo / Electric Blue | `bg-indigo-500/15 text-indigo-600 dark:text-indigo-400 border-indigo-500/30` |
| **Phone / Direct** | 📞 هاتف الفرع | Neutral Slate / Bronze | `bg-slate-500/15 text-slate-700 dark:text-slate-300 border-slate-500/30` |

---

## 5. File Boundaries & Modular Architecture Recommendations

```
src/
├── components/
│   ├── orders/
│   │   ├── pos/
│   │   │   ├── receipt-ticket-preview.tsx    # R3: Thermal slip receipt panel with tabular-nums
│   │   │   ├── payment-selector.tsx          # R3: Touch-optimized 44px+ payment cards
│   │   │   ├── customer-loyalty-badge.tsx    # R3: Tiered loyalty badges (Regular/VIP/Legend)
│   │   │   └── product-pos-card.tsx          # R3: Fast-add product tile with quantity counter
│   │   ├── views/
│   │   │   ├── orders-data-table.tsx         # R4: Refactored clean Table & mobile cards view
│   │   │   ├── kitchen-kanban-board.tsx      # R4: Multi-column Live Kitchen Board container
│   │   │   ├── kanban-column.tsx             # R4: Individual column with count & drop handlers
│   │   │   └── kanban-order-card.tsx         # R4: High-visibility order card with live prep timer
│   │   ├── order-form.tsx                    # R3: Enhanced POS form orchestrator
│   │   └── orders-table.tsx                  # R4: View-switcher dashboard orchestrator
│   └── ui/
│       └── brand-badge.tsx                   # R2/R3/R4: Unified Brand & Platform badge component
└── lib/
    ├── brandStyles.ts                        # Shared visual signature definitions
    └── hooks/
        └── usePrepTimer.ts                   # Reusable live timer with >15m pulsing alert
```

---

## 6. Verification and Testing Method

1. **Static Analysis & Type Checking**:
   - `npm run typecheck` (`tsc --noEmit`) — must pass with 0 errors.
   - `npm run lint` — must pass with 0 errors.
2. **Build Verification**:
   - `npm run build` — Turbopack compilation must succeed across all localized dashboard routes.
3. **Behavioral Acceptance Tests**:
   - **R3**: Navigate to `/orders/new`, search customer `01012345678`, select brand/platform, add products. Verify receipt ticket updates instantly with aligned `tabular-nums`. Click Cash/Visa/Online touch buttons. Confirm order and verify receipt summary.
   - **R4**: Navigate to `/orders`, toggle between Table View and Kanban Board. Verify active filters persist. In Kanban mode, verify 5 columns. Check that an order in `PREPARING` displays a live ticking timer. If preparation $> 15$ min, verify red pulsing warning badge. Drag order or click 1-click status advance and verify backend state updates.

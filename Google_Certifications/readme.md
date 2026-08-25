# Woodcraft App — Full Code Explanation

> ⚠️ Note: `Billing.tsx` and `Index.tsx` were referenced in your upload but their
> file contents were not actually available in this session (empty on disk).
> Everything else — 18 files — is fully explained below.

This app is a React + TypeScript + Vite + Tailwind admin/customer/employee
portal for a woodworking business ("The Atelier"). It uses `react-router-dom`
for routing, `sonner` for toast notifications, `recharts` for charts, and a
`dev` mock-data mode (`import.meta.env.DEV`) that lets the UI run against
fake in-memory data before a real backend exists.

---

## Part 1 — File Descriptions (Overview)

| File | Role |
|---|---|
| **AdminProfile.tsx** | Admin's own account page: view/edit profile, change password, logout. |
| **CustomerProfile.tsx** | Customer's own account page: view/edit profile, change password, logout. |
| **CustomerOverview.tsx** | Customer's dashboard "home": stats, commission request dialog, order/payment/invoice tables. |
| **Customers.tsx** | Admin-facing directory of all customers as cards, with computed spend per customer. |
| **Dashboard.tsx** | Router component: picks which dashboard (Admin/Customer/Employee) to render based on logged-in user's role. |
| **EmployeeProfile.tsx** | Employee's own account page: view/edit profile (incl. hourly rate, bank account), change password. |
| **EmployeeOverview.tsx** | Employee's dashboard "home": task stats, recent tasks, worklogs, wage summary. |
| **Employees.tsx** | Admin-facing directory ("Artisans") of employees as cards with task/hour/wage stats. |
| **Inventory.tsx** | Materials/stock management page: list materials, low-stock warnings, add material (admin), update stock quantity (admin). |
| **Login.tsx** | Login screen with demo account quick-fill buttons and a marketing side-panel. |
| **NotFound.tsx** | Generic 404 page, logs the bad path to console. |
| **Orders.tsx** | Central order management table: search/filter, view order detail modal, employee status-update modal, customer cancel-order modal. |
| **Register.tsx** | Sign-up form (customer/employee/admin), shows a "submitted"/"welcome" confirmation screen after submit. |
| **Reports.tsx** | Admin analytics page: 4 charts (monthly income/expenses, order completion pie, top materials bar, artisan productivity bar). |
| **UserAccess.tsx** | Admin user-management console: pending signup requests (approve/reject), active users (block), blocked users (unblock), rejected/deleted log. |
| **WageSummary.tsx** | Employee's own wage history table (hardcoded demo months) with totals. |
| **Worklog.tsx** | Time-tracking page: table of logged hours/wages per order, per-order summary table, "Add worklog entry" modal. |
| **AdminOverview.tsx** | Admin dashboard "home": KPI stat cards, revenue line chart, order-status bar chart, low-stock list, recent activity feed. |

---

## Part 2 — Line-by-Line Explanations

---

### 1. `AdminProfile.tsx`

```tsx
import { useState } from "react";
```
Imports React's `useState` hook for local component state.

```tsx
import { useAuth } from "@/lib/auth";
```
Imports a custom hook that exposes the logged-in `user` object and auth actions (`login`, `logout`, `register`) from a shared auth context/store.

```tsx
import { useNavigate } from "react-router-dom";
```
Imports the router hook used to programmatically redirect the browser (e.g., after logout).

```tsx
import { AppShell, PageHeader } from "@/components/layout/AppShell";
```
Imports layout wrapper components: `AppShell` provides the page chrome (sidebar/topbar), `PageHeader` renders a consistent page title block.

```tsx
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
```
Design-system form primitives (shadcn/ui-style wrappers around native elements).

```tsx
import { toast } from "sonner";
```
Imports the toast-notification function used to show success/error popups.

```tsx
import { Eye, EyeOff } from "lucide-react";
```
Icon components used to toggle password visibility.

```tsx
type Section = "view" | "edit" | "password";
```
A union type restricting the tab state to exactly 3 valid values — used for type-safety on the tab switcher.

```tsx
export default function AdminProfile() {
```
Declares and default-exports the `AdminProfile` React component.

```tsx
  const { user, logout } = useAuth();
```
Pulls the current `user` object and the `logout` function from the auth hook.

```tsx
  const nav = useNavigate();
```
Gets the navigate function so the component can redirect after logout.

```tsx
  const [activeSection, setActiveSection] = useState<Section>("view");
```
State holding which tab is currently active; defaults to `"view"`.

```tsx
  const [showCurrentPassword, setShowCurrentPassword] = useState(false);
  const [showNewPassword, setShowNewPassword] = useState(false);
  const [showConfirmPassword, setShowConfirmPassword] = useState(false);
```
Three independent booleans controlling whether each of the three password fields shows plain text or dots.

```tsx
  const [loading, setLoading] = useState(false);
```
Tracks whether an async save/update action is in flight, used to disable buttons and change their label.

```tsx
  // Edit form state
  const [editForm, setEditForm] = useState({
    fullName: user?.name || "Ahmed Khan",
    username: "ahmed_admin",
    email: user?.email || "admin@woodcraft.com",
    contactNumber: "+92-300-0000000",
  });
```
Initializes the editable profile fields. Uses the real logged-in user's `name`/`email` if present, otherwise falls back to hardcoded demo values (`Ahmed Khan`, `admin@woodcraft.com`) — this is placeholder/mock data since there's no backend field for username/contact yet.

```tsx
  // Password form state
  const [passwordForm, setPasswordForm] = useState({
    current: "",
    new: "",
    confirm: "",
  });
```
Empty initial state for the password-change form's three fields.

```tsx
  const handleEditChange = (field: string, value: string) => {
    setEditForm(prev => ({ ...prev, [field]: value }));
  };
```
Generic setter: spreads the previous `editForm` object and overwrites one key (`field`) with the new `value`. Used by every input's `onChange`.

```tsx
  const handlePasswordChange = (field: string, value: string) => {
    setPasswordForm(prev => ({ ...prev, [field]: value }));
  };
```
Same pattern as above but for the password form.

```tsx
  const handleSaveProfile = async (e: React.FormEvent) => {
    e.preventDefault();
```
Form submit handler; `e.preventDefault()` stops the browser's default full-page-reload form submission.

```tsx
    setLoading(true);
    try {
      // TODO: Call API to update admin profile
      toast.success("Profile updated successfully");
      setActiveSection("view");
    } catch (err: any) {
      toast.error(err.message || "Failed to update profile");
    } finally {
      setLoading(false);
    }
  };
```
Sets `loading` true, then (since there's no real API call yet — see the `TODO`) immediately shows a success toast and switches back to the "view" tab. `catch` would show an error toast if a real API call were added and it threw. `finally` always turns `loading` back off.

```tsx
  const handleChangePassword = async (e: React.FormEvent) => {
    e.preventDefault();
    
    if (passwordForm.new !== passwordForm.confirm) {
      toast.error("New password and confirm password do not match");
      return;
    }

    if (passwordForm.new.length < 6) {
      toast.error("New password must be at least 6 characters");
      return;
    }
```
Client-side validation: (1) new/confirm must match, (2) new password must be at least 6 characters. Either failure shows a toast and exits early (`return`) without submitting.

```tsx
    setLoading(true);
    try {
      // TODO: Call API to change password
      toast.success("Password changed successfully");
      setPasswordForm({ current: "", new: "", confirm: "" });
      setActiveSection("view");
    } catch (err: any) {
      toast.error(err.message || "Failed to change password");
    } finally {
      setLoading(false);
    }
  };
```
Same simulated-success pattern as `handleSaveProfile`: shows success, clears the password fields, returns to the "view" tab.

```tsx
  const handleLogout = () => {
    logout();
    nav("/login", { replace: true });
  };
```
Calls the auth hook's `logout()` (clears session/user), then redirects to `/login`. `replace: true` means this navigation replaces the current history entry instead of pushing a new one, so the back button won't return to the authenticated page.

```tsx
  return (
    <AppShell>
      <PageHeader kicker="ACCOUNT SETTINGS" title="Admin Profile" />
```
Wraps the page in the shared shell and renders a header with a small "kicker" label above the big title.

```tsx
      <div className="max-w-2xl">
        {/* Tabs */}
        <div className="flex gap-2 mb-8 border-b border-border">
          {[
            { id: "view" as Section, label: "View Profile" },
            { id: "edit" as Section, label: "Edit Profile" },
            { id: "password" as Section, label: "Change Password" },
          ].map(tab => (
            <button
              key={tab.id}
              onClick={() => setActiveSection(tab.id)}
              className={`px-4 py-2 font-medium text-sm transition-colors ${
                activeSection === tab.id
                  ? "text-[#b8860b] border-b-2 border-b-[#b8860b]"
                  : "text-[#5c4033] hover:text-[#2c1a0e]"
              }`}
            >
              {tab.label}
            </button>
          ))}
        </div>
```
Builds a 3-item array describing each tab, maps it to `<button>` elements. Clicking a button sets `activeSection`. The className is conditional: the active tab gets a gold underline/text color, inactive tabs get a muted brown that darkens on hover.

```tsx
        {/* View Profile Section */}
        {activeSection === "view" && (
          <div className="bg-gradient-card rounded-md border border-border p-6 space-y-6">
```
Only renders this block when the "view" tab is active (short-circuit `&&` rendering pattern).

```tsx
            <div>
              <label className="stat-label">Full Name</label>
              <p className="text-[#2c1a0e] font-medium mt-1">{editForm.fullName}</p>
            </div>
```
Read-only display row for Full Name, pulling from `editForm` state (not a separate "saved" state — meaning the "view" always reflects whatever is currently in the edit form, even if not actually persisted to a server).

```tsx
            <div>
              <label className="stat-label">Username</label>
              <p className="text-[#2c1a0e] font-medium mt-1">{editForm.username}</p>
            </div>
            <div>
              <label className="stat-label">Email</label>
              <p className="text-[#2c1a0e] font-medium mt-1">{editForm.email}</p>
            </div>
            <div>
              <label className="stat-label">Contact Number</label>
              <p className="text-[#2c1a0e] font-medium mt-1">{editForm.contactNumber}</p>
            </div>
```
Same read-only display pattern repeated for Username, Email, Contact Number.

```tsx
            <div>
              <label className="stat-label">Role</label>
              <p className="text-[#2c1a0e] font-medium mt-1">Administrator</p>
            </div>
```
Static (hardcoded) "Role" field — always shows "Administrator" since this page is only reachable by admins.

```tsx
            <Button
              onClick={() => setActiveSection("edit")}
              variant="gold"
              size="sm"
            >
              Edit Profile
            </Button>
          </div>
        )}
```
Button that switches to the "edit" tab.

```tsx
        {/* Edit Profile Section */}
        {activeSection === "edit" && (
          <form onSubmit={handleSaveProfile} className="bg-gradient-card rounded-md border border-border p-6 space-y-5">
```
Renders the edit form only when `activeSection === "edit"`; submitting it calls `handleSaveProfile`.

```tsx
            <div>
              <Label htmlFor="fullName">Full Name</Label>
              <Input
                id="fullName"
                type="text"
                value={editForm.fullName}
                onChange={e => handleEditChange("fullName", e.target.value)}
                className="mt-2"
              />
            </div>
```
A controlled text input: its `value` is bound to state and every keystroke calls `handleEditChange` to update state. `htmlFor`/`id` link the label to the input for accessibility.

```tsx
            <div>
              <Label htmlFor="username">Username</Label>
              <Input
                id="username"
                type="text"
                value={editForm.username}
                onChange={e => handleEditChange("username", e.target.value)}
                className="mt-2"
              />
            </div>
```
Same controlled-input pattern for Username.

```tsx
            <div>
              <Label htmlFor="email">Email</Label>
              <Input
                id="email"
                type="email"
                value={editForm.email}
                onChange={e => handleEditChange("email", e.target.value)}
                className="mt-2"
              />
            </div>
```
Same pattern for Email, but `type="email"` triggers browser email-format validation/keyboard.

```tsx
            <div>
              <Label htmlFor="contact">Contact Number</Label>
              <Input
                id="contact"
                type="text"
                value={editForm.contactNumber}
                onChange={e => handleEditChange("contactNumber", e.target.value)}
                className="mt-2"
              />
            </div>
```
Same pattern for Contact Number.

```tsx
            <div className="flex gap-3 pt-4">
              <Button type="submit" variant="gold" disabled={loading}>
                {loading ? "Saving..." : "Save Changes"}
              </Button>
              <Button
                type="button"
                variant="outline"
                onClick={() => setActiveSection("view")}
              >
                Cancel
              </Button>
            </div>
          </form>
        )}
```
Submit button (disabled while `loading`, label toggles) and a Cancel button (`type="button"` so it doesn't trigger form submission) that discards edits by just switching back to "view" — note it does **not** reset `editForm`, so any typed-but-unsaved changes remain in state.

```tsx
        {/* Change Password Section */}
        {activeSection === "password" && (
          <form onSubmit={handleChangePassword} className="bg-gradient-card rounded-md border border-border p-6 space-y-5">
```
Password form, shown only on the "password" tab.

```tsx
            <div>
              <Label htmlFor="current">Current Password</Label>
              <div className="relative mt-2">
                <Input
                  id="current"
                  type={showCurrentPassword ? "text" : "password"}
                  value={passwordForm.current}
                  onChange={e => handlePasswordChange("current", e.target.value)}
                />
                <button
                  type="button"
                  onClick={() => setShowCurrentPassword(!showCurrentPassword)}
                  className="absolute right-3 top-1/2 -translate-y-1/2"
                >
                  {showCurrentPassword ? <EyeOff className="h-4 w-4" /> : <Eye className="h-4 w-4" />}
                </button>
              </div>
            </div>
```
Current-password input whose `type` toggles between `"password"` (masked) and `"text"` (visible) based on `showCurrentPassword`. The eye icon button (absolutely positioned inside the relatively-positioned wrapper) flips that boolean; the icon itself swaps between `Eye`/`EyeOff` to reflect current state.

```tsx
            <div>
              <Label htmlFor="new">New Password</Label>
              <div className="relative mt-2">
                <Input
                  id="new"
                  type={showNewPassword ? "text" : "password"}
                  value={passwordForm.new}
                  onChange={e => handlePasswordChange("new", e.target.value)}
                />
                <button
                  type="button"
                  onClick={() => setShowNewPassword(!showNewPassword)}
                  className="absolute right-3 top-1/2 -translate-y-1/2"
                >
                  {showNewPassword ? <EyeOff className="h-4 w-4" /> : <Eye className="h-4 w-4" />}
                </button>
              </div>
            </div>
```
Identical show/hide pattern for the New Password field.

```tsx
            <div>
              <Label htmlFor="confirm">Confirm New Password</Label>
              <div className="relative mt-2">
                <Input
                  id="confirm"
                  type={showConfirmPassword ? "text" : "password"}
                  value={passwordForm.confirm}
                  onChange={e => handlePasswordChange("confirm", e.target.value)}
                />
                <button
                  type="button"
                  onClick={() => setShowConfirmPassword(!showConfirmPassword)}
                  className="absolute right-3 top-1/2 -translate-y-1/2"
                >
                  {showConfirmPassword ? <EyeOff className="h-4 w-4" /> : <Eye className="h-4 w-4" />}
                </button>
              </div>
              {passwordForm.new !== passwordForm.confirm && passwordForm.confirm && (
                <p className="text-red-600 text-xs mt-1">Passwords do not match</p>
              )}
            </div>
```
Same show/hide pattern, plus a **live** mismatch warning: shown only if `new !== confirm` **and** the user has actually typed something in `confirm` (so the warning doesn't appear while `confirm` is still empty).

```tsx
            <div className="flex gap-3 pt-4">
              <Button type="submit" variant="gold" disabled={loading}>
                {loading ? "Updating..." : "Update Password"}
              </Button>
              <Button
                type="button"
                variant="outline"
                onClick={() => {
                  setPasswordForm({ current: "", new: "", confirm: "" });
                  setActiveSection("view");
                }}
              >
                Cancel
              </Button>
            </div>
          </form>
        )}
```
Submit button (disabled/relabeled while loading) and a Cancel button that, unlike the profile-edit Cancel, **does** reset the password fields before returning to "view" — sensible since leaving typed passwords lying around in memory is undesirable.

```tsx
        {/* Logout Section */}
        <div className="mt-8 pt-6 border-t border-border">
          <Button onClick={handleLogout} variant="destructive" size="lg">
            Sign Out
          </Button>
        </div>
      </div>
    </AppShell>
  );
}
```
Always-visible logout button at the bottom (outside the tab conditionals), styled as a destructive/red action, that calls `handleLogout`.

---

### 2. `CustomerProfile.tsx`

This file mirrors `AdminProfile.tsx` closely but is customer-specific. Only the differences are explained below in depth; shared patterns (controlled inputs, tabs, password show/hide, validation) work exactly as described above.

```tsx
import { Textarea } from "@/components/ui/textarea";
```
Extra import vs. AdminProfile — needed because customers have an "Address" field, which uses a multi-line textarea.

```tsx
import dev from "@/lib/devData";
```
Imports the mock dev-data module (customers, orders, etc.) used only in development mode.

```tsx
  const customer = dev.devCustomers.find(c => c.id === user?.id) || {};
```
Looks up the full mock customer record matching the logged-in user's ID from the dev dataset; falls back to an empty object if not found (so property access below doesn't crash).

```tsx
  const [editForm, setEditForm] = useState({
    fullName: user?.name || "Sarah Ali",
    email: user?.email || "sarah@example.com",
    phone: customer?.phone || "+92-300-0000001",
    address: customer?.address || "123 Main Street, City",
  });
```
Seeds the edit form from the real user where available, otherwise from the matched dev-data record, otherwise hardcoded demo defaults — three-level fallback chain.

```tsx
  const handleSaveProfile = async (e: React.FormEvent) => {
    e.preventDefault();
    setLoading(true);
    try {
      // in DEV we just toast; in prod call customerService.update
      toast.success("Profile updated");
      setActiveSection("view");
    } catch (err: any) {
      toast.error(err.message || "Failed to update");
    } finally { setLoading(false); }
  };
```
Same simulate-success pattern as AdminProfile; the comment documents intended production behavior (`customerService.update`) which isn't wired up yet.

```tsx
  const handleChangePassword = async (e: React.FormEvent) => {
    e.preventDefault();
    if (passwordForm.new !== passwordForm.confirm) { toast.error("Passwords do not match"); return; }
    if (passwordForm.new.length < 6) { toast.error("Password too short"); return; }
    setLoading(true);
    try {
      toast.success("Password updated");
      setPasswordForm({ current: "", new: "", confirm: "" });
      setActiveSection("view");
    } catch (err: any) {
      toast.error(err.message || "Failed to change password");
    } finally { setLoading(false); }
  };
```
Same two validations (match + min length) as admin, written more tersely on one line each; same simulated success flow.

```tsx
  const handleLogout = () => { logout(); nav('/login', { replace: true }); };
```
Same logout redirect logic as AdminProfile.

```tsx
        {activeSection === 'view' && (
          <div className="bg-gradient-card rounded-md border border-border p-6 space-y-4">
            <div>
              <Label>Customer ID</Label>
              <div className="text-[#2c1a0e] font-medium mt-1">{user?.id || '—'}</div>
            </div>
```
Unlike AdminProfile's view tab, this one shows the actual **Customer ID** (`user?.id`), falling back to an em-dash if missing.

```tsx
            <div>
              <Label>Account Status</Label>
              <div className="text-[#2c1a0e] font-medium mt-1 inline-block px-2 py-1 bg-green-100 text-green-800 rounded text-sm">Active</div>
            </div>
```
Hardcoded green "Active" pill — always shows Active regardless of real account state (no backend field wired in yet).

```tsx
            <div>
              <Label>Member Since</Label>
              <div className="text-[#2c1a0e] font-medium mt-1">{customer?.memberSince || 'January 2024'}</div>
            </div>
```
Shows the dev-data `memberSince` if present, else a hardcoded fallback date.

```tsx
            <div className="flex gap-3 pt-4">
              <Button onClick={() => setActiveSection('edit')} variant="gold">Edit Profile</Button>
              <Button onClick={() => setActiveSection('password')} variant="outline">Change Password</Button>
              <Button onClick={handleLogout} variant="outline">Sign Out</Button>
            </div>
          </div>
        )}
```
Difference from AdminProfile: all three actions (Edit, Change Password, **and Sign Out**) live together on the "view" tab, rather than Sign Out being a separate section below the tabs.

```tsx
        {activeSection === 'edit' && (
          <form onSubmit={handleSaveProfile} className="bg-gradient-card rounded-md border border-border p-6 space-y-4">
            <div>
              <Label>Full Name</Label>
              <Input value={editForm.fullName} onChange={e => handleEditChange('fullName', e.target.value)} className="mt-2" title="Your full name" />
            </div>
```
Same controlled-input pattern as AdminProfile, plus a `title` attribute (native browser tooltip on hover) — a small UX addition not present in AdminProfile.

```tsx
            <div>
              <Label>Address</Label>
              <Textarea value={editForm.address} onChange={e => handleEditChange('address', e.target.value)} className="mt-2" rows={3} title="Your shipping address" placeholder="Street address, city, country..." />
            </div>
```
Multi-line `Textarea` (3 rows) for the shipping address, with placeholder guidance text.

The password section (`activeSection === 'password'`) is functionally identical to AdminProfile's, just with `title`/`placeholder` attributes added to each input for extra affordance. The final logout block that AdminProfile has separately is **absent** here since Sign Out already lives in the "view" tab.

---

### 3. `Customers.tsx`

```tsx
import { PageHeader } from "@/components/layout/AppShell";
import { calcOrderTotals } from "@/lib/calc";
```
Imports the page header component and a shared pricing/total-calculation utility used across the app (materials cost + labor, etc.).

```tsx
import * as customerService from "@/services/customerService";
import * as orderService from "@/services/orderService";
import * as employeeService from "@/services/employeeService";
```
Imports the three backend-service modules (each is a namespace with functions like `.list()`, `.create()`) used to fetch real data in production mode.

```tsx
import dev from "@/lib/devData";
import { Mail, User as UserIcon } from "lucide-react";
import { useEffect, useState } from "react";
```
Dev mock data, icons (Mail icon, and `User` icon aliased to `UserIcon` to avoid clashing with a `User` type name elsewhere), and React hooks.

```tsx
export default function Customers() {
  const [customers, setCustomers] = useState<any[]>([]);
  const [orders, setOrders] = useState<any[]>([]);
  const [employees, setEmployees] = useState<any[]>([]);
```
Three pieces of state to hold fetched customers, orders, and employees (all typed loosely as `any[]` since the shape varies between dev-mock and real API data).

```tsx
  useEffect(() => {
    (async () => {
      if (import.meta.env.DEV) {
```
On mount, runs an async IIFE (immediately-invoked function expression) inside `useEffect` (since `useEffect` callbacks can't be `async` directly). `import.meta.env.DEV` is a Vite build-time flag: true during local development.

```tsx
        setCustomers(dev.devCustomers);
        setOrders(dev.devOrders.map((o: any) => ({ ...o, assignedEmployeeId: o.employeeId || o.assignedEmployeeId })));
        setEmployees(dev.devEmployees.map((u: any) => ({ ...u, hourlyRate: u.rate || u.hourlyRate })));
        return;
      }
```
In dev mode: loads mock customers directly; normalizes orders so every order has a consistent `assignedEmployeeId` field (some mock records use `employeeId` instead); normalizes employees so every one has `hourlyRate` (some mock records use `rate`). Then `return`s early so the real API call below is skipped.

```tsx
      const [c, o, e] = await Promise.all([customerService.list(), orderService.list(), employeeService.list()]);
      setCustomers(c); setOrders(o); setEmployees(e);
    })();
  }, []);
```
In production: fetches all three lists **in parallel** via `Promise.all`, then stores the results. Empty dependency array `[]` means this effect runs once on mount only.

```tsx
  return (
    <>
      <PageHeader kicker="Directory" title="Customers." />
```
Fragment wrapper (no extra DOM node) containing the page header.

```tsx
      {customers.length === 0 ? (
        <div className="border border-dashed border-border rounded-2xl p-16 text-center text-muted-foreground">No customers registered.</div>
      ) : (
```
Empty-state message if there are no customers; otherwise renders the grid below.

```tsx
        <div className="grid md:grid-cols-2 lg:grid-cols-3 gap-4">
          {customers.map(c => {
            const myOrders = orders.filter(o => o.customerId === c.id);
```
Responsive grid (1 col on mobile, 2 on medium screens, 3 on large). For each customer `c`, filters the full orders list down to just this customer's orders.

```tsx
            const spend = myOrders.reduce((s, o) => {
              const emp = employees.find(u => u.id === o.assignedEmployeeId);
              return s + calcOrderTotals(o, emp, undefined).total;
            }, 0);
```
Computes total spend: for each of the customer's orders, finds the employee assigned to it (to factor in labor cost), calls the shared `calcOrderTotals(order, employee, materialsList)` helper (materials list passed as `undefined` here — meaning this total calculation doesn't factor in live material pricing, only what `calcOrderTotals` can derive from the order + employee alone), and sums the `.total` field across all orders. **Note:** `spend` is computed but never actually rendered anywhere in the returned JSX below — it's a leftover/unused calculation.

```tsx
            return (
              <div key={c.id} className="rounded-2xl border border-gold/15 bg-gradient-card p-6 ring-inset-light">
                <div className="flex items-center gap-3 mb-4">
                  <div className="h-11 w-11 rounded-xl bg-gradient-gold flex items-center justify-center shadow-gold">
                    <UserIcon className="h-5 w-5 text-primary-foreground" />
                  </div>
```
Each customer renders as a card. A small gold gradient square with a person icon acts as an avatar placeholder (no real profile photos).

```tsx
                  <div>
                    <div className="font-display text-lg leading-tight">{c.name}</div>
                    <div className="text-xs text-muted-foreground flex items-center gap-1"><Mail className="h-3 w-3" />{c.email}</div>
                  </div>
                </div>
```
Customer's name (large display font) and email (small, with a mail icon) next to the avatar.

```tsx
                <div className="gold-divider w-10 mb-4" />
```
A short decorative horizontal divider line.

```tsx
                <div className="grid grid-cols-2 gap-3">
                  <div>
                    <div className="stat-label">Total orders</div>
                    <div className="font-display text-2xl text-[#2c1a0e]">{myOrders.length}</div>
                  </div>
                  <div>
                    <div className="stat-label">Pending orders</div>
                    <div className="font-display text-2xl value-amber">{myOrders.filter(o => o.status === "Pending").length}</div>
                  </div>
                </div>
              </div>
            );
          })}
        </div>
      )}
    </>
  );
}
```
A 2-column stat block: total order count for that customer, and count of orders currently in "Pending" status (highlighted in amber/gold color via the `value-amber` class).

---

### 4. `Dashboard.tsx`

```tsx
import { useAuth } from "@/lib/auth";
import AdminOverview from "./dashboards/AdminOverview";
import CustomerOverview from "./dashboards/CustomerOverview";
import EmployeeOverview from "./dashboards/EmployeeOverview";
```
Imports the auth hook and the three role-specific dashboard components (located in a `dashboards` subfolder relative to this file).

```tsx
export default function Dashboard() {
  const { user } = useAuth();
```
Grabs the currently logged-in `user`.

```tsx
  if (!user) return null;
```
Guard clause: if there's no logged-in user (e.g., auth state hasn't loaded yet, or somehow reached this route unauthenticated), render nothing rather than crashing on `user.role` below.

```tsx
  if (user.role === "admin") return <AdminOverview />;
  if (user.role === "customer") return <CustomerOverview />;
  return <EmployeeOverview />;
}
```
Simple role-based router: admin → `AdminOverview`, customer → `CustomerOverview`, and **any other role** (in practice, `"employee"`) falls through to `EmployeeOverview` as the default. This file itself renders no shared layout/markup — it's purely a switch.

---

### 5. `EmployeeProfile.tsx`

Structurally identical to `CustomerProfile.tsx` (view/edit/password tabs, controlled inputs, show/hide password, validation, logout). Only employee-specific differences are detailed here.

```tsx
  const [editForm, setEditForm] = useState({
    fullName: user?.name || "Bilal",
    email: user?.email || "bilal@example.com",
    phone: user?.phone || "+92-300-0000000",
    hourlyRate: (user as any)?.hourlyRate ?? 25,
    bankAccount: (user as any)?.bankAccount || "",
  });
```
Seeds the form from the real `user` object where possible. `hourlyRate` and `bankAccount` are cast to `any` because the base `user` type likely doesn't formally declare those employee-only fields. `?? 25` (nullish coalescing) defaults `hourlyRate` to 25 only if it's `null`/`undefined` (unlike `||`, this would preserve a real value of `0`). `bankAccount` defaults to an empty string if absent.

```tsx
  const handleEditChange = (field: string, value: any) => setEditForm(prev => ({ ...prev, [field]: value }));
```
Same generic field-updater as other profile pages, but typed `value: any` instead of `string` because `hourlyRate` needs to accept a `number`.

```tsx
            <div>
              <Label>Hourly Rate</Label>
              <div className="text-[#2c1a0e] font-medium mt-1">${editForm.hourlyRate}/hr</div>
            </div>
            <div>
              <Label>Bank Account (for wages)</Label>
              <div className="text-[#2c1a0e] font-medium mt-1">{editForm.bankAccount || 'Not set'}</div>
            </div>
```
View-tab rows unique to employees: hourly rate (formatted as `$X/hr`) and bank account (shows "Not set" placeholder text if empty).

```tsx
            <div>
              <Label>Hourly Rate</Label>
              <Input type="number" value={String(editForm.hourlyRate)} onChange={e => handleEditChange('hourlyRate', parseFloat(e.target.value) || 0)} className="mt-2" />
            </div>
```
Edit-tab numeric input for hourly rate: the state value (a number) is converted to a string for the controlled `<Input>`'s `value` prop; on change, the typed string is parsed back to a float, defaulting to `0` if parsing fails (e.g., empty or non-numeric string → `NaN` → `0`).

```tsx
            <div>
              <Label>Bank Account</Label>
              <Input value={editForm.bankAccount} onChange={e => handleEditChange('bankAccount', e.target.value)} className="mt-2" />
            </div>
```
Plain text input for bank account details.

```tsx
        {activeSection === 'view' && (
          ...
            <div className="flex gap-3 pt-4">
              <Button onClick={() => setActiveSection('edit')} variant="gold">Edit Profile</Button>
              <Button onClick={() => setActiveSection('password')} variant="outline">Change Password</Button>
            </div>
          </div>
        )}
```
Unlike CustomerProfile, the Sign Out button is **not** included in this action row — it's placed separately below (same as AdminProfile's pattern):

```tsx
        <div className="mt-8 pt-6 border-t border-border">
          <Button onClick={handleLogout} variant="destructive" size="lg">Sign Out</Button>
        </div>
```
Always-visible Sign Out button at the very bottom of the page, outside the tab conditionals.

---

### 6. `Employees.tsx`

```tsx
import { PageHeader } from "@/components/layout/AppShell";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import * as employeeService from "@/services/employeeService";
import * as orderService from "@/services/orderService";
import dev from "@/lib/devData";
import { Dialog, DialogContent, DialogHeader, DialogTitle, DialogTrigger, DialogFooter } from "@/components/ui/dialog";
import { Plus, Trash2 } from "lucide-react";
import { useState, useEffect } from "react";
import { toast } from "sonner";
```
Imports: layout, form UI, backend services, mock data, a full modal Dialog kit, icons (`Plus` for "add" button, `Trash2` unused in this file's JSX but imported for potential removal UI), React hooks, and toast.

```tsx
export default function Employees() {
  const [tick, setTick] = useState(0);
  const refresh = () => setTick(t => t + 1);
```
`tick` is a dummy counter used purely to **trigger a re-fetch**: incrementing it changes a value in the `useEffect` dependency array below, forcing data to reload.

```tsx
  const [open, setOpen] = useState(false);
  const [form, setForm] = useState({ name: "", email: "", password: "employee123", hourlyRate: 25, skills: "" });
```
`open` controls the "Add employee" dialog's visibility. `form` holds the new-employee form fields, pre-filled with a default password (`employee123`) and hourly rate (`25`).

```tsx
  const [employees, setEmployees] = useState<any[]>([]);
  const [orders, setOrders] = useState<any[]>([]);
```
State for the fetched employee and order lists.

```tsx
  useEffect(() => {
    (async () => {
      if (import.meta.env.DEV) {
        setEmployees(dev.devEmployees.map((u: any) => ({ ...u, hourlyRate: u.rate || u.hourlyRate })));
        setOrders(dev.devOrders.map((o: any) => ({ ...o, assignedEmployeeId: o.employeeId || o.assignedEmployeeId })));
        return;
      }
      const [e, o] = await Promise.all([employeeService.list(), orderService.list()]);
      setEmployees(e); setOrders(o);
    })();
  }, [tick]);
```
Same dev/prod data-loading pattern as `Customers.tsx`. Runs again whenever `tick` changes (i.e., after `refresh()` is called), so newly-added employees show up.

```tsx
  const submit = async () => {
    if (!form.name || !form.email) return toast.error("Name and email required");
    try {
      await employeeService.create({ name: form.name, email: form.email, password: form.password, hourlyRate: form.hourlyRate, skills: form.skills });
      setOpen(false); refresh();
      setForm({ name: "", email: "", password: "employee123", hourlyRate: 25, skills: "" });
      toast.success("Artisan added");
    } catch (e) { toast.error("Failed to add artisan"); }
  };
```
Validates required fields; calls `employeeService.create()` (a real API call, unlike some other pages' TODO stubs); on success closes the dialog, refreshes the list, resets the form to defaults, and toasts success; on failure toasts an error. Note: this function always calls the real `employeeService` even in DEV mode — it isn't wrapped in an `import.meta.env.DEV` branch like data-loading is, meaning in dev mode this call would either hit a stub/mock service or fail depending on how `employeeService` itself is implemented.

```tsx
  const removeEmp = async (id: string) => {
    try {
      await employeeService.remove(id);
      refresh();
      toast.success("Artisan removed");
    } catch { toast.error("Failed to remove artisan"); }
  };
```
Deletes an employee by ID via the service, refreshes the list, and shows a success/failure toast. **Note:** this function is defined but never actually called/wired to any button in the JSX below — it's currently dead code (the `Trash2` icon import is similarly unused in the render).

```tsx
  return (
    <>
      <PageHeader kicker="Crew Console" title="Artisans." />
```
Header for this "Artisans" (employees) page.

```tsx
      <div className="grid md:grid-cols-2 lg:grid-cols-3 gap-4">
        {employees.map(emp => {
          const empOrders = orders.filter(o => o.assignedEmployeeId === emp.id);
          const hours = empOrders.reduce((s, o) => s + o.laborHours, 0);
          const wages = hours * (emp.hourlyRate || 0);
```
For each employee: filters orders assigned to them, sums their `laborHours` across those orders, and multiplies total hours by their hourly rate (defaulting rate to `0` if missing) to get total wages.

```tsx
          return (
            <div key={emp.id} className="rounded-sm border border-border bg-gradient-card p-6 ring-inset-light">
              <div className="flex items-start justify-between mb-4">
                <div className="flex items-center gap-3">
                  <div className="h-12 w-12 rounded-full bg-gradient-gold flex items-center justify-center font-display text-primary-foreground">
                    {emp.name.split(" ").map(n => n[0]).join("").slice(0, 2)}
                  </div>
```
A circular gold avatar showing the employee's **initials**: splits the name on spaces, takes the first letter of each word, joins them, and truncates to 2 characters (so "Bilal Ahmed Khan" → "BAK" → sliced to "BA").

```tsx
                  <div>
                    <div className="font-display text-lg leading-none">{emp.name}</div>
                    <div className="text-xs text-muted-foreground mt-1">{emp.email}</div>
                  </div>
                </div>
              </div>
```
Employee's full name and email next to the avatar.

```tsx
              <div className="flex flex-wrap gap-1 mb-4">
                {(emp.skills || []).map(s => <span key={s} className="text-[10px] uppercase tracking-widest px-2 py-1 border border-gold/30 text-gold rounded-sm">{s}</span>)}
              </div>
```
Renders each of the employee's skills as a small gold-bordered pill/tag. `emp.skills || []` guards against `skills` being `undefined`.

```tsx
              <div className="gold-divider mb-4" />
              <div className="grid grid-cols-3 gap-2 text-center">
                <div><div className="stat-label">Tasks</div><div className="font-display text-xl text-[#2c1a0e]">{empOrders.length}</div></div>
                <div><div className="stat-label">Hours</div><div className="font-display text-xl text-[#2c1a0e]">{hours}</div></div>
                <div><div className="stat-label">Wages</div><div className="font-display text-xl value-amber">${wages.toFixed(0)}</div></div>
              </div>
            </div>
          );
        })}
      </div>
    </>
  );
}
```
Three-column stat footer per card: number of assigned tasks, total hours, and computed wages (rounded to whole dollars via `.toFixed(0)`, styled in amber). **Note:** the "Add material"-style `Dialog`/`DialogTrigger` imported for adding a new employee is imported but — looking at the returned JSX — is not actually rendered anywhere in this excerpt, meaning the "Add Artisan" UI (`open`/`form`/`submit`) is currently unused/not wired into the visible page (likely cut off or a work-in-progress feature).

---

### 7. `Inventory.tsx`

```tsx
import { PageHeader } from "@/components/layout/AppShell";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import { Material } from "@/lib/store";
import * as inventoryService from "@/services/inventoryService";
import dev from "@/lib/devData";
import { useEffect, useState } from "react";
import { Dialog, DialogContent, DialogHeader, DialogTitle, DialogTrigger, DialogFooter } from "@/components/ui/dialog";
import { Plus, Trash2, AlertTriangle } from "lucide-react";
import { toast } from "sonner";
import { useAuth } from "@/lib/auth";
```
Standard imports; `Material` is a shared TypeScript type/interface describing a material record; `AlertTriangle` icon flags low-stock items.

```tsx
const empty: Omit<Material, "id"> = { name: "", type: "Hardwood", price: 0, quantity: 0, threshold: 0, supplier: "" };
```
A module-level constant representing a blank "new material" form. `Omit<Material, "id">` is a TypeScript utility type meaning "the `Material` shape, minus the `id` field" (since IDs are usually server-generated).

```tsx
export default function Inventory() {
  const { user } = useAuth();
  const isAdmin = user!.role === "admin";
```
Gets the current user (`user!` — the `!` asserts to TypeScript that `user` is definitely non-null here, since this page is only reachable when authenticated) and derives a boolean for whether admin-only controls should render.

```tsx
  const [tick, setTick] = useState(0);
  const refresh = () => setTick(t => t + 1);
  const [open, setOpen] = useState(false);
  const [form, setForm] = useState(empty);
```
Refresh-trigger counter, dialog open state, and the new-material form state (initialized from the `empty` template).

```tsx
  const [materials, setMaterials] = useState<any[]>([]);
```
State holding the fetched/normalized materials list.

```tsx
  useEffect(() => {
    (async () => {
      if (import.meta.env.DEV) {
        const mapMaterials = (arr: any[]) => arr.map(m => ({ id: m.id, name: m.name, type: m.type, price: m.unitPrice || m.price || 0, quantity: parseFloat(String(m.qty || m.quantity || "0").replace(/[^0-9.]/g, "")) || 0, threshold: parseFloat(String(m.minThreshold || m.threshold || "0").replace(/[^0-9.]/g, "")) || 0, supplier: m.supplier }));
        setMaterials(mapMaterials(dev.devMaterials));
        return;
      }
      const m = await inventoryService.list(); setMaterials(m);
    })();
  }, [tick]);
```
Defines a normalizer `mapMaterials` that reshapes raw dev-mock records into a consistent shape: `price` falls back from `unitPrice`→`price`→`0`; `quantity` and `threshold` are extracted by first coercing to a string, then **stripping out any non-digit/non-decimal-point characters** via regex (`.replace(/[^0-9.]/g, "")`) — this handles mock data where quantity might be stored as a string like `"120 units"` — then parsed to a float, defaulting to `0` if parsing fails. In production, just fetches the already-clean list from `inventoryService`.

```tsx
  const submit = async () => {
    if (!form.name) return toast.error("Name required");
    try {
      if (import.meta.env.DEV) {
        const newMat = {
          id: `M-${Date.now().toString().slice(-4)}`,
          name: form.name,
          type: form.type || "Hardwood",
          price: form.price || 0,
          quantity: form.quantity || 0,
          threshold: form.threshold || 0,
          supplier: form.supplier || "N/A"
        };
```
Validates the name is present. In dev mode, builds a fake new material record with a generated ID: `Date.now()` (milliseconds since epoch) converted to string, keeping only the **last 4 digits** (`.slice(-4)`), prefixed with `M-` — a quick pseudo-unique ID scheme for mock data. Applies sensible fallback defaults for optional fields.

```tsx
        dev.devMaterials.push(newMat);
        setMaterials(prev => [...prev, newMat]);
      } else {
        await inventoryService.create(form);
        refresh();
      }
      setForm(empty);
      setOpen(false);
      toast.success("Material added");
    } catch {
      toast.error("Failed to add material");
    }
  };
```
In dev mode: mutates the shared mock array directly (`dev.devMaterials.push`) so the change persists across component remounts within the session, and appends it to local state for immediate UI update. In production: calls the real API then triggers a refetch via `refresh()`. Either way: resets the form, closes the dialog, and toasts success (or error on failure).

```tsx
  return (
    <>
      <PageHeader kicker="Material Vault" title="Inventory."
        action={isAdmin && (
          <Dialog open={open} onOpenChange={setOpen}>
            <DialogTrigger asChild><Button variant="gold"><Plus className="h-4 w-4" /> Add material</Button></DialogTrigger>
```
The header's `action` slot (likely rendered top-right) only shows the "Add material" dialog trigger button if `isAdmin` is true. `<DialogTrigger asChild>` means the trigger renders as the `<Button>` itself (no extra wrapper element) and clicking it opens the dialog.

```tsx
            <DialogContent>
              <DialogHeader><DialogTitle className="font-display text-2xl">New material</DialogTitle></DialogHeader>
              <div className="grid grid-cols-2 gap-3 py-2">
                <div className="col-span-2"><Label>Name</Label><Input value={form.name} onChange={e => setForm({ ...form, name: e.target.value })} /></div>
                <div><Label>Type</Label><Input value={form.type} onChange={e => setForm({ ...form, type: e.target.value })} /></div>
                <div><Label>Supplier</Label><Input value={form.supplier} onChange={e => setForm({ ...form, supplier: e.target.value })} /></div>
                <div><Label>Price ($)</Label><Input type="number" value={form.price} onChange={e => setForm({ ...form, price: parseFloat(e.target.value) || 0 })} /></div>
                <div><Label>Quantity</Label><Input type="number" value={form.quantity} onChange={e => setForm({ ...form, quantity: parseInt(e.target.value) || 0 })} /></div>
                <div><Label>Low threshold</Label><Input type="number" value={form.threshold} onChange={e => setForm({ ...form, threshold: parseInt(e.target.value) || 0 })} /></div>
              </div>
              <DialogFooter><Button variant="gold" onClick={submit}>Save</Button></DialogFooter>
            </DialogContent>
          </Dialog>
        )} />
```
A 2-column form grid inside the dialog: Name spans both columns (`col-span-2`); Type, Supplier, Price, Quantity, Low threshold each take one column. Numeric fields parse input as float/int with a `0` fallback. Save button calls `submit`.

```tsx
      <div className="grid sm:grid-cols-2 lg:grid-cols-3 gap-4">
        {materials.map(m => {
          const low = m.quantity <= m.threshold;
```
Responsive materials grid; `low` flags whether stock is at or below the reorder threshold.

```tsx
          return (
            <div key={m.id} className={"rounded-sm border bg-gradient-card p-5 ring-inset-light " + (low ? "low-stock-card" : "border-border")}>
              <div className="flex items-start justify-between mb-2">
                <div className="section-header !text-[10px] !py-1.5 !px-3 !rounded-md">{m.type}</div>
                {low && <AlertTriangle className="h-4 w-4 text-destructive" />}
              </div>
```
Card border styling switches to a special `low-stock-card` class when stock is low. Shows material type as a small pill; shows a warning triangle icon only if `low` is true.

```tsx
              <div className="font-display text-lg">{m.name}</div>
              <div className="text-xs text-muted-foreground mb-4">{m.supplier}</div>
```
Material name and supplier text.

```tsx
              <div className="flex items-end justify-between">
                <div>
                  <div className="stat-label">Stock</div>
                  <div className={"font-display text-3xl " + (low ? "low-stock-value" : "text-[#2c1a0e]")}>{m.quantity}</div>
                  <div className="text-[10px] text-[#5c4033]">min {m.threshold}</div>
                </div>
                <div className="text-right">
                  <div className="stat-label">Price</div>
                  <div className="font-display text-2xl value-amber">${m.price}</div>
                </div>
              </div>
```
Left side: current stock quantity (large number, red-tinted if low) with the minimum threshold noted below it. Right side: unit price in amber.

```tsx
              {isAdmin && (
                <div className="mt-4 flex gap-2 items-center">
                  <Input
                    type="number"
                    id={`qty-input-${m.id}`}
                    defaultValue={m.quantity}
                    className="h-8 flex-1"
                  />
```
Admin-only inline stock-update control: an **uncontrolled** number input (`defaultValue` not `value` — React won't re-render it on every keystroke; its live value is read directly from the DOM when needed) with a unique `id` per material so it can be looked up later.

```tsx
                  <Button
                    variant="gold"
                    size="sm"
                    className="h-8 px-3 text-xs"
                    onClick={async () => {
                      const inputEl = document.getElementById(`qty-input-${m.id}`) as HTMLInputElement;
                      const newQty = parseInt(inputEl?.value || "0") || 0;
```
On click: grabs the raw DOM `<input>` by its generated ID (a direct-DOM-access pattern, bypassing React state — a deliberate simplification since this field doesn't need to be controlled), reads its current value as an integer, defaulting to `0` for empty/invalid input.

```tsx
                      try {
                        if (import.meta.env.DEV) {
                          setMaterials(prev => prev.map(item => item.id === m.id ? { ...item, quantity: newQty } : item));
                          const devItem = dev.devMaterials.find(item => item.id === m.id);
                          if (devItem) devItem.qty = newQty;
                        } else {
                          await inventoryService.update(m.id, { quantity: newQty });
                          refresh();
                        }
                        toast.success("Stock updated");
                      } catch {
                        toast.error("Update failed");
                      }
                    }}
                  >
                    Update
                  </Button>
                </div>
              )}
            </div>
          );
        })}
      </div>
    </>
  );
}
```
In dev mode: updates the local `materials` state array immutably (maps, replacing the matching item's `quantity`), **and** also mutates the underlying raw mock record (`devItem.qty = newQty`) directly so the change is reflected if data gets re-normalized later. In production: calls the real update API then refetches. Either path ends with a success/failure toast.

---

### 8. `Login.tsx`

```tsx
import { Link, useLocation, useNavigate } from "react-router-dom";
import { useState } from "react";
import { useAuth } from "@/lib/auth";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import { toast } from "sonner";
import { Logo } from "@/components/Logo";
import hero from "@/assets/hero-workshop.jpg";
```
Router primitives (`Link` for navigation without full reload, `useLocation`/`useNavigate` for redirect logic), form UI, auth hook, toast, a `Logo` component, and a bundled hero image asset.

```tsx
export default function Login() {
  const { login } = useAuth();
  const nav = useNavigate();
  const loc = useLocation() as any;
```
Pulls `login` from the auth hook; gets navigate and location objects. `as any` bypasses strict typing on `loc` so `loc.state?.from` (custom redirect-origin data) can be read without a type error.

```tsx
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [loading, setLoading] = useState(false);
```
Local state for the two form fields and a submit-in-progress flag.

```tsx
  const submit = async (e: React.FormEvent) => {
    e.preventDefault();
    if (!email || !password) return toast.error("Enter email and password");
    setLoading(true);
    const r = await login(email, password);
    setLoading(false);
    if (!r.ok) return toast.error(r.error || "Login failed");
    toast.success("Welcome back");
    nav(loc.state?.from || "/app", { replace: true });
  };
```
Prevents default form submission; requires both fields non-empty; sets loading, awaits the `login()` call (which presumably returns `{ ok: boolean, error?: string }`); turns loading off; on failure shows the returned error (or a generic fallback); on success shows a welcome toast and redirects — to wherever the user originally tried to go (`loc.state.from`, set by a protected-route redirect elsewhere in the app) or to `/app` by default.

```tsx
  const quick = (e: string, p: string) => { setEmail(e); setPassword(p); };
```
Helper that fills the email/password fields for the "Demo accounts" quick-login buttons — does **not** submit automatically, just pre-fills.

```tsx
  return (
    <div className="min-h-screen grid lg:grid-cols-2">
```
Full-height two-column layout on large screens (single column implicitly on smaller screens since only `lg:grid-cols-2` is specified).

```tsx
      <div className="relative hidden lg:block grain bg-[linear-gradient(90deg,_rgba(0,0,0,0.7),_transparent)]">
        <img src={hero} alt="Artisan workshop" className="absolute inset-0 h-full w-full object-cover opacity-70" width={1920} height={1280} />
        <div className="absolute inset-0 bg-gradient-to-r from-background via-background/40 to-transparent" />
```
Left marketing panel, hidden on small screens (`hidden lg:block`). Background hero photo fills the panel (`object-cover`), at 70% opacity, layered under a left-to-right gradient overlay for text legibility.

```tsx
        <div className="relative h-full flex flex-col justify-between p-12">
          <Logo to="/" size="md" showTagline />
          <div>
            <p className="serif italic text-2xl text-foreground/90 max-w-md leading-snug">
              "Every grain tells a story. Every joint, a quiet promise."
            </p>
            <div className="gold-divider w-32 mt-6" />
            <p className="text-xs uppercase tracking-[0.3em] text-gold mt-3">— The Atelier, est. 1972</p>
          </div>
        </div>
      </div>
```
Content overlaid on the image: logo (linking home) at top, a stylized italic quote and attribution at the bottom, pushed apart via `justify-between`.

```tsx
      <div className="flex items-center justify-center p-6 md:p-12">
        <div className="w-full max-w-md">
          <div className="text-[11px] uppercase tracking-[0.3em] text-gold mb-3">The bench is warm</div>
          <h1 className="font-display text-4xl md:text-5xl mb-2 leading-tight">Shape Ideas Into <span className="serif italic text-gold">Masterpieces</span>.</h1>
          <p className="text-muted-foreground mb-8">Sign in to power your woodwork operations.</p>
```
Right column: the actual login form area, centered, max-width constrained. Marketing headline with a styled `<span>` for emphasis.

```tsx
          <form onSubmit={submit} className="space-y-4">
            <div>
              <Label htmlFor="email">Email</Label>
              <Input id="email" type="email" value={email} onChange={e => setEmail(e.target.value)} placeholder="you@woodcraft.com" />
            </div>
            <div>
              <Label htmlFor="password">Password</Label>
              <Input id="password" type="password" value={password} onChange={e => setPassword(e.target.value)} />
            </div>
            <Button type="submit" variant="gold" size="lg" className="w-full" disabled={loading}>
              {loading ? "Entering..." : "Sign in"}
            </Button>
          </form>
```
Standard controlled email/password form, submit button full-width, disabled and relabeled while `loading`.

```tsx
          <div className="mt-8">
            <div className="text-xs uppercase tracking-widest text-muted-foreground mb-3">Demo accounts</div>
            <div className="grid grid-cols-3 gap-2">
              {[
                { l: "Admin", e: "admin@woodcraft.com", p: "admin123" },
                { l: "Customer", e: "customer@woodcraft.com", p: "customer123" },
                { l: "Employee", e: "employee@woodcraft.com", p: "employee123" },
              ].map(d => (
                <button key={d.l} type="button" onClick={() => quick(d.e, d.p)}
                  className="text-xs px-3 py-2 border border-border rounded-sm hover:border-gold hover:text-gold transition">
                  {d.l}
                </button>
              ))}
            </div>
          </div>
```
Three quick-fill demo-account buttons in a 3-column row, each pre-filling the credential fields for that role (does not auto-submit — the user still has to click Sign In).

```tsx
          <p className="mt-8 text-sm text-muted-foreground flex flex-wrap items-center gap-x-3 gap-y-1">
            <span>New here?</span>
            <Link to="/register" className="text-gold hover:underline">Create an account</Link>
            <Link to="/" className="text-gold hover:underline">Back to home</Link>
          </p>
        </div>
      </div>
    </div>
  );
}
```
Footer links to the Register page and the marketing home page, using client-side `<Link>` navigation (no full page reload).

---

### 9. `NotFound.tsx`

```tsx
import { useLocation } from "react-router-dom";
import { useEffect } from "react";
```
Imports the router location hook and `useEffect`.

```tsx
const NotFound = () => {
  const location = useLocation();
```
Declares the component as a const arrow function (rather than `function NotFound()` like the other files); reads the current URL location object.

```tsx
  useEffect(() => {
    console.error("404 Error: User attempted to access non-existent route:", location.pathname);
  }, [location.pathname]);
```
Logs an error to the browser console whenever the pathname changes (useful for catching broken links / debugging in dev, and would show up in error-monitoring tools in production). Dependency array ensures it re-logs if the user somehow navigates to a different bad route while already on this page.

```tsx
  return (
    <div className="flex min-h-screen items-center justify-center bg-muted">
      <div className="text-center">
        <h1 className="mb-4 text-4xl font-bold">404</h1>
        <p className="mb-4 text-xl text-muted-foreground">Oops! Page not found</p>
        <a href="/" className="text-primary underline hover:text-primary/90">
          Return to Home
        </a>
      </div>
    </div>
  );
};
```
Centered 404 message with a plain `<a href="/">` link back home. **Note:** unlike the rest of the app which uses router `<Link>`, this uses a native `<a>` tag, which will trigger a full page reload rather than client-side navigation — likely because this is boilerplate from the default Vite/shadcn template and wasn't updated to match the rest of the app's routing style.

```tsx
export default NotFound;
```
Default export (note: exported at the bottom rather than inline with `export default function`, a stylistic difference from other files).

---

### 10. `Orders.tsx`

```tsx
import { PageHeader } from "@/components/layout/AppShell";
import { StatusBadge } from "@/components/ui/stat-card";
import { Button } from "@/components/ui/button";
import { useAuth } from "@/lib/auth";
import * as orderService from "@/services/orderService";
import * as userService from "@/services/customerService";
import * as employeeService from "@/services/employeeService";
import * as inventoryService from "@/services/inventoryService";
import dev from "@/lib/devData";
import { calcOrderTotals } from "@/lib/calc";
import { OrderStatus, Order } from "@/lib/store";
import { useEffect, useState } from "react";
import { Dialog, DialogContent, DialogHeader, DialogTitle, DialogTrigger, DialogFooter } from "@/components/ui/dialog";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import { Textarea } from "@/components/ui/textarea";
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "@/components/ui/select";
import { Plus, Trash2 } from "lucide-react";
import { toast } from "sonner";
```
A large import list reflecting this being the most complex page: layout/UI kit, all four backend services (orders, customers — imported oddly as `userService`, employees, inventory), mock data, pricing calc utility, shared types, dialog/select/textarea UI, icons, toast.

```tsx
export default function Orders() {
  const { user } = useAuth();
  const [tick, setTick] = useState(0);
  const [open, setOpen] = useState(false);
  const [cancelConfirm, setCancelConfirm] = useState<string | null>(null);
  const refresh = () => setTick(t => t + 1);
```
`user` for role-based rendering; `tick`/`refresh` for re-fetch triggering; `open` for the (currently unused-in-JSX, but present) "new order" dialog; `cancelConfirm` holds the order ID pending cancellation confirmation, or `null` when no confirmation dialog is open.

```tsx
  const [customers, setCustomers] = useState<any[]>([]);
  const [employees, setEmployees] = useState<any[]>([]);
  const [materials, setMaterials] = useState<any[]>([]);
  const [orders, setOrders] = useState<any[]>([]);
```
Four separate data lists needed to build/display orders: who can be a customer, who can be assigned, what materials are available, and the orders themselves.

```tsx
  const [form, setForm] = useState({ customerId: "", title: "", description: "", type: "Furniture" as Order["type"], matId: "", matQty: 1 });
```
Form state for creating a new order: customer, title, description, type (typed against the `Order["type"]` union from the shared `Order` interface), and a single material selection (`matId`) with quantity (`matQty`).

```tsx
  const visible = user && orders ? (user.role === "admin" ? orders : user.role === "employee" ? orders.filter(o => o.assignedEmployeeId === user.id) : orders.filter(o => o.customerId === user.id)) : [];
```
Role-based visibility filter: admins see **all** orders; employees see only orders **assigned to them**; everyone else (customers) see only orders **they placed**. Falls back to an empty array if `user` or `orders` isn't ready yet.

```tsx
  const [searchQ, setSearchQ] = useState("");
  const [statusFilter, setStatusFilter] = useState<string>("All");
  const [selectedOrder, setSelectedOrder] = useState<any | null>(null);
  const [statusModalOpen, setStatusModalOpen] = useState(false);
  const [newStatus, setNewStatus] = useState<string>("Not Started");
```
Table search text, status dropdown filter, the currently-selected order (for the detail modal), whether the status-update modal is open, and the pending new status value in that modal.

```tsx
  useEffect(() => {
    (async () => {
      try {
        if (import.meta.env.DEV) {
          const mapMaterials = (arr: any[]) => arr.map(m => ({ id: m.id, name: m.name, quantity: parseFloat(String(m.qty || m.quantity || "0").replace(/[^0-9.]/g, "")) || 0, threshold: parseFloat(String(m.minThreshold || m.threshold || "0").replace(/[^0-9.]/g, "")) || 0 }));
          const customers = dev.devCustomers;
          const employees = dev.devEmployees.map((u: any) => ({ ...u, hourlyRate: u.rate || u.hourlyRate }));
          const materials = mapMaterials(dev.devMaterials);
          const orders = dev.devOrders.map((o: any) => ({ ...o, assignedEmployeeId: o.employeeId || o.assignedEmployeeId }));
          setCustomers(customers);
          setEmployees(employees);
          setMaterials(materials);
          setOrders(orders);
          setForm(f => ({ ...f, customerId: customers[0]?.id || "", matId: materials[0]?.id || "" }));
          return;
        }
```
Dev mode: normalizes materials (same regex-stripping technique as `Inventory.tsx`), normalizes employees' rate field, normalizes orders' assignee field, sets all four state lists, and **pre-fills the new-order form** with the first available customer and material as sensible defaults so the "create order" dialog isn't empty.

```tsx
        const [c, e, m, o] = await Promise.all([userService.list(), employeeService.list(), inventoryService.list(), orderService.list()]);
        setCustomers(c);
        setEmployees(e);
        setMaterials(m);
        setOrders(o);
        setForm(f => ({ ...f, customerId: c[0]?.id || "", matId: m[0]?.id || "" }));
      } catch (err) { console.error(err); }
    })();
  }, [tick]);
```
Production mode: fetches all four resources in parallel, sets state, and applies the same default-prefill logic. Wraps everything in `try/catch` (logging to console on failure) — more defensive than most other pages.

```tsx
  const submit = async () => {
    if (!form.title || !form.customerId) return toast.error("Fill required fields");
    try {
      await orderService.create({ customerId: form.customerId, title: form.title, description: form.description, type: form.type, materials: form.matId ? [{ materialId: form.matId, qty: form.matQty }] : [] });
      toast.success("Order created");
      setOpen(false); refresh();
    } catch (e) { toast.error("Failed to create order"); }
  };
```
Validates title and customer are set; calls the real create API (always, not dev-branched) with a `materials` array containing one line item if a material was selected, or empty array otherwise; on success closes dialog and refreshes; on failure toasts error.

```tsx
  return (
    <>
      <PageHeader kicker={user!.role === 'employee' ? 'TASK BOARD' : 'Order Atelier'} title={user!.role === 'employee' ? 'My assigned tasks.' : 'All commissions.'} />
```
Header text adapts by role: employees see "TASK BOARD" / "My assigned tasks.", everyone else sees "Order Atelier" / "All commissions."

```tsx
      <div className="mb-3">
        <div className="flex flex-wrap items-center gap-3 mb-2">
          <Input placeholder="Search by order name" value={searchQ} onChange={e => setSearchQ(e.target.value)} className="w-60" />
          <Select value={statusFilter} onValueChange={(v) => setStatusFilter(v)}>
            <SelectTrigger className="w-44 h-8"><SelectValue /></SelectTrigger>
            <SelectContent>
              <SelectItem value="All">All</SelectItem>
              <SelectItem value="Pending">Pending</SelectItem>
              <SelectItem value="In Progress">In Progress</SelectItem>
              <SelectItem value="Completed">Completed</SelectItem>
            </SelectContent>
          </Select>
          <Button variant="outline" onClick={() => { /* noop - filters applied below */ refresh(); }} className="h-8">Search</Button>
        </div>
```
Search input (live-filters as you type — the "Search" button is actually a no-op that just triggers a data refresh, since filtering already happens reactively on every keystroke via the `.filter()` calls in the table below).

```tsx
        <div className="py-2 px-3 rounded-t-md bg-[#3d2b1f]">
          <div className="text-white font-bold uppercase">{user!.role === 'customer' ? 'MY ORDERS' : 'MY ASSIGNED ORDERS'}</div>
        </div>
      </div>
```
A dark-brown section-title bar above the table, wording differs slightly for customers vs. others (note: the "MY ASSIGNED ORDERS" label is technically shown to admins too, even though admins see *all* orders, not just assigned ones — a minor copy inconsistency).

```tsx
      <div className="rounded-sm border border-border bg-gradient-card overflow-hidden ring-inset-light">
        <table className="w-full text-sm">
          <thead>
            <tr className="bg-[#3d2b1f]">
              <th className="px-5 py-3 text-white font-bold uppercase">ORDER ID</th>
              <th className="px-5 py-3 text-white font-bold uppercase">ORDER NAME</th>
              <th className="px-5 py-3 text-white font-bold uppercase">CUSTOMER</th>
              <th className="px-5 py-3 text-white font-bold uppercase">TYPE</th>
              <th className="px-5 py-3 text-white font-bold uppercase">ORDER DATE</th>
              <th className="px-5 py-3 text-white font-bold uppercase">DESCRIPTION</th>
              <th className="px-5 py-3 text-white font-bold uppercase">STATUS</th>
              <th className="px-5 py-3 text-white font-bold uppercase">ACTION</th>
            </tr>
          </thead>
```
Static table header row with 8 dark-styled columns.

```tsx
          <tbody>
            {visible
              .filter(o => searchQ ? (o.title || '').toLowerCase().includes(searchQ.toLowerCase()) : true)
              .filter(o => statusFilter && statusFilter !== 'All' ? ((o.status || '') === statusFilter) : true)
              .map((o, idx) => {
              const rowBg = idx % 2 === 0 ? 'bg-white' : 'bg-[#fff7ef]';
```
Chains two filters on the role-scoped `visible` list: text search (case-insensitive substring match on title, only applied if `searchQ` is non-empty) and status filter (only applied if not "All"). Then maps to rows, alternating background color (zebra striping) based on even/odd index.

```tsx
              return (
                <tr key={o.id} className={`border-t border-border ${rowBg} text-[#2c1a0e]`}>
                  <td className="px-5 py-3 font-medium text-[#2c1a0e]"><button className="underline" onClick={() => { setSelectedOrder(o); }}>{o.id}</button></td>
                  <td className="px-5 py-3">{o.title}</td>
                  <td className="px-5 py-3">{o.customer}</td>
                  <td className="px-5 py-3">{o.type}</td>
                  <td className="px-5 py-3">{o.orderDate}</td>
                  <td className="px-5 py-3">{o.description || '—'}</td>
                  <td className="px-5 py-3"><StatusBadge status={o.status} /></td>
```
Each row: Order ID rendered as an underlined clickable button that opens the detail modal (`setSelectedOrder(o)`); plain text cells for name/customer/type/date/description (with em-dash fallback for missing description); a colored `StatusBadge` for status.

```tsx
                  <td className="px-5 py-3">
                    {user!.role === 'employee' ? (
                      <div className="flex items-center gap-2">
                        <button className="px-2 py-1 border border-amber-500 text-amber-700 rounded text-sm" onClick={() => { setSelectedOrder(o); setNewStatus(o.status || 'Not Started'); setStatusModalOpen(true); }}>Update Status</button>
                      </div>
                    ) : user!.role === 'customer' ? (
                      <div className="flex items-center gap-2">
                        <button className="px-2 py-1 border border-amber-500 text-amber-700 rounded text-sm" onClick={() => { setSelectedOrder(o); }}>View</button>
                        {o.status === 'Pending' && (
                          <button className="px-2 py-1 border border-red-500 text-red-700 rounded text-sm" onClick={() => setCancelConfirm(o.id)}>Cancel</button>
                        )}
                      </div>
                    ) : (
                      <StatusBadge status={o.status} />
                    )}
                  </td>
                </tr>
              );
            })}
          </tbody>
        </table>
      </div>
```
Action column varies by role: **employees** get an "Update Status" button that selects the order, seeds `newStatus` from the order's current status (or "Not Started" default), and opens the status modal. **Customers** get a "View" button (opens detail modal) plus a conditional "Cancel" button shown only while the order is still "Pending" (once work has started, cancellation isn't offered). **Admins** (the `else` branch) just see a redundant status badge (no interactive action) in this column.

```tsx
      {/* Order detail modal */}
      <Dialog open={!!selectedOrder} onOpenChange={(v) => { if (!v) setSelectedOrder(null); }}>
          <DialogContent className="max-w-lg">
```
Dialog visibility is derived from whether `selectedOrder` is truthy (`!!selectedOrder`). When the dialog is closed by the user (clicking outside/Esc), `onOpenChange` fires with `v=false`, which clears `selectedOrder`.

```tsx
          <DialogHeader><DialogTitle className="font-display text-2xl bg-[#3d2b1f] text-white p-2">ORDER DETAIL</DialogTitle></DialogHeader>
          {selectedOrder && (
            <div className="p-4">
              <div className="grid grid-cols-2 gap-3">
                <div><Label className="text-xs uppercase tracking-widest text-muted-foreground">Order ID</Label><div className="mt-1 font-medium">{selectedOrder.id}</div></div>
                <div><Label className="text-xs uppercase tracking-widest text-muted-foreground">Order Name</Label><div className="mt-1 font-medium">{selectedOrder.title}</div></div>
                <div><Label className="text-xs uppercase tracking-widest text-muted-foreground">Customer Name</Label><div className="mt-1 font-medium">{selectedOrder.customer}</div></div>
                <div><Label className="text-xs uppercase tracking-widest text-muted-foreground">Order Type</Label><div className="mt-1 font-medium">{selectedOrder.type}</div></div>
                <div><Label className="text-xs uppercase tracking-widest text-muted-foreground">Order Date</Label><div className="mt-1 font-medium">{selectedOrder.orderDate}</div></div>
                <div><Label className="text-xs uppercase tracking-widest text-muted-foreground">Assigned Date</Label><div className="mt-1 font-medium">{selectedOrder.assignedDate || '—'}</div></div>
                <div className="col-span-2"><Label className="text-xs uppercase tracking-widest text-muted-foreground">Description</Label><div className="mt-1 font-medium">{selectedOrder.description || '—'}</div></div>
                <div><Label className="text-xs uppercase tracking-widest text-muted-foreground">Estimated Cost</Label><div className="mt-1 font-medium">${selectedOrder.estimatedCost ?? '—'}</div></div>
                <div><Label className="text-xs uppercase tracking-widest text-muted-foreground">Final Cost</Label><div className="mt-1 font-medium">${selectedOrder.finalCost ?? '—'}</div></div>
                <div><Label className="text-xs uppercase tracking-widest text-muted-foreground">Current Status</Label><div className="mt-1 font-medium">{selectedOrder.status}</div></div>
                <div><Label className="text-xs uppercase tracking-widest text-muted-foreground">Assigned By</Label><div className="mt-1 font-medium">Admin</div></div>
                <div className="col-span-2"><Label className="text-xs uppercase tracking-widest text-muted-foreground">Materials Used</Label><div className="mt-1 font-medium">{(selectedOrder.materials || []).map((m:any)=>`${m.name || m.materialId} ${m.qty || m.qtyOrdered || m.quantity || ''}`).join(', ') || '—'}</div></div>
              </div>
            </div>
          )}
        </DialogContent>
      </Dialog>
```
A 2-column detail grid inside the modal, guarded by `selectedOrder &&` so it only renders when an order is actually selected. Notable fields: `estimatedCost`/`finalCost` use `??` (nullish coalescing, so `0` displays as `$0` rather than falling back to em-dash) with em-dash fallback only for actually missing values; "Assigned By" is hardcoded to "Admin" (no dynamic assignor tracking yet); Materials Used builds a human-readable string by mapping each material line item — trying several possible field-name variants (`name`/`materialId`, `qty`/`qtyOrdered`/`quantity`) to be resilient to inconsistent mock-data shapes — and joining with commas, falling back to em-dash if no materials.

```tsx
      {/* Status modal */}
      <Dialog open={statusModalOpen} onOpenChange={setStatusModalOpen}>
        <DialogContent className="max-w-sm">
          <DialogHeader><DialogTitle className="font-display text-lg bg-[#3d2b1f] text-white p-2">Update Status</DialogTitle></DialogHeader>
          <div className="p-4">
            <div className="space-y-2">
              <div>
                <Select value={newStatus} onValueChange={(v) => setNewStatus(v)}>
                  <SelectTrigger className="w-full h-9"><SelectValue /></SelectTrigger>
                  <SelectContent>
                    <SelectItem value="Not Started">Not Started</SelectItem>
                    <SelectItem value="In Progress">In Progress</SelectItem>
                    <SelectItem value="Work Completed">Work Completed</SelectItem>
                  </SelectContent>
                </Select>
              </div>
```
Employee-facing status modal: a dropdown of 3 possible statuses bound to `newStatus`.

```tsx
              <div className="flex justify-end gap-2 mt-4">
                <Button variant="outline" onClick={() => setStatusModalOpen(false)}>Cancel</Button>
                <Button variant="gold" onClick={async () => {
                  if (!selectedOrder) return;
                  try {
                    const statusToSave = newStatus === 'Work Completed' ? 'Completed' : newStatus;
```
Cancel just closes the modal (order stays selected). Save button: guards against no selected order; **maps** the UI label "Work Completed" to the backend/status-badge value `"Completed"` (since the dropdown's user-facing wording differs from the internal status vocabulary used elsewhere, e.g. `StatusBadge`).

```tsx
                    if (import.meta.env.DEV) {
                      setOrders(prev => prev.map(o => o.id === selectedOrder.id ? { ...o, status: statusToSave } : o));
                      const devItem = dev.devOrders.find(o => o.id === selectedOrder.id);
                      if (devItem) devItem.status = statusToSave;
                    } else {
                      await orderService.update(selectedOrder.id, { status: statusToSave });
                      refresh();
                    }
                    toast.success('Status updated');
                    setStatusModalOpen(false); setSelectedOrder(null);
                  } catch (e) { toast.error('Failed to update'); }
                }}>Save</Button>
              </div>
            </div>
          </div>
        </DialogContent>
      </Dialog>
```
Dev mode: updates local state immutably plus mutates the underlying mock record directly. Production: calls the real update API and refetches. Both paths close the modal, clear selection, and toast success (or error on failure).

```tsx
      {/* Cancel Order Confirmation */}
      <Dialog open={!!cancelConfirm} onOpenChange={(v) => { if (!v) setCancelConfirm(null); }}>
        <DialogContent className="max-w-sm">
          <DialogHeader><DialogTitle className="font-display text-lg">Cancel Order</DialogTitle></DialogHeader>
          {cancelConfirm && selectedOrder && (
```
Customer-facing cancel-confirmation dialog, shown when `cancelConfirm` holds an order ID. **Note:** it requires **both** `cancelConfirm` and `selectedOrder` to be truthy to render its body — but clicking "Cancel" in the table only sets `cancelConfirm`, not `selectedOrder` (that's only set by clicking the Order ID or "View" button). This means the confirmation dialog's inner content (which references `selectedOrder.title`) will only actually populate correctly if the user had previously viewed/selected that same order — otherwise the dialog would open with an empty body since the guard fails.

```tsx
            <div className="p-4">
              <div className="space-y-3">
                <p className="text-sm">Are you sure you want to cancel <strong>{selectedOrder.title}</strong>?</p>
                <p className="text-xs text-muted-foreground">This action cannot be undone.</p>
                <div className="flex justify-end gap-2 mt-4">
                  <Button variant="outline" onClick={() => { setCancelConfirm(null); setSelectedOrder(null); }}>Go Back</Button>
```
Confirmation copy referencing the order title; "Go Back" clears both state values, dismissing the dialog without cancelling.

```tsx
                  <Button variant="destructive" onClick={async () => {
                    try {
                      if (import.meta.env.DEV) {
                        setOrders(prev => prev.filter(o => o.id !== cancelConfirm));
                        toast.success('Order cancelled');
                        setCancelConfirm(null);
                        setSelectedOrder(null);
                        return;
                      }
                      await orderService.remove(cancelConfirm);
                      toast.success('Order cancelled');
                      setCancelConfirm(null); setSelectedOrder(null); refresh();
                    } catch (e) { toast.error('Failed to cancel order'); }
                  }}>Confirm Cancel</Button>
                </div>
              </div>
            </div>
          )}
        </DialogContent>
      </Dialog>
    </>
  );
}
```
"Confirm Cancel" (styled destructive/red): dev mode removes the order from local state entirely (note: unlike other dev-mode mutations, it does **not** also splice it out of `dev.devOrders`, so the underlying mock array still contains it — meaning a re-fetch via `tick` would bring it back); production calls the real delete API and refetches. Both show a success toast and reset the confirmation/selection state.

---

### 11. `Register.tsx`

```tsx
import { Link, useNavigate } from "react-router-dom";
import { useState } from "react";
import { useAuth } from "@/lib/auth";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import { toast } from "sonner";
import { Role } from "@/lib/store";
import { Logo } from "@/components/shared/Logo";
```
Standard imports; `Role` is a shared union type (`"admin" | "customer" | "employee"`, inferred from usage). Note the `Logo` import path here is `@/components/shared/Logo`, differing from `Login.tsx`'s `@/components/Logo` — possibly two different logo components or an inconsistent import path.

```tsx
export default function Register() {
  const { register } = useAuth();
  const nav = useNavigate();
  const [form, setForm] = useState({ name: "", email: "", password: "", role: "customer" as Role });
  const [registered, setRegistered] = useState(false);
  const [isAdmin, setIsAdmin] = useState(false);
```
Pulls `register` from auth; form state defaults role to `"customer"`; `registered` flags whether the "thank you" confirmation screen should replace the form; `isAdmin` remembers whether the just-registered account was an admin (to show a different confirmation message).

```tsx
  const submit = async (e: React.FormEvent) => {
    e.preventDefault();
    if (form.name.trim().length < 2) return toast.error("Name too short");
    if (!/^\S+@\S+\.\S+$/.test(form.email)) return toast.error("Invalid email format");
    if (form.password.length < 6) return toast.error("Password must be 6+ chars");
```
Three validations: trimmed name must be at least 2 characters; email must match a simple regex (`non-space characters, @, non-space characters, ., non-space characters` — a loose but reasonable email shape check); password must be 6+ characters.

```tsx
    const r = await register(form);
    if (!r.ok) return toast.error(r.error!);
```
Calls the auth hook's `register()`; if it fails, shows the returned error message (`!` asserts it's non-null, trusting that `ok: false` always comes with an `error` string).

```tsx
    setIsAdmin(form.role === 'admin');
    toast.success(form.role === 'admin' ? "Account created and signed in!" : "Registration submitted!");
    setRegistered(true);
```
Records whether this was an admin signup; shows a role-appropriate success toast; flips to the confirmation screen.

```tsx
    // Admin accounts auto-login and go to dashboard
    if (form.role === 'admin') {
      setTimeout(() => nav("/app", { replace: true }), 2000);
    } else {
      // Customer/Employee go to login page after approval message
      setTimeout(() => nav("/login", { replace: true }), 3000);
    }
  };
```
Business logic difference by role: **admin** accounts are auto-approved/logged-in, so after a 2-second delay (giving the user time to read the confirmation message) it redirects straight into the app. **Customer/employee** accounts require admin approval first, so after a longer 3-second delay it sends them to the login page instead (they can't get in yet).

```tsx
  if (registered) {
    if (isAdmin) {
      return (
        <div className="min-h-screen flex items-center justify-center p-6">
          <div className="w-full max-w-md text-center">
            <div className="mb-8"><Logo to="/" size="md" showTagline /></div>
            <div className="text-[11px] uppercase tracking-[0.3em] text-gold mb-3">WELCOME ADMIN</div>
            <h1 className="font-display text-4xl mb-4">Account Created!</h1>
            <p className="text-muted-foreground mb-6">Your admin account is active and ready to use. Redirecting to dashboard...</p>
          </div>
        </div>
      );
    }
```
Early-return pattern: once `registered` is true, the component renders one of two confirmation screens instead of the form. Admin variant: "Account Created!" with a redirecting message.

```tsx
    return (
      <div className="min-h-screen flex items-center justify-center p-6">
        <div className="w-full max-w-md text-center">
          <div className="mb-8"><Logo to="/" size="md" showTagline /></div>
          <div className="text-[11px] uppercase tracking-[0.3em] text-gold mb-3">REGISTRATION SUBMITTED</div>
          <h1 className="font-display text-4xl mb-4">Thank you for joining!</h1>
          <p className="text-muted-foreground mb-6">Your account request has been sent to our admin team. You will receive an email notification once your account is approved. This usually takes 24 hours.</p>
          <Button variant="gold" onClick={() => nav("/login")} className="w-full">Back to Sign In</Button>
        </div>
      </div>
    );
  }
```
Non-admin variant: explains the pending-approval flow and offers a manual "Back to Sign In" button (in addition to the automatic 3-second redirect already scheduled).

```tsx
  return (
    <div className="min-h-screen flex items-center justify-center p-6">
      <div className="w-full max-w-md">
        <div className="mb-8"><Logo to="/" size="md" showTagline /></div>
        <div className="text-[11px] uppercase tracking-[0.3em] text-gold mb-3">Create account</div>
        <h1 className="font-display text-4xl mb-2">Join the atelier.</h1>
        <p className="text-muted-foreground mb-8">Start commissioning timeless pieces.</p>
```
The default view (form) — logo, kicker text, headline, subtext.

```tsx
        <form onSubmit={submit} className="space-y-4">
          <div><Label>Full name</Label><Input value={form.name} onChange={e => setForm({ ...form, name: e.target.value })} /></div>
          <div><Label>Email</Label><Input type="email" value={form.email} onChange={e => setForm({ ...form, email: e.target.value })} /></div>
          <div><Label>Password</Label><Input type="password" value={form.password} onChange={e => setForm({ ...form, password: e.target.value })} /></div>
```
Controlled name/email/password inputs, each updating one key of `form` via inline spread (rather than a separate helper function like the profile pages use).

```tsx
          <div>
            <Label>Account type</Label>
            <div className="grid grid-cols-3 gap-2 mt-1">
              {(["customer", "employee", "admin"] as Role[]).map(r => (
                <button key={r} type="button" onClick={() => setForm({ ...form, role: r })}
                  className={"text-xs uppercase tracking-widest px-3 py-2 border rounded-sm transition " +
                    (form.role === r ? "border-gold text-gold bg-secondary" : "border-border text-muted-foreground hover:text-foreground")}>
                  {r}
                </button>
              ))}
            </div>
          </div>
```
Three-button role picker (acting like radio buttons): maps over a hardcoded array of roles, each button sets `form.role` on click; the currently-selected role gets gold-highlighted styling, others stay muted.

```tsx
          <Button type="submit" variant="gold" size="lg" className="w-full">Create account</Button>
        </form>

        <p className="mt-6 text-sm text-muted-foreground">
          Already a member? <Link to="/login" className="text-gold hover:underline">Sign in</Link>
        </p>
      </div>
    </div>
  );
}
```
Submit button and a link back to the login page for existing users.

---

### 12. `Reports.tsx`

```tsx
/**
 * FILE: src/pages/Reports.tsx
 * PURPOSE: Admin analytics: income vs expenses, top materials, productivity charts.
 * TYPE: Page
 * DEPENDENCIES: @/services/orderService, @/services/billingService, @/services/inventoryService, @/services/employeeService, recharts, @/components/ui/*
 * USED BY: src/App.tsx
 */
```
A structured JSDoc-style file-header comment documenting the file's purpose, type, dependencies, and where it's consumed — a convention used only in this file and `Worklog.tsx` (suggesting these two were documented more recently/carefully than the rest).

```tsx
import { PageHeader } from "@/components/layout/AppShell";
import * as orderService from "@/services/orderService";
import * as billingService from "@/services/billingService";
import * as inventoryService from "@/services/inventoryService";
import * as employeeService from "@/services/employeeService";
import dev from "@/lib/devData";
import { useEffect, useMemo, useState } from "react";
import { ResponsiveContainer, BarChart, Bar, XAxis, YAxis, CartesianGrid, Tooltip, PieChart, Pie, Cell, Legend, LineChart, Line } from "recharts";
```
Imports the four data services needed (orders, billing/payments, inventory, employees), and a large set of `recharts` chart-building components.

```tsx
const COLORS = ["hsl(var(--gold))", "hsl(41 70% 68%)", "hsl(38 50% 42%)", "hsl(28 30% 40%)", "hsl(24 35% 25%)"];
```
A 5-color palette (using CSS custom-property-based HSL colors for theme consistency) used to color the pie chart slices.

```tsx
export default function Reports() {
  const [orders, setOrders] = useState<any[]>([]);
  const [materials, setMaterials] = useState<any[]>([]);
  const [payments, setPayments] = useState<any[]>([]);
  const [employees, setEmployees] = useState<any[]>([]);
```
Four data lists needed across the various charts.

```tsx
  useEffect(() => {
    (async () => {
      try {
        if (import.meta.env.DEV) {
          const mapMaterials = (arr: any[]) => arr.map(m => ({ id: m.id, name: m.name, price: m.unitPrice || m.price || 0 }));
          setOrders(dev.devOrders.map((o: any) => ({ ...o, assignedEmployeeId: o.employeeId || o.assignedEmployeeId })));
          setMaterials(mapMaterials(dev.devMaterials));
          setPayments(dev.devPayments);
          setEmployees(dev.devEmployees.filter((u: any) => u.role && u.role.toLowerCase().includes("carpenter") || u.role === "Repair Specialist" || u.role === "Furniture Maker" ? true : u.role === "Employee"));
          return;
        }
```
Dev mode: normalizes orders/materials as elsewhere; loads payments directly; **employees filter is oddly written** — due to JS operator precedence, this evaluates as `(u.role && u.role.toLowerCase().includes("carpenter")) || (u.role === "Repair Specialist") || (u.role === "Furniture Maker" ? true : u.role === "Employee")`. In effect it keeps any employee whose role contains "carpenter" (case-insensitive), OR is exactly "Repair Specialist", OR is exactly "Furniture Maker" (redundant `? true :` ternary — could just be `||`), OR (as the final fallback of that ternary) is exactly "Employee". This is a somewhat convoluted/likely-buggy filter for narrowing mock employees down to production-floor roles.

```tsx
        const [o, m, p, e] = await Promise.all([
          orderService.list(),
          inventoryService.list(),
          billingService.list(),
          employeeService.list(),
        ]);
        setOrders(o);
        setMaterials(m);
        setPayments(p);
        setEmployees(e.filter((u: any) => u.role === "employee"));
      } catch (err) {
        console.error(err);
      }
    })();
  }, []);
```
Production mode: fetches all four in parallel, and filters employees down to those with `role === "employee"` (much simpler/cleaner than the dev-mode filter above — an inconsistency between dev and prod filtering logic).

```tsx
  const monthly = useMemo(() => {
    // build a quick lookup for material prices
    const matPrice = new Map<string, number>();
    materials.forEach((m: any) => matPrice.set(String(m.id), Number(m.price || 0)));
```
`useMemo` recomputes only when dependencies change (listed at the bottom). Builds a `Map` from material ID → price for fast lookup.

```tsx
    const m = new Map<string, { month: string; income: number; expenses: number }>();

    // accumulate income from payments
    payments.forEach((p: any) => {
      const dateStr = p.date || p.createdAt || p.created_at || new Date().toISOString();
      const k = new Date(dateStr).toLocaleString("en", { month: "short" });
      const e = m.get(k) || { month: k, income: 0, expenses: 0 };
      e.income += Number(p.amount || 0);
      m.set(k, e);
    });
```
Builds a map keyed by short month name (e.g. "Jan"). For each payment: determines its date from several possible field names, falling back to "now" if none present; derives the short month label; gets or creates that month's accumulator object; adds the payment amount to `income`; stores it back.

```tsx
    // accumulate expenses from orders' materials
    orders.forEach((o: any) => {
      const dateStr = o.createdAt || o.created_at || new Date().toISOString();
      const k = new Date(dateStr).toLocaleString("en", { month: "short" });
      const e = m.get(k) || { month: k, income: 0, expenses: 0 };
      let orderMatCost = 0;
      (o.materials || []).forEach((om: any) => {
        const matId = String(om.materialId ?? om.material_id ?? om.id ?? "");
        const qty = Number(om.qty ?? om.quantity ?? 0);
        orderMatCost += (matPrice.get(matId) || 0) * qty;
      });
      e.expenses += orderMatCost;
      m.set(k, e);
    });
```
For each order: similarly determines its month; sums material cost across its line items by looking up each material's price via the `matPrice` map (checking several possible ID field names via `??`) multiplied by quantity; adds that total to the month's `expenses`.

```tsx
    // produce array sorted by month order (Jan..Dec)
    const monthOrder = ["Jan","Feb","Mar","Apr","May","Jun","Jul","Aug","Sep","Oct","Nov","Dec"];
    const arr = Array.from(m.values()).sort((a, b) => monthOrder.indexOf(a.month) - monthOrder.indexOf(b.month));
    return arr.length > 0 ? arr : [{ month: "No data", income: 0, expenses: 0 }];
  }, [payments, orders, materials]);
```
Converts the map's values to an array, sorts chronologically (Jan first) rather than insertion order, and falls back to a placeholder "No data" entry if the array would otherwise be empty (so the chart renders something instead of a blank space). Recomputes when `payments`, `orders`, or `materials` change.

```tsx
  const materialUsage = useMemo(() => {
    const map = new Map<string, number>();
    orders.forEach((o: any) => {
      // Handle both materials array and material_id relationships
      if (o.materials && Array.isArray(o.materials)) {
        o.materials.forEach((om: any) => {
          const matId = om.materialId || om.material_id;
          if (matId) map.set(matId, (map.get(matId) || 0) + (om.qty || om.quantity || 0));
        });
      }
    });
```
Builds a map of material ID → total quantity used across all orders (only if a valid material ID was found).

```tsx
    const result = Array.from(map, ([id, qty]) => ({
      name: materials.find((m: any) => m.id === id)?.name || `Material ${id}`,
      qty
    })).sort((a, b) => b.qty - a.qty).slice(0, 5);
    return result.length > 0 ? result : [{ name: "No data", qty: 0 }];
  }, [orders, materials]);
```
Converts the map's entries to `{ name, qty }` objects (resolving each material's display name from the `materials` list, falling back to a generic "Material {id}" label if not found), sorts descending by quantity, and keeps only the top 5. Falls back to a placeholder if empty.

```tsx
  const productivity = employees.map((e: any) => {
    const eo = orders.filter((o: any) => o.assignedEmployeeId === e.id);
    return { name: e.name.split(" ")[0], hours: eo.reduce((s: number, o: any) => s + o.laborHours, 0), tasks: eo.length };
  });
```
**Not memoized** (recomputed on every render, unlike the other two derived datasets) — for each employee: finds their assigned orders, computes total hours and task count, and uses just their first name for the chart label.

```tsx
  const completion = ["Pending", "In Progress", "Completed", "Delivered"].map(s => ({
    name: s, value: orders.filter((o: any) => o.status === s).length,
  }));
```
Also not memoized — counts orders in each of the four possible statuses, building data for the pie chart.

```tsx
  return (
    <>
      <PageHeader kicker="Atelier Analytics" title="Reports." />

      <div className="grid lg:grid-cols-2 gap-4 mb-4">
        <Card title="Monthly income" subtitle="Payments received">
          <ResponsiveContainer width="100%" height={288}>
            <LineChart data={monthly}>
              <CartesianGrid stroke="hsl(var(--border))" strokeDasharray="3 3" vertical={false} />
              <XAxis dataKey="month" stroke="hsl(var(--muted-foreground))" fontSize={11} tickLine={false} axisLine={false} />
              <YAxis stroke="hsl(var(--muted-foreground))" fontSize={11} tickLine={false} axisLine={false} />
              <Tooltip contentStyle={{ background: "hsl(var(--popover))", border: "1px solid hsl(var(--border))" }} />
              <Legend wrapperStyle={{ fontSize: 11 }} />
              <Line type="monotone" dataKey="income" stroke="hsl(var(--gold))" strokeWidth={3} dot={{ fill: "hsl(var(--gold))", r: 5 }} name="Income" />
              <Line type="monotone" dataKey="expenses" stroke="hsl(0 60% 55%)" strokeWidth={3} dot={{ r: 4 }} name="Expenses" />
            </LineChart>
          </ResponsiveContainer>
        </Card>
```
First chart: a 2-column responsive grid layout, wrapped in the local `Card` helper (defined at the bottom of the file). A dual-line chart plotting `income` (gold line) and `expenses` (red line) per month, with grid lines, axis labels, a hover tooltip, and a legend.

```tsx
        <Card title="Order completion" subtitle="By status">
          <ResponsiveContainer>
            <PieChart>
              <Pie data={completion} dataKey="value" nameKey="name" innerRadius={50} outerRadius={90} paddingAngle={2}>
                {completion.map((_: any, i: number) => <Cell key={i} fill={COLORS[i % COLORS.length]} />)}
              </Pie>
              <Legend wrapperStyle={{ fontSize: 11 }} />
              <Tooltip contentStyle={{ background: "hsl(var(--popover))", border: "1px solid hsl(var(--border))" }} />
            </PieChart>
          </ResponsiveContainer>
        </Card>
```
Second chart: a donut-style pie chart (`innerRadius` > 0 makes it a ring) of order counts by status, with each slice colored by cycling through the `COLORS` array (`i % COLORS.length` wraps around if there are more slices than colors).

```tsx
        <Card title="Most used materials" subtitle="Top 5">
          <ResponsiveContainer>
            <BarChart data={materialUsage} layout="vertical">
              <CartesianGrid stroke="hsl(var(--border))" strokeDasharray="3 3" horizontal={false} />
              <XAxis type="number" stroke="hsl(var(--muted-foreground))" fontSize={11} tickLine={false} axisLine={false} />
              <YAxis type="category" dataKey="name" stroke="hsl(var(--muted-foreground))" fontSize={11} tickLine={false} axisLine={false} width={130} />
              <Tooltip contentStyle={{ background: "hsl(var(--popover))", border: "1px solid hsl(var(--border))" }} />
              <Bar dataKey="qty" fill="hsl(var(--gold))" radius={[0, 2, 2, 0]} />
            </BarChart>
          </ResponsiveContainer>
        </Card>
```
Third chart: a **horizontal** bar chart (`layout="vertical"` in recharts terminology flips axes so bars extend sideways) of the top 5 materials by quantity used, with material names on the Y-axis (given extra width for long labels) and quantity on the X-axis.

```tsx
        <Card title="Artisan productivity" subtitle="Hours logged">
          <ResponsiveContainer>
            <BarChart data={productivity}>
              <CartesianGrid stroke="hsl(var(--border))" strokeDasharray="3 3" vertical={false} />
              <XAxis dataKey="name" stroke="hsl(var(--muted-foreground))" fontSize={11} tickLine={false} axisLine={false} />
              <YAxis stroke="hsl(var(--muted-foreground))" fontSize={11} tickLine={false} axisLine={false} />
              <Tooltip contentStyle={{ background: "hsl(var(--popover))", border: "1px solid hsl(var(--border))" }} />
              <Bar dataKey="hours" fill="hsl(var(--gold))" radius={[2, 2, 0, 0]} />
              <Bar dataKey="tasks" fill="hsl(38 50% 42%)" radius={[2, 2, 0, 0]} />
            </BarChart>
          </ResponsiveContainer>
        </Card>
      </div>
    </>
  );
}
```
Fourth chart: a grouped vertical bar chart with two bars per employee (hours in gold, tasks in a darker gold/bronze), X-axis labeled by first name.

```tsx
function Card({ title, subtitle, children }: { title: string; subtitle?: string; children: React.ReactNode }) {
  return (
    <div className="rounded-sm border border-border bg-gradient-card p-6 ring-inset-light">
      <div className="text-[10px] uppercase tracking-[0.3em] text-muted-foreground">{subtitle}</div>
      <div className="font-display text-2xl mt-1 mb-6">{title}</div>
      <div className="h-72">{children}</div>
    </div>
  );
}
```
A small local (non-exported) helper component used only within this file: renders a consistent card wrapper with an optional subtitle line, a title, and a fixed-height (`h-72`) container for whatever chart is passed as `children`.

---

### 13. `UserAccess.tsx`

```tsx
import { useState, useEffect } from "react";
import { PageHeader } from "@/components/layout/AppShell";
import { Button } from "@/components/ui/button";
import { useAuth } from "@/lib/auth";
import { Dialog, DialogContent, DialogHeader, DialogTitle } from "@/components/ui/dialog";
import { toast } from "sonner";
import { CheckCircle, XCircle, Lock, Unlock } from "lucide-react";
import dev from "@/lib/devData";
```
Standard imports plus icons for approve/reject/block/unblock actions.

```tsx
export default function UserAccess() {
  const { user } = useAuth();
  const [tick, setTick] = useState(0);
  const [pendingRequests, setPendingRequests] = useState<any[]>([]);
  const [activeUsers, setActiveUsers] = useState<any[]>([]);
  const [blockedUsers, setBlockedUsers] = useState<any[]>([]);
  const [rejectedLog, setRejectedLog] = useState<any[]>([]);
  const [selectedRequest, setSelectedRequest] = useState<any>(null);
  const [showDetail, setShowDetail] = useState(false);
```
`user` is fetched but not actually used in this file's logic (likely reserved for a future permission check). Four parallel lists represent the four sections of this page. `selectedRequest`/`showDetail` appear to support a request-detail modal, but (as seen below) neither is actually wired to any click handler in the rendered table — dead state.

```tsx
  useEffect(() => {
    if (import.meta.env.DEV) {
      // Load pending requests from localStorage (created during registration)
      const devPendingReqs = JSON.parse(localStorage.getItem("dev_pending_requests") || "[]");
```
Reads any pending-signup requests that were stashed in `localStorage` by the Register flow (parses the JSON string, defaulting to an empty array literal if the key doesn't exist).

```tsx
      const devRequestsFromData = dev.devUserRequests || [];
      const allRequests = [...devPendingReqs, ...devRequestsFromData].filter((r, idx, self) => self.findIndex(x => x.id === r.id) === idx);
```
Combines localStorage-persisted requests with any seeded mock requests, then **de-duplicates** by ID: for each item, `self.findIndex(...)` finds the *first* occurrence of that ID in the combined array; if the current item's index equals that first-found index, it's the first occurrence and is kept, otherwise it's a duplicate and filtered out.

```tsx
      setPendingRequests(allRequests);
      setActiveUsers(dev.devActiveUsers);
      setBlockedUsers(dev.devBlockedUsers);
      setRejectedLog(dev.devRejectedLog);
    }
  }, [tick]);
```
Sets all four lists from mock/localStorage data. **Note:** unlike other pages, there is no `else` branch here for production mode — this page currently only functions in dev mode; in production the lists would stay empty since no service call is made.

```tsx
  const refresh = () => setTick(t => t + 1);
```
Re-fetch trigger, defined after the effect (works fine in JS due to function-scoping/hoisting-independent closures, though it's stylistically unusual to declare it below its usage in the dependency array — it's fine since `refresh` isn't in the dependency array, only `tick` is).

```tsx
  const approvePending = (id: string) => {
    const request = pendingRequests.find(r => r.id === id);
    if (!request) return;
    setPendingRequests(prev => prev.filter(r => r.id !== id));
    setActiveUsers(prev => [...prev, { 
      id: request.id, 
      type: request.type, 
      name: request.name, 
      username: request.username, 
      email: request.email, 
      status: 'Active', 
      approvedDate: new Date().toISOString().split('T')[0],
      lastLogin: null 
    }]);
    toast.success(`${request.name} approved`);
  };
```
Finds the matching pending request by ID; bails if not found; removes it from `pendingRequests`; adds a new record to `activeUsers` built from the request's fields plus a computed `approvedDate` (today's date, extracted from the ISO timestamp by splitting on `"T"` and taking the date portion, e.g. `"2026-06-15"`) and `lastLogin: null` (never logged in yet). Shows a success toast. **Note:** this is purely local state — it does **not** persist to `localStorage` or a backend, so a page refresh would lose the approval.

```tsx
  const rejectPending = (id: string) => {
    const request = pendingRequests.find(r => r.id === id);
    if (!request) return;
    setPendingRequests(prev => prev.filter(r => r.id !== id));
    setRejectedLog(prev => [...prev, {
      id: `LOG-${Math.floor(Math.random()*900)+100}`,
      type: request.type,
      name: request.name,
      email: request.email,
      action: 'Rejected',
      date: new Date().toISOString().split('T')[0],
      doneBy: 'Admin-01'
    }]);
    toast.success(`${request.name} rejected`);
  };
```
Similar pattern: removes from pending, adds an entry to `rejectedLog` with a randomly-generated log ID (`Math.floor(Math.random()*900)+100` produces an integer from 100–999), today's date, and a hardcoded `doneBy: 'Admin-01'` (not tied to the actual logged-in admin's identity).

```tsx
  const blockUser = (id: string) => {
    const user = activeUsers.find(u => u.id === id);
    if (!user) return;
    setActiveUsers(prev => prev.filter(u => u.id !== id));
    setBlockedUsers(prev => [...prev, {
      id: user.id,
      type: user.type,
      name: user.name,
      email: user.email,
      blockedDate: new Date().toISOString().split('T')[0],
      reason: 'Blocked by admin'
    }]);
    toast.success(`${user.name} blocked`);
  };
```
Moves a user from `activeUsers` to `blockedUsers`, stamping today's date and a generic hardcoded reason. **Note:** this local `user` variable shadows the outer `useAuth()`'s `user` within this function's scope — harmless here since the outer `user` isn't referenced inside, but a naming collision worth flagging.

```tsx
  const unblockUser = (id: string) => {
    const user = blockedUsers.find(u => u.id === id);
    if (!user) return;
    setBlockedUsers(prev => prev.filter(u => u.id !== id));
    setActiveUsers(prev => [...prev, {
      id: user.id,
      type: user.type,
      name: user.name,
      username: user.email.split('@')[0],
      email: user.email,
      status: 'Active',
      approvedDate: new Date().toISOString().split('T')[0],
      lastLogin: null
    }]);
    toast.success(`${user.name} unblocked`);
  };
```
Reverses `blockUser`: moves the user back to `activeUsers`, re-deriving a `username` from the email's local-part (everything before `@`) since blocked-user records don't store a username separately, and resetting `approvedDate`/`lastLogin` as if freshly re-approved.

```tsx
  return (
    <>
      <PageHeader kicker="ADMIN CONTROL" title="User access management." />

      {/* PENDING REQUESTS SECTION */}
      <div className="mb-10">
        <div className="py-2 px-3 rounded-t-md bg-[#3d2b1f]">
          <div className="text-white font-bold uppercase">PENDING REQUESTS ({pendingRequests.length})</div>
        </div>
```
Section header showing a live count of pending requests.

```tsx
        <div className="rounded-sm border border-border bg-gradient-card overflow-hidden">
          <table className="w-full text-sm">
            <thead>
              <tr className="bg-[#3d2b1f]">
                <th className="px-5 py-3 text-white font-bold uppercase">REQUEST ID</th>
                <th className="px-5 py-3 text-white font-bold uppercase">TYPE</th>
                <th className="px-5 py-3 text-white font-bold uppercase">NAME</th>
                <th className="px-5 py-3 text-white font-bold uppercase">EMAIL</th>
                <th className="px-5 py-3 text-white font-bold uppercase">REQUESTED DATE</th>
                <th className="px-5 py-3 text-white font-bold uppercase">ACTION</th>
              </tr>
            </thead>
            <tbody>
              {pendingRequests.length === 0 ? (
                <tr><td colSpan={6} className="px-5 py-8 text-center text-muted-foreground">No pending requests</td></tr>
              ) : (
```
6-column table header; empty state spans all columns with a centered message when there are no requests.

```tsx
                pendingRequests.map((req, idx) => (
                  <tr key={req.id} className={`border-t border-border ${idx%2===0 ? 'bg-white' : 'bg-[#fff7ef]'} text-[#2c1a0e]`}>
                    <td className="px-5 py-3 font-medium">{req.id}</td>
                    <td className="px-5 py-3"><span className="px-2 py-1 bg-blue-100 text-blue-800 text-xs rounded">{req.type}</span></td>
                    <td className="px-5 py-3">{req.name}</td>
                    <td className="px-5 py-3">{req.email}</td>
                    <td className="px-5 py-3">{req.date}</td>
                    <td className="px-5 py-3 flex gap-2">
                      <Button size="sm" variant="gold" onClick={() => approvePending(req.id)}><CheckCircle className="h-4 w-4" /> Approve</Button>
                      <Button size="sm" variant="outline" onClick={() => rejectPending(req.id)}><XCircle className="h-4 w-4" /> Reject</Button>
                    </td>
                  </tr>
                ))
```
Zebra-striped rows; type shown as a blue pill; Approve/Reject buttons wired to the handlers above.

```tsx
              )}
            </tbody>
          </table>
        </div>
      </div>
```
Closes the conditional and table.

```tsx
      {/* ACTIVE USERS SECTION */}
      <div className="mb-10">
        <div className="py-2 px-3 rounded-t-md bg-[#3d2b1f]">
          <div className="text-white font-bold uppercase">ACTIVE USERS ({activeUsers.length})</div>
        </div>
        <div className="rounded-sm border border-border bg-gradient-card overflow-hidden">
          <table className="w-full text-sm">
            <thead>
              <tr className="bg-[#3d2b1f]">
                <th className="px-5 py-3 text-white font-bold uppercase">USER ID</th>
                <th className="px-5 py-3 text-white font-bold uppercase">TYPE</th>
                <th className="px-5 py-3 text-white font-bold uppercase">NAME</th>
                <th className="px-5 py-3 text-white font-bold uppercase">USERNAME</th>
                <th className="px-5 py-3 text-white font-bold uppercase">EMAIL</th>
                <th className="px-5 py-3 text-white font-bold uppercase">APPROVED DATE</th>
                <th className="px-5 py-3 text-white font-bold uppercase">LAST LOGIN</th>
                <th className="px-5 py-3 text-white font-bold uppercase">ACTION</th>
              </tr>
            </thead>
            <tbody>
              {activeUsers.length === 0 ? (
                <tr><td colSpan={8} className="px-5 py-8 text-center text-muted-foreground">No active users</td></tr>
              ) : (
                activeUsers.map((u, idx) => (
                  <tr key={u.id} className={`border-t border-border ${idx%2===0 ? 'bg-white' : 'bg-[#fff7ef]'} text-[#2c1a0e]`}>
                    <td className="px-5 py-3 font-medium">{u.id}</td>
                    <td className="px-5 py-3"><span className={`px-2 py-1 text-xs rounded ${u.type === 'Employee' ? 'bg-green-100 text-green-800' : 'bg-purple-100 text-purple-800'}`}>{u.type}</span></td>
                    <td className="px-5 py-3">{u.name}</td>
                    <td className="px-5 py-3">{u.username}</td>
                    <td className="px-5 py-3 text-sm">{u.email}</td>
                    <td className="px-5 py-3">{u.approvedDate}</td>
                    <td className="px-5 py-3">{u.lastLogin || '—'}</td>
                    <td className="px-5 py-3">
                      <Button size="sm" variant="outline" onClick={() => blockUser(u.id)}><Lock className="h-4 w-4" /> Block</Button>
                    </td>
                  </tr>
                ))
              )}
            </tbody>
          </table>
        </div>
      </div>
```
8-column table; type pill color-codes Employee (green) vs. other (purple, i.e., Customer); "Block" button per row calls `blockUser`.

```tsx
      {/* BLOCKED USERS SECTION */}
      <div className="mb-10">
        ... (structurally identical table pattern) ...
                    <td className="px-5 py-3">
                      <Button size="sm" variant="gold" onClick={() => unblockUser(u.id)}><Unlock className="h-4 w-4" /> Unblock</Button>
                    </td>
        ...
      </div>
```
Same table structure as Active Users, but with a "Reason" column instead of Username/Approved/LastLogin, and an "Unblock" action button.

```tsx
      {/* REJECTED / DELETED LOG SECTION */}
      <div>
        ... (same structural pattern, read-only — no action column) ...
      </div>
    </>
  );
}
```
A pure audit-log table (no interactive buttons): Log ID, Type, Name, Email, Action (color-coded red for "Rejected", gray otherwise), Date, Done By.

---

### 14. `WageSummary.tsx`

```tsx
import { PageHeader } from "@/components/layout/AppShell";
import dev from "@/lib/devData";
```
Minimal imports. `dev` is imported but, notably, **not actually used anywhere** in this file — it's dead/unused (all data here is hardcoded inline).

```tsx
export default function WageSummary() {
  // For development show the provided summary for Bilal
  const monthly = [
    { month: 'April 2026', hours: 12, rate: 10, wages: 120, status: 'Paid' },
    { month: 'May 2026', hours: 22, rate: 10, wages: 220, status: 'Paid' },
    { month: 'June 2026', hours: 27, rate: 10, wages: 270, status: 'Pending' },
  ];
```
A **fully hardcoded** array of 3 months of wage data for a specific demo employee ("Bilal", per the comment) — not fetched from any service or dev-data module. This is clearly a placeholder/demo implementation rather than a production-ready feature.

```tsx
  const totalHours = monthly.reduce((s, m) => s + m.hours, 0);
  const totalWages = monthly.reduce((s, m) => s + m.wages, 0);
```
Sums hours and wages across all three hardcoded months.

```tsx
  return (
    <>
      <PageHeader kicker="EARNINGS OVERVIEW" title="My wage summary." />

      <div className="grid lg:grid-cols-3 gap-4 mb-8">
        <div className="rounded-sm border border-border bg-gradient-card p-6">
          <div className="stat-label">TOTAL EARNED (ALL TIME)</div>
          <div className="font-display text-3xl value-amber mt-2">${610}</div>
        </div>
```
Three stat cards. **Note:** "Total Earned (All Time)" is hardcoded to the literal `${610}` rather than computed from `totalWages` (which happens to equal 610 given the data above, but would silently go stale if the `monthly` array were ever edited without also updating this number) — a maintainability smell.

```tsx
        <div className="rounded-sm border border-border bg-gradient-card p-6">
          <div className="stat-label">THIS MONTH WAGES</div>
          <div className="font-display text-3xl text-[#2c1a0e] mt-2">${monthly[2].wages}</div>
        </div>
        <div className="rounded-sm border border-border bg-gradient-card p-6">
          <div className="stat-label">PAYMENT STATUS</div>
          <div className="font-display text-3xl text-[#2c1a0e] mt-2">{monthly[2].status}</div>
        </div>
      </div>
```
"This Month" cards pull from `monthly[2]` (hardcoded index assuming June/index 2 is always "this month" — brittle if the array changes).

```tsx
      <div className="rounded-sm border border-border bg-gradient-card overflow-hidden">
        <div className="py-2 px-3 rounded-t-md bg-[#3d2b1f]">
          <div className="text-white font-bold uppercase">MONTHLY WAGE SUMMARY</div>
        </div>
        <table className="w-full text-sm">
          <thead>
            <tr className="bg-[#3d2b1f]">
              <th className="px-5 py-3 text-white font-bold uppercase">Month</th>
              <th className="px-5 py-3 text-white font-bold uppercase">Total Hours</th>
              <th className="px-5 py-3 text-white font-bold uppercase">Hourly Rate</th>
              <th className="px-5 py-3 text-white font-bold uppercase">Total Wages</th>
              <th className="px-5 py-3 text-white font-bold uppercase">Payment Status</th>
            </tr>
          </thead>
          <tbody>
            {monthly.map((m, idx) => (
              <tr key={m.month} className={`border-t border-border ${idx%2===0 ? 'bg-white' : 'bg-[#fff7ef]'} text-[#2c1a0e]`}>
                <td className="px-5 py-4">{m.month}</td>
                <td className="px-5 py-4">{m.hours.toFixed(2)} hrs</td>
                <td className="px-5 py-4">${m.rate}/hr</td>
                <td className="px-5 py-4 text-[#b8860b] font-bold">${m.wages.toFixed(2)}</td>
                <td className="px-5 py-4">{m.status}</td>
              </tr>
            ))}
```
Standard zebra-striped table row per month, using `m.month` as the React key (safe here since months are unique strings).

```tsx
            <tr className="border-t border-border bg-[#f5ede3]">
              <td className="px-5 py-4 font-bold">Total</td>
              <td className="px-5 py-4 font-bold">{totalHours.toFixed(2)} hrs</td>
              <td className="px-5 py-4">—</td>
              <td className="px-5 py-4 font-bold text-[#b8860b]">${totalWages.toFixed(2)}</td>
              <td className="px-5 py-4">—</td>
            </tr>
          </tbody>
        </table>
      </div>
    </>
  );
}
```
A summary/totals row (distinct beige background) at the bottom, using the properly-**computed** `totalHours`/`totalWages` (unlike the top stat card, which was hardcoded).

---

### 15. `Worklog.tsx`

```tsx
/**
 * FILE: src/pages/Worklog.tsx
 * PURPOSE: Time-tracking module: aggregates labor hours and computed wages per artisan.
 * TYPE: Page
 * DEPENDENCIES: @/services/orderService, @/services/employeeService, @/components/ui/*
 * USED BY: src/App.tsx
 */
```
Same documented-header convention as `Reports.tsx`.

```tsx
import { useEffect, useState } from "react";
import { PageHeader } from "@/components/layout/AppShell";
import { StatusBadge } from "@/components/ui/stat-card";
import { useAuth } from "@/lib/auth";
import * as orderService from "@/services/orderService";
import * as employeeService from "@/services/employeeService";
import dev from "@/lib/devData";
import { Clock } from "lucide-react";
import * as worklogService from "@/services/worklogService";
import { Dialog, DialogContent, DialogHeader, DialogTitle } from "@/components/ui/dialog";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "@/components/ui/select";
import { Button } from "@/components/ui/button";
import { toast } from "sonner";
```
Standard imports plus a dedicated `worklogService`. `StatusBadge`/`Clock` icon are imported but `Clock` is never actually used in the JSX (unused import).

```tsx
export default function Worklog() {
  const { user } = useAuth();
  const [orders, setOrders] = useState<any[]>([]);
  const [employees, setEmployees] = useState<any[]>([]);
  const [adding, setAdding] = useState(false);
  const [newEntry, setNewEntry] = useState({ orderId: '', hours: 0, note: '', date: new Date().toISOString().slice(0,10) });
  const [worklogs, setWorklogs] = useState<any[]>([]);
```
`adding` controls the "Add Entry" dialog. `newEntry` is the new-worklog form, pre-filled with today's date (`toISOString().slice(0,10)` extracts just the `YYYY-MM-DD` portion).

```tsx
  const [searchOrderName, setSearchOrderName] = useState('');
  const [monthFilter, setMonthFilter] = useState('All');
  const [dateFrom, setDateFrom] = useState('');
  const [dateTo, setDateTo] = useState('');
```
Four independent filter controls for the worklog table.

```tsx
  useEffect(() => {
    (async () => {
      try {
        if (import.meta.env.DEV) {
          setOrders(dev.devOrders.map((o: any) => ({ ...o, assignedEmployeeId: o.employeeId || o.assignedEmployeeId, laborHours: o.laborHours || 0 })));
          setEmployees(dev.devEmployees.map((u: any) => ({ ...u, hourlyRate: u.rate || u.hourlyRate })));
```
Dev mode: normalizes orders (assignee + defaulted `laborHours`) and employees (`hourlyRate`).

```tsx
          // load dev worklogs + any saved local dev_worklogs
          const stash = JSON.parse(localStorage.getItem('dev_worklogs') || '[]');
          const combined = (dev.devWorklogs || []).concat(stash || []);
          // Filter: admin sees all, employee sees only their own
          const filtered = user!.role === 'admin' ? combined : combined.filter((w:any) => w.artisanId === user!.id);
          setWorklogs(filtered);
          return;
        }
```
Combines seed mock worklogs with any previously user-added ones stored in `localStorage`; scopes them by role (admin sees all, employee sees only their own entries).

```tsx
        const [o, e, w] = await Promise.all([orderService.list(), employeeService.list(), worklogService.list()]);
        setOrders(o);
        setEmployees(e);
        setWorklogs(user!.role === 'admin' ? w : w.filter((x:any)=> x.artisanId === user!.id));
      } catch (err) {
        console.error(err);
      }
    })();
  }, [user]);
```
Production mode: fetches all three lists in parallel, applies the same role-based scoping. Effect depends on `user` (re-runs if the logged-in user changes, e.g., after login/logout without a full page reload).

```tsx
  const isAdmin = user?.role === "admin";
```
Derived boolean flag (uses optional chaining, unlike the `user!.role` non-null assertions used above/below — a minor inconsistency in null-safety style within the same file).

```tsx
  const visible = isAdmin ? orders.filter(o => o.assignedEmployeeId) : orders.filter(o => o.assignedEmployeeId === user!.id);
```
Admin sees all orders that **have** an assignee at all (filters out unassigned ones); employees see only orders assigned specifically to them. (Note: `visible` itself is computed but not directly rendered as a table in this file — it seems to exist to support the "Add Entry" dialog's order dropdown, filtered again there.)

```tsx
  const myWorklogs = isAdmin ? worklogs : worklogs.filter(w => w.artisanId === user!.id);
```
Since `worklogs` state is already role-scoped from the fetch above, this re-applies the same filter — redundant for employees (double-filtering to the same result) but harmless.

```tsx
  const filtered = myWorklogs.filter(w => {
    const order = orders.find(o => o.id === w.orderId);
    if (searchOrderName && order && !(order.title || '').toLowerCase().includes(searchOrderName.toLowerCase())) return false;
```
For each worklog: finds its parent order; if a search term is present and the order exists and its title doesn't match (case-insensitive substring), exclude it.

```tsx
    if (monthFilter && monthFilter !== 'All') {
      const m = new Date(w.date).toLocaleString('default', { month: 'long' });
      if (m !== monthFilter) return false;
    }
```
If a specific month is selected (not "All"), computes the worklog's full month name (e.g., "June") and excludes it if it doesn't match.

```tsx
    if (dateFrom && new Date(w.date) < new Date(dateFrom)) return false;
    if (dateTo && new Date(w.date) > new Date(dateTo)) return false;
    return true;
  });
```
Additional date-range filtering: excludes entries before `dateFrom` or after `dateTo` (if those filters are set). If nothing excluded it, the entry passes (`return true`).

```tsx
  const totalHours = filtered.reduce((s, w) => s + (w.hours || 0), 0);
  const totalWages = filtered.reduce((s, w) => s + (w.wages || 0), 0);
```
Sums hours/wages across the currently-filtered set (so totals update live as filters change).

```tsx
  // breakdown by order
  const byOrder: any[] = Array.from(filtered.reduce((map, w) => {
    const o = orders.find(x => x.id === w.orderId) || { id: w.orderId, title: w.orderId };
    const key = w.orderId;
    if (!map.has(key)) map.set(key, { orderId: key, orderName: o.title, totalHours: 0, totalWages: 0, count: 0 });
    const cur = map.get(key);
    cur.totalHours += w.hours || 0;
    cur.totalWages += w.wages || 0;
    cur.count += 1;
    return map;
  }, new Map<string, any>()).values());
```
Builds a per-order summary: reduces the filtered worklogs into a `Map` keyed by `orderId`, initializing each entry with the order's title (falling back to using the order ID itself as a display title if the order record can't be found) and zeroed totals, then accumulates hours/wages/count for each worklog belonging to that order. Finally converts the map's values to a plain array via `Array.from(...)`.

```tsx
  return (
    <>
      <PageHeader kicker="WORK RECORDS" title={isAdmin ? "Team work log." : "My worklog."} />
```
Title text adapts by role.

```tsx
      <div className="mb-3">
        <div className="flex flex-wrap items-center gap-3 mb-2">
          <Input placeholder="Search by order name" value={searchOrderName} onChange={e => setSearchOrderName(e.target.value)} className="w-60" />
          <Select value={monthFilter} onValueChange={v => setMonthFilter(v)}>
            <SelectTrigger className="w-44 h-8"><SelectValue /></SelectTrigger>
            <SelectContent>
              <SelectItem value="All">All</SelectItem>
              <SelectItem value="May">May</SelectItem>
              <SelectItem value="June">June</SelectItem>
            </SelectContent>
          </Select>
```
Search box; month dropdown hardcoded to only offer "All"/"May"/"June" (rather than being dynamically derived from the actual data's date range — a limitation if worklogs exist in other months).

```tsx
          <div className="flex items-center gap-2">
            <Label className="text-xs">From</Label>
            <Input type="date" value={dateFrom} onChange={e => setDateFrom(e.target.value)} />
            <Label className="text-xs">To</Label>
            <Input type="date" value={dateTo} onChange={e => setDateTo(e.target.value)} />
            <Button variant="outline" onClick={() => { /* filters applied live */ }}>Search</Button>
          </div>
        </div>
```
Native `<input type="date">` pickers for date-range filtering; "Search" button is a no-op (filtering already happens live via the `filtered` computation above) — same pattern as `Orders.tsx`.

```tsx
        <div className="py-2 px-3 rounded-t-md bg-[#3d2b1f]">
          <div className="text-white font-bold uppercase">{isAdmin ? 'ALL WORKLOG ENTRIES' : 'MY WORKLOG ENTRIES'}</div>
        </div>
      </div>
```
Section title bar, role-adaptive.

```tsx
      <div className="rounded-sm border border-border bg-gradient-card overflow-hidden">
        <table className="w-full text-sm">
          <thead>
            <tr className="bg-[#3d2b1f]">
              <th className="px-5 py-3 text-white font-bold uppercase">LOG ID</th>
              {isAdmin && <th className="px-5 py-3 text-white font-bold uppercase">ARTISAN</th>}
              <th className="px-5 py-3 text-white font-bold uppercase">ORDER ID</th>
              <th className="px-5 py-3 text-white font-bold uppercase">ORDER NAME</th>
              <th className="px-5 py-3 text-white font-bold uppercase">TYPE</th>
              <th className="px-5 py-3 text-white font-bold uppercase">HOURS WORKED</th>
              <th className="px-5 py-3 text-white font-bold uppercase">WAGES EARNED</th>
              <th className="px-5 py-3 text-white font-bold uppercase">DATE</th>
              {!isAdmin && <th className="px-5 py-3 text-white font-bold uppercase">NOTE</th>}
            </tr>
          </thead>
```
Dynamic column set: admins get an extra "Artisan" column (since they see everyone's entries) but lose the "Note" column; employees see their own note but not an artisan name (since it's always themselves).

```tsx
          <tbody>
            {filtered.map((w, idx) => (
              <tr key={w.id} className={`border-t border-border ${idx%2===0 ? 'bg-white' : 'bg-[#fff7ef]'} text-[#2c1a0e]`}>
                <td className="px-5 py-3">{w.id}</td>
                {isAdmin && <td className="px-5 py-3">{w.artisanName}</td>}
                <td className="px-5 py-3">{w.orderId}</td>
                <td className="px-5 py-3">{(orders.find(o => o.id === w.orderId)?.title) || w.orderId}</td>
                <td className="px-5 py-3">{orders.find(o => o.id === w.orderId)?.type || '—'}</td>
                <td className="px-5 py-3">{(w.hours||0).toFixed(2)} hrs</td>
                <td className="px-5 py-3 text-[#b8860b] font-bold">${(w.wages||0).toFixed(2)}</td>
                <td className="px-5 py-3">{w.date}</td>
                {!isAdmin && <td className="px-5 py-3">{w.note}</td>}
              </tr>
            ))}
          </tbody>
        </table>
```
Row rendering matches the conditional column set above; order name/type are looked up live from the `orders` list by ID each render (rather than being denormalized onto the worklog record itself).

```tsx
        <div className="p-4 bg-[#fff7ef] border-t border-border">
          <div className="grid grid-cols-3 gap-4">
            <div>TOTAL ENTRIES<br /><strong>{filtered.length} logs</strong></div>
            <div>TOTAL HOURS LOGGED<br /><strong>{totalHours.toFixed(2)} hrs</strong></div>
            <div>TOTAL WAGES EARNED<br /><strong className="text-[#b8860b]">${totalWages.toFixed(2)}</strong></div>
          </div>
        </div>
      </div>
```
A summary footer row beneath the table: entry count, total hours, total wages (computed above from the currently-filtered set).

```tsx
      <div className="mt-6">
        <div className="py-2 px-3 rounded-t-md bg-[#3d2b1f]">
          <div className="text-white font-bold uppercase">WORKLOG BY ORDER</div>
        </div>
        <div className="rounded-sm border border-border bg-gradient-card overflow-hidden">
          <table className="w-full text-sm">
            <thead>
              <tr className="bg-[#3d2b1f]">
                <th className="px-5 py-3 text-white font-bold uppercase">ORDER ID</th>
                <th className="px-5 py-3 text-white font-bold uppercase">ORDER NAME</th>
                <th className="px-5 py-3 text-white font-bold uppercase">TOTAL HOURS</th>
                <th className="px-5 py-3 text-white font-bold uppercase">TOTAL WAGES</th>
                <th className="px-5 py-3 text-white font-bold uppercase">ENTRIES COUNT</th>
              </tr>
            </thead>
            <tbody>
              {byOrder.map((b, idx) => (
                <tr key={b.orderId} className={`border-t border-border ${idx%2===0 ? 'bg-white' : 'bg-[#fff7ef]'} text-[#2c1a0e]`}>
                  <td className="px-5 py-3">{b.orderId}</td>
                  <td className="px-5 py-3">{b.orderName}</td>
                  <td className="px-5 py-3">{b.totalHours.toFixed(2)} hrs</td>
                  <td className="px-5 py-3 text-[#b8860b] font-bold">${b.totalWages.toFixed(2)}</td>
                  <td className="px-5 py-3">{b.count} entries</td>
                </tr>
              ))}
            </tbody>
          </table>
        </div>
      </div>
```
A second table rendering the `byOrder` per-order breakdown computed earlier.

```tsx
      {/* Add Entry Modal */}
      {adding && (
        <Dialog open={adding} onOpenChange={setAdding}>
          <DialogContent className="max-w-md">
            <DialogHeader><DialogTitle className="font-display text-lg bg-[#3d2b1f] text-white p-2">Add Worklog Entry</DialogTitle></DialogHeader>
```
Modal only mounted at all when `adding` is true (double-guarded: both the `{adding && ...}` wrapper and the Dialog's own `open` prop).

```tsx
            <div className="p-4">
              <div className="space-y-3">
                <div>
                  <Label>Order</Label>
                  <Select value={newEntry.orderId} onValueChange={v => setNewEntry(prev => ({ ...prev, orderId: v }))}>
                    <SelectTrigger className="w-full h-9"><SelectValue /></SelectTrigger>
                    <SelectContent>
                      {orders.filter(o => o.assignedEmployeeId === user!.id).map(o => <SelectItem key={o.id} value={o.id}>{o.id} — {o.title}</SelectItem>)}
                    </SelectContent>
                  </Select>
                </div>
```
Order dropdown, options limited to orders assigned to the **current logged-in user** specifically (regardless of admin status — this dialog is really an employee self-service form even though it's defined in a component both roles can view).

```tsx
                <div>
                  <Label>Order Name</Label>
                  <Input value={orders.find(o=>o.id===newEntry.orderId)?.title || ''} readOnly className="mt-1" />
                </div>
```
A **read-only** auto-populated field showing the selected order's title (derived, not independently editable) — purely a confirmation display.

```tsx
                <div className="grid grid-cols-2 gap-3">
                  <div>
                    <Label>Date</Label>
                    <Input type="date" value={newEntry.date} onChange={e => setNewEntry(prev => ({ ...prev, date: e.target.value }))} />
                  </div>
                  <div>
                    <Label>Hours Worked</Label>
                    <Input type="number" step="0.25" value={newEntry.hours} onChange={e => setNewEntry(prev => ({ ...prev, hours: parseFloat(e.target.value) || 0 }))} />
                  </div>
                </div>
```
Two-column row: date picker, and hours input allowing quarter-hour increments (`step="0.25"`).

```tsx
                <div>
                  <Label>Wages (auto)</Label>
                  <div className="mt-1 text-[#b8860b] font-bold">${(newEntry.hours * 10).toFixed(2)}</div>
                </div>
```
Live-computed wage preview — **hardcoded to a $10/hr rate** rather than using the logged-in employee's actual `hourlyRate` — a bug/placeholder, since it ignores the real rate used elsewhere (e.g. `EmployeeOverview.tsx` uses `user!.hourlyRate`).

```tsx
                <div>
                  <Label>Note</Label>
                  <textarea title="Note" placeholder="Optional note" className="w-full border p-2" value={newEntry.note} onChange={e => setNewEntry(prev => ({ ...prev, note: e.target.value }))} />
                </div>
```
A raw native `<textarea>` (not the shared `Textarea` UI component used elsewhere) for an optional note.

```tsx
                <div className="flex justify-end gap-2">
                  <Button variant="outline" onClick={() => setAdding(false)}>Cancel</Button>
                  <Button variant="gold" onClick={async () => {
                    // validation
                    if (!newEntry.orderId) return toast.error('Order must be selected');
                    if (!(newEntry.hours > 0)) return toast.error('Hours must be > 0');
                    if (new Date(newEntry.date) > new Date()) return toast.error('Date cannot be in future');
```
Cancel just closes the dialog. Submit validates: an order must be chosen, hours must be a positive number, and the date can't be in the future.

```tsx
                    const id = `WL-${Math.floor(Math.random()*900)+100}`;
                    const wages = parseFloat((newEntry.hours * 10).toFixed(2));
                    const entry = { id, artisanId: user!.id, artisanName: user!.name, orderId: newEntry.orderId, hours: newEntry.hours, wages, date: newEntry.date, note: newEntry.note };
```
Generates a pseudo-random log ID (same `WL-XXX` pattern as elsewhere); computes wages again at the hardcoded `$10/hr` rate; builds the full entry object stamped with the current user's ID/name.

```tsx
                    try {
                      if (import.meta.env.DEV) {
                        const stash = JSON.parse(localStorage.getItem('dev_worklogs') || '[]'); stash.push(entry); localStorage.setItem('dev_worklogs', JSON.stringify(stash));
                        setWorklogs(prev => [entry, ...prev]);
                        setOrders(prev => prev.map(o => o.id === newEntry.orderId ? { ...o, laborHours: (o.laborHours || 0) + newEntry.hours } : o));
                        toast.success('Entry saved');
                        setAdding(false);
                        return;
                      }
```
Dev mode: persists the new entry into `localStorage` (so it survives a page reload, unlike other dev-mode mutations in this app which only live in memory), prepends it to local `worklogs` state, and **also bumps the related order's `laborHours`** by the new entry's hours (keeping the order's cumulative labor total in sync). Shows success and closes.

```tsx
                      await worklogService.create(entry);
                      setWorklogs(prev => [entry, ...prev]);
                      setOrders(prev => prev.map(o => o.id === newEntry.orderId ? { ...o, laborHours: (o.laborHours || 0) + newEntry.hours } : o));
                      toast.success('Entry saved');
                      setAdding(false);
                    } catch (e) { toast.error('Failed to save'); }
                  }}>Submit</Button>
                </div>
              </div>
            </div>
          </DialogContent>
        </Dialog>
      )}
    </>
  );
}
```
Production mode: same local-state and order-hours updates, but persists via the real `worklogService.create()` API call instead of `localStorage`.

---

### 16. `AdminOverview.tsx`

```tsx
import { useEffect, useMemo, useState } from "react";
import { PageHeader } from "@/components/layout/AppShell";
import { StatCard, StatusBadge } from "@/components/ui/stat-card";
import { AlertTriangle, ClipboardList, DollarSign, Hammer, Package, Receipt, ShieldCheck, Users } from "lucide-react";
import * as orderService from "@/services/orderService";
import * as inventoryService from "@/services/inventoryService";
import * as billingService from "@/services/billingService";
import * as customerService from "@/services/customerService";
import * as employeeService from "@/services/employeeService";
import dev from "@/lib/devData";
import { Bar, BarChart, CartesianGrid, Cell, Line, LineChart, ResponsiveContainer, Tooltip, XAxis, YAxis } from "recharts";
import { Link } from "react-router-dom";
```
Imports a `StatCard` component (a KPI tile), 8 different icons (one per stat card), all 5 backend services, dev data, chart components (note: `Cell`, `Legend`-less/`Pie`-less subset since only Bar/Line charts appear here), and `Link` for internal navigation.

```tsx
type MonthPoint = { month: string; revenue: number };

type StatusPoint = { status: string; count: number; fill: string };
```
Two small TypeScript interfaces describing the shape of chart data points, used for type-safety in the `useMemo` calls below.

```tsx
const STATUS_ORDER = ["Pending", "In Progress", "Completed", "Delivered", "Rejected"];
const STATUS_COLORS: Record<string, string> = {
  Pending: "#9ca3af",
  "In Progress": "#c9a227",
  Completed: "#b45309",
  Delivered: "#1e40af",
  Rejected: "#dc2626",
};
```
Fixed display order and a hex-color lookup for each possible order status, so the bar chart always shows statuses in a consistent order/color regardless of what's present in the data.

```tsx
export default function AdminOverview() {
  const [orders, setOrders] = useState<any[]>([]);
  const [materials, setMaterials] = useState<any[]>([]);
  const [payments, setPayments] = useState<any[]>([]);
  const [customers, setCustomers] = useState<any[]>([]);
  const [employees, setEmployees] = useState<any[]>([]);
```
Five data lists — this dashboard aggregates data from every domain in the app.

```tsx
  useEffect(() => {
    (async () => {
      if (import.meta.env.DEV) {
        const mapMaterials = (arr: any[]) => arr.map(m => ({ id: m.id, name: m.name, supplier: m.supplier, quantity: parseFloat(String(m.qty || m.quantity || "0").replace(/[^0-9.]/g, "")) || 0, threshold: parseFloat(String(m.minThreshold || m.threshold || "0").replace(/[^0-9.]/g, "")) || 0, price: Number(m.unitPrice || m.price || 0), type: m.type }));
        const mapEmployees = (arr: any[]) => arr.map(u => ({ ...u, hourlyRate: u.rate || u.hourlyRate }));
        const mapOrders = (arr: any[]) => arr.map(o => ({ ...o, assignedEmployeeId: o.employeeId || o.assignedEmployeeId, laborHours: o.laborHours || o.laborHours || 0 }));
```
Same normalizer pattern seen throughout the app, defined as local named functions this time (rather than inline). Note `laborHours: o.laborHours || o.laborHours || 0` is a redundant self-OR (likely a copy-paste artifact — functionally identical to `o.laborHours || 0`).

```tsx
        setOrders(mapOrders(dev.devOrders));
        setMaterials(mapMaterials(dev.devMaterials));
        setPayments(dev.devPayments);
        setCustomers(dev.devCustomers);
        setEmployees(mapEmployees(dev.devEmployees));
        return;
      }
      const [o, m, p, c, e] = await Promise.all([
        orderService.list(),
        inventoryService.list(),
        billingService.list(),
        customerService.list(),
        employeeService.list(),
      ]);
      setOrders(o);
      setMaterials(m);
      setPayments(p);
      setCustomers(c);
      setEmployees(e);
    })();
  }, []);
```
Applies normalizers in dev mode; fetches all 5 resources in parallel in production. Runs once on mount.

```tsx
  const revenue = useMemo(
    () => payments.reduce((sum, p) => sum + Number(p.amount || 0), 0),
    [payments]
  );
```
Total revenue = sum of all payment amounts, memoized on `payments`.

```tsx
  const totalLabourCost = useMemo(
    () => orders.reduce((sum, o) => sum + Number(o.laborHours || 0) * 25, 0),
    [orders]
  );
```
Total labor cost estimate: sums each order's hours **times a flat $25/hr** (not each employee's actual individual rate — a simplification/approximation, since this figure aggregates across potentially many different employees with different rates).

```tsx
  const lowStock = useMemo(
    () => materials.filter((m) => Number(m.quantity || 0) <= Number(m.threshold || 0)),
    [materials]
  );
```
Materials at or below their reorder threshold.

```tsx
  const pendingWorklogs = useMemo(
    () => orders.filter((o) => Number(o.laborHours || 0) > 0 && o.status !== "Completed" && o.status !== "Delivered").length,
    [orders]
  );
```
Counts orders that have **some** logged hours but aren't yet finished/delivered — used as a rough proxy for "work in progress that still needs attention."

```tsx
  const monthlyRevenue = useMemo<MonthPoint[]>(() => {
    const map = new Map<number, MonthPoint>();
    payments.forEach((p) => {
      const d = new Date(p.createdAt || p.date || Date.now());
      const monthIndex = d.getMonth();
      if (!map.has(monthIndex)) {
        map.set(monthIndex, {
          month: d.toLocaleString("en", { month: "short" }),
          revenue: 0,
        });
      }
      const curr = map.get(monthIndex);
      if (curr) curr.revenue += Number(p.amount || 0);
    });
    return Array.from(map.entries())
      .sort((a, b) => a[0] - b[0])
      .map(([, v]) => v);
  }, [payments]);
```
Builds a month-by-month revenue series, keyed this time by **numeric month index (0–11)** rather than the month-name string used in `Reports.tsx` — this naturally handles chronological sorting without needing a separate `monthOrder` lookup array (sorts entries by their numeric key `a[0] - b[0]`, then discards the key and keeps just the value object via destructuring `([, v]) => v`).

```tsx
  const orderStatusData = useMemo<StatusPoint[]>(() => {
    const counts = new Map<string, number>();
    orders.forEach((o) => {
      const k = String(o.status || "Pending");
      counts.set(k, (counts.get(k) || 0) + 1);
    });
    return STATUS_ORDER.map((status) => ({
      status,
      count: counts.get(status) || 0,
      fill: STATUS_COLORS[status],
    }));
  }, [orders]);
```
Counts orders per status (defaulting missing statuses to "Pending"), then maps over the **fixed** `STATUS_ORDER` array (not the counted keys) to produce a consistently-ordered, consistently-colored dataset — statuses with zero orders still appear in the chart with `count: 0`.

```tsx
  const recentOrders = useMemo(() => orders.slice(0, 5), [orders]);
  const recentPayments = useMemo(() => payments.slice(0, 5), [payments]);
  const recentlyAssigned = useMemo(
    () => orders.filter((o) => o.assignedEmployeeId).slice(0, 5),
    [orders]
  );
```
Three "recent activity" lists, each capped at 5 items. **Note:** none of these actually sort by date/recency first — they just take the first 5 items of whatever order the underlying array/API returned them in, so "recent" is really "however the source data happens to be ordered," not guaranteed to reflect true chronological recency.

```tsx
  return (
          <div>
            <PageHeader kicker="ATELIER OVERVIEW" title="Good craft, today." />
```
No `<>` fragment here — wraps everything in an actual `<div>` (unlike most other page components in this app which use fragments).

```tsx
            <div className="grid grid-cols-1 md:grid-cols-2 xl:grid-cols-4 gap-4 mb-10">
              <StatCard label="Total Orders" value={orders.length} hint="All orders" icon={<Package className="h-5 w-5" />} accent />
              <StatCard label="Total Customers" value={customers.length} hint="Approved customers" icon={<Users className="h-5 w-5" />} />
              <StatCard label="Total Employees" value={employees.length} hint="Approved artisans" icon={<Hammer className="h-5 w-5" />} />
              <StatCard label="Total Revenue" value={<span className="value-amber">${revenue.toLocaleString()}</span>} hint="Paid amount" icon={<Receipt className="h-5 w-5" />} />
              <StatCard label="Total Labour Cost" value={`$${totalLabourCost.toLocaleString()}`} hint="Estimated from logged hours" icon={<DollarSign className="h-5 w-5" />} />
              <StatCard label="Low Stock Alerts" value={lowStock.length} hint="Materials below threshold" icon={<AlertTriangle className="h-5 w-5" />} />
              <StatCard label="Pending Worklogs" value={pendingWorklogs} hint="Hours logged on open orders" icon={<ClipboardList className="h-5 w-5" />} />
              <StatCard label="Pending Approvals" value={0} hint="User access requests" icon={<ShieldCheck className="h-5 w-5" />} />
            </div>
```
A responsive grid of 8 KPI tiles (1 col mobile → 2 col medium → 4 col extra-large, so it wraps into two rows of 4 on wide screens). Each `StatCard` gets a label, value (some are raw numbers, one is formatted JSX with a `$` and locale-formatted thousands separators via `.toLocaleString()`), a descriptive hint, and an icon. The first card has an `accent` prop (likely gives it special/highlighted styling). **Note:** "Pending Approvals" is hardcoded to `0` rather than being wired to the actual pending-signup-request count (which `UserAccess.tsx` tracks) — a disconnect between this overview and the real data.

```tsx
            <div className="grid lg:grid-cols-3 gap-4 mb-10">
              <div className="lg:col-span-2 rounded-sm border border-border bg-gradient-card p-6 ring-inset-light">
                <div className="flex items-baseline justify-between mb-6">
                  <div>
                    <div className="text-[10px] uppercase tracking-[0.3em] text-muted-foreground">Revenue</div>
                    <div className="font-display text-2xl mt-1 text-[#3d2b1f] italic">📈 Monthly flow</div>
                    <div className="mt-3 h-px w-24 bg-gradient-to-r from-transparent via-[#b8860b] to-transparent" />
                  </div>
                  <div className="text-xs text-muted-foreground">Last activity</div>
                </div>
```
A 3-column grid where the revenue chart card spans 2 columns (`lg:col-span-2`). Card header includes an emoji (📈) directly in the JSX text, an italic styled subtitle, and a decorative gradient divider line.

```tsx
                <div className="h-64">
                  <ResponsiveContainer>
                    <LineChart data={monthlyRevenue}>
                      <CartesianGrid stroke="hsl(var(--border))" strokeDasharray="3 3" vertical={false} />
                      <XAxis dataKey="month" stroke="hsl(var(--muted-foreground))" fontSize={11} tickLine={false} axisLine={false} />
                      <YAxis stroke="hsl(var(--muted-foreground))" fontSize={11} tickLine={false} axisLine={false} />
                      <Tooltip contentStyle={{ background: "hsl(var(--popover))", border: "1px solid hsl(var(--border))", borderRadius: 4 }} />
                      <Line type="monotone" dataKey="revenue" stroke="hsl(var(--gold))" strokeWidth={2.5} dot={{ fill: "hsl(var(--gold))", r: 4 }} />
                    </LineChart>
                  </ResponsiveContainer>
                </div>
              </div>
```
A single-line revenue-over-time chart (smoothed/`monotone` curve), gold-colored, with rounded tooltip styling.

```tsx
              <div className="rounded-sm border border-border bg-gradient-card p-6 ring-inset-light">
                <div className="text-[10px] uppercase tracking-[0.3em] text-muted-foreground">Pipeline</div>
                <div className="font-display text-2xl mt-1 text-[#3d2b1f] italic">📊 Order status</div>
                <div className="mt-3 h-px w-24 bg-gradient-to-r from-transparent via-[#b8860b] to-transparent" />
                
                <div className="h-64">
                  <ResponsiveContainer>
                    <BarChart data={orderStatusData}>
                      <CartesianGrid stroke="hsl(var(--border))" strokeDasharray="3 3" vertical={false} />
                      <XAxis dataKey="status" stroke="hsl(var(--muted-foreground))" fontSize={10} tickLine={false} axisLine={false} />
                      <YAxis stroke="hsl(var(--muted-foreground))" fontSize={11} tickLine={false} axisLine={false} />
                      <Tooltip contentStyle={{ background: "hsl(var(--popover))", border: "1px solid hsl(var(--border))", borderRadius: 4 }} />
                      <Bar dataKey="count" fill="hsl(var(--gold))" radius={[2, 2, 0, 0]} />
                    </BarChart>
                  </ResponsiveContainer>
                </div>
              </div>
            </div>
```
The remaining 1-column slot holds an order-status bar chart. **Note:** all bars use a single flat gold `fill` here, meaning the per-status `fill` colors computed in `orderStatusData` (`STATUS_COLORS`) are actually **not applied** — this chart doesn't use `<Cell>` components to individually color each bar (unlike the Reports.tsx pie chart, which does), so the `fill` field on each data point goes unused. This looks like an intended-but-not-fully-wired feature.

```tsx
            <div className="grid lg:grid-cols-[320px_minmax(0,1fr)] gap-4 mb-4 items-start">
              <div className="rounded-sm border border-border bg-gradient-card overflow-hidden ring-inset-light">
                <div className="font-display text-2xl mt-1 text-[#3d2b1f] italic"> ⚠️ Low Stock Alerts</div>
                <div className="mt-3 h-px w-24 bg-gradient-to-r from-transparent via-[#b8860b] to-transparent" />
                <div className="p-8">
```
A custom two-column CSS grid: a fixed 320px sidebar and a flexible remaining column (`minmax(0,1fr)`), for the Low Stock panel and Recent Activity panel respectively.

```tsx
                  {lowStock.length === 10 ? (
                    <div className="text-sm text-[#5c4033] py-4 text-center">All materials well stocked.</div>
                  ) : (
```
**Bug:** this checks `lowStock.length === 10` to decide whether to show the "well stocked" empty-state message — almost certainly should be `=== 0`. As written, the empty-state message will only ever appear if there happen to be exactly 10 low-stock materials (an essentially arbitrary/wrong condition), and the actual "zero low-stock items" case will incorrectly fall through to the list-rendering branch (which would just render an empty `.map()`, showing nothing rather than a friendly message).

```tsx
                    <div className="space-y-50">
                      {lowStock.map((m) => (
                        <div key={m.id} className="flex items-center justify-between gap-3 px-50 py-2 border-b border-border last:border-0">
```
**Likely bug:** `space-y-50` and `px-50` are not standard Tailwind spacing scale values (Tailwind's default scale tops out around `96`; `50` isn't a defined utility unless a custom config defines it) — these classes probably don't apply any actual spacing and were likely meant to be smaller numbers like `space-y-5`/`px-5`, causing items to render without the intended gap/padding.

```tsx
                          <div className="min-w-0">
                            <div className="font-medium text-[#2c1a0e] text-sm truncate">{m.name}</div>
                            <div className="text-[11px] text-[#8b6f5e] truncate">{m.supplier || "No supplier"}</div>
                          </div>
                          <div className="text-right shrink-0">
                            <div className="low-stock-value text-sm">Stock: {m.quantity}</div>
                            <div className="text-[10px] uppercase tracking-widest text-[#8b6f5e]">Min: {m.threshold}</div>
                          </div>
                        </div>
                      ))}
                      <div className="pt-2">
                        <Link to="/app/inventory" className="text-xs font-medium text-[#b8860b] hover:underline">View Inventory</Link>
                      </div>
                    </div>
                  )}
                </div>
              </div>
```
Each low-stock row: material name/supplier (truncated if too long, `min-w-0` needed alongside `truncate` for flex children), current stock and minimum threshold on the right, with `border-b ... last:border-0` giving dividers between items except after the last one. A link at the bottom navigates to the full Inventory page.

```tsx
              <div className="rounded-sm border border-border bg-gradient-card overflow-hidden ring-inset-light">
                <div className="font-display text-2xl mt-1 text-[#3d2b1f] italic">  💰 Recent Activity</div>
                <div className="mt-3 h-px w-24 bg-gradient-to-r from-transparent via-[#b8860b] to-transparent" />
                <div className="p-6 space-y-7">
                  <div>
                    <h3 className="inline-flex items-center rounded-sm bg-[#3d2b1f] px-4 py-2 text-xs font-semibold uppercase tracking-[0.18em] text-white shadow-sm mb-3">Recent Orders</h3>
                    <div className="space-y-2">
                      {recentOrders.length === 0 ? <div className="text-sm text-[#5c4033]">No orders yet.</div> : recentOrders.map((o) => (
                        <div key={o.id} className="flex items-center justify-between border-b border-border pb-2">
                          <div>
                            <div className="font-medium text-[#2c1a0e]">{o.title}</div>
                            <div className="text-xs text-[#8b6f5e]">{o.type}</div>
                          </div>
                          <StatusBadge status={o.status || "Pending"} />
                        </div>
                      ))}
                    </div>
                  </div>
```
"Recent Orders" sub-panel: title label styled as a small dark pill, then either an empty-state message or up to 5 order rows (title + type on the left, colored status badge on the right).

```tsx
                  <div>
                    <h3 className="inline-flex items-center rounded-sm bg-[#3d2b1f] px-4 py-2 text-xs font-semibold uppercase tracking-[0.18em] text-white shadow-sm mb-3">Recent Payments</h3>
                    <div className="space-y-2">
                      {recentPayments.length === 0 ? <div className="text-sm text-[#5c4033]">No payments yet.</div> : recentPayments.map((p) => (
                        <div key={p.id} className="flex items-center justify-between border-b border-border pb-2">
                          <div className="text-sm text-[#2c1a0e]">Payment #{p.id}</div>
                          <div className="text-sm value-amber">${Number(p.amount || 0).toFixed(2)}</div>
                        </div>
                      ))}
                    </div>
                  </div>
```
"Recent Payments" sub-panel: same pattern, showing a payment ID and amount.

```tsx
                  <div>
                    <h3 className="inline-flex items-center rounded-sm bg-[#3d2b1f] px-4 py-2 text-xs font-semibold uppercase tracking-[0.18em] text-white shadow-sm mb-3">Recently Assigned Tasks</h3>
                    <div className="space-y-2">
                      {recentlyAssigned.length === 0 ? <div className="text-sm text-[#5c4033]">No assignments yet.</div> : recentlyAssigned.map((o) => (
                        <div key={o.id} className="flex items-center justify-between border-b border-border pb-2">
                          <div>
                            <div className="text-sm text-[#2c1a0e]">{o.title}</div>
                            <div className="text-xs text-[#8b6f5e]">Employee ID: {o.assignedEmployeeId}</div>
                          </div>
                          <div className="text-xs text-[#8b6f5e]">{new Date(o.createdAt || Date.now()).toLocaleDateString()}</div>
                        </div>
                      ))}
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
  );
}
```
"Recently Assigned Tasks" sub-panel: order title and the raw employee **ID** (not resolved to a display name — a UX gap, since every other panel shows human-readable names), plus a formatted assignment/creation date.

---

### 17. `CustomerOverview.tsx`

```tsx
import { PageHeader } from "@/components/layout/AppShell";
import { StatusBadge } from "@/components/ui/stat-card";
import { Button } from "@/components/ui/button";
import { useAuth } from "@/lib/auth";
import * as orderService from "@/services/orderService";
import * as billingService from "@/services/billingService";
import dev from "@/lib/devData";
import { useState, useEffect } from "react";
import { Dialog, DialogContent, DialogHeader, DialogTitle, DialogTrigger, DialogFooter } from "@/components/ui/dialog";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import { Textarea } from "@/components/ui/textarea";
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "@/components/ui/select";
import { Plus } from "lucide-react";
import { toast } from "sonner";
import { Order } from "@/lib/store";
```
Standard imports; only needs `orderService` and `billingService` (customers don't touch inventory/employee services directly).

```tsx
export default function CustomerOverview() {
  const { user } = useAuth();
  const [tick, setTick] = useState(0);
  const [open, setOpen] = useState(false);
  const [form, setForm] = useState({ title: "", description: "", type: "Furniture" as Order["type"] });
  const refresh = () => setTick(t => t + 1);
```
`open`/`form` support the "New Commission" (order request) dialog.

```tsx
  const [myOrders, setMyOrders] = useState<any[]>([]);
  const [myPayments, setMyPayments] = useState<any[]>([]);
  const [myInvoices, setMyInvoices] = useState<any[]>([]);
```
Three lists scoped specifically to this customer.

```tsx
  useEffect(() => {
    (async () => {
      try {
        if (import.meta.env.DEV) {
          const orders = dev.devOrders.filter(o => o.customerId === user!.id);
          const payments = dev.devPayments.filter(p => p.customerId === user!.id);
          const invoices = dev.devInvoices.filter(i => i.customerId === user!.id);
          setMyOrders(orders);
          setMyPayments(payments);
          setMyInvoices(invoices);
          return;
        }
```
Dev mode: filters the mock global lists down to just this customer's records by `customerId`.

```tsx
        const [o, p, inv] = await Promise.all([orderService.list(), billingService.list(), billingService.list()]);
        setMyOrders(o.filter((x: any) => x.customerId === user!.id));
        setMyPayments(p.filter((x: any) => x.customerId === user!.id));
        setMyInvoices(inv.filter((x: any) => x.customerId === user!.id));
      } catch (err) { console.error(err); }
    })();
  }, [tick, user]);
```
**Note:** `billingService.list()` is called **twice** (once for `p`, once for `inv`) — likely meant to be two different service calls (e.g., `billingService.listPayments()` and `billingService.listInvoices()`), but as written it fetches the same data twice and filters it into two separately-named-but-identically-sourced state variables (`myPayments` and `myInvoices` would end up holding the same underlying records, just independently filtered) — an inefficiency/likely bug in the production code path (dev mode correctly separates `dev.devPayments` vs `dev.devInvoices`).

```tsx
  const submit = async () => {
    if (!form.title.trim()) return toast.error("Title required");
    try {
      if (import.meta.env.DEV) {
        const newOrder = { id: `ORD-${Math.floor(Math.random()*900)+100}`, title: form.title, description: form.description, type: form.type, customerId: user!.id, customer: user!.name, employeeId: null, employee: null, status: 'Pending', orderDate: new Date().toISOString().slice(0,10), estimatedCost: 0, finalCost: null, materials: [], laborHours: 0, assignedDate: null };
```
Validates a non-blank (trimmed) title. In dev mode, builds a complete new order object with a random `ORD-XXX` ID, defaulting status to `'Pending'`, no assigned employee yet, today's date, zeroed costs, empty materials, zero labor hours.

```tsx
        const orders = JSON.parse(localStorage.getItem('dev_orders') || '[]');
        orders.push(newOrder);
        localStorage.setItem('dev_orders', JSON.stringify(orders));
        setMyOrders(prev => [newOrder, ...prev]);
        toast.success('Commission requested');
        setForm({ title: "", description: "", type: "Furniture" });
        setOpen(false);
        return;
      }
```
Persists the new order into a **separate** `localStorage` key (`dev_orders`) — note this is distinct from the main `dev.devOrders` in-memory array, meaning this new order is **not** visible to other pages' dev-mode data loading (e.g., `Orders.tsx`, `AdminOverview.tsx`) unless they specifically also read from `dev_orders`, which — based on the code shown — they don't. This customer-created order effectively only shows up on this customer's own overview page's local state list, not app-wide, in dev mode. Also prepends it to local `myOrders` state for immediate display, resets the form, closes the dialog.

```tsx
      await orderService.create({ customerId: user!.id, title: form.title, description: form.description, type: form.type, materials: [] });
      toast.success("Commission requested");
      setForm({ title: "", description: "", type: "Furniture" });
      setOpen(false); refresh();
    } catch { toast.error("Failed to request commission"); }
  };
```
Production mode: calls the real create API, then refetches.

```tsx
  const totalBilled = myInvoices.reduce((s, i) => s + (i.totalAmount || 0), 0);
  const totalPaid = myInvoices.reduce((s, i) => s + (i.paidAmount || 0), 0);
  const totalBalance = totalBilled - totalPaid;
```
Computes billing totals (though `totalBalance` is computed but, notably, **not displayed anywhere** in the JSX below — dead calculation).

```tsx
  const statCards = [
    { label: 'Total My Orders', value: myOrders.length, detail: 'All orders placed by ' + user!.name.split(' ')[0] },
    { label: 'Pending Orders', value: myOrders.filter(o => o.status === 'Pending').length, detail: 'Waiting for admin approval' },
    { label: 'In Progress Orders', value: myOrders.filter(o => o.status === 'In Progress').length, detail: 'Currently being worked on' },
    { label: 'Completed / Delivered', value: myOrders.filter(o => o.status === 'Completed' || o.status === 'Delivered').length, detail: 'Successfully finished orders' },
  ];
```
A declarative array describing the 4 stat cards to render, each with a personalized detail line (using the customer's first name in the first card).

```tsx
  return (
    <>
      <PageHeader kicker={`WELCOME, ${user!.name.split(" ")[0].toUpperCase()}`} title="Your commissions."
        action={
          <Dialog open={open} onOpenChange={setOpen}>
            <DialogTrigger asChild><Button variant="gold"><Plus className="h-4 w-4" /> New Commission</Button></DialogTrigger>
```
Personalized greeting kicker using the customer's uppercased first name; header action slot contains the "New Commission" dialog trigger (always visible — no admin-only gating needed here, since this whole page is customer-only).

```tsx
            <DialogContent>
              <DialogHeader><DialogTitle className="font-display text-2xl">Request a commission</DialogTitle></DialogHeader>
              <div className="space-y-4 py-2">
                <div><Label>Commission Title</Label><Input value={form.title} onChange={e => setForm({ ...form, title: e.target.value })} placeholder="Walnut dining table" /></div>
                <div><Label>Category</Label>
                  <Select value={form.type} onValueChange={v => setForm({ ...form, type: v as Order["type"] })}>
                    <SelectTrigger><SelectValue /></SelectTrigger>
                    <SelectContent>
                      <SelectItem value="Furniture">Furniture</SelectItem>
                      <SelectItem value="Repair">Repair</SelectItem>
                      <SelectItem value="Custom Design">Custom Design</SelectItem>
                      <SelectItem value="Other">Other</SelectItem>
                    </SelectContent>
                  </Select>
                </div>
                <div><Label>Description / Notes</Label><Textarea value={form.description} onChange={e => setForm({ ...form, description: e.target.value })} placeholder="Full details of your requirement..." rows={4} /></div>
              </div>
              <DialogFooter><Button variant="gold" onClick={submit}>Submit request</Button></DialogFooter>
            </DialogContent>
          </Dialog>
        } />
```
Form fields: title (text with example placeholder), category (dropdown with 4 fixed options, cast to `Order["type"]` for type-safety on selection), description (4-row textarea). Submit button calls `submit`.

```tsx
      {/* Stats Cards Row */}
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4 mb-8">
        {statCards.map((card, idx) => (
          <div key={idx} className="rounded-sm border border-border bg-gradient-card p-6">
            <div className="text-[10px] uppercase tracking-[0.3em] text-[#5c4033] mb-2 font-semibold">{card.label}</div>
            <div className="font-display text-4xl text-[#2c1a0e]">{card.value}</div>
            <div className="text-sm text-muted-foreground mt-1">{card.detail}</div>
          </div>
        ))}
      </div>
```
Renders the 4 stat cards declared earlier as a responsive grid (using array index as key, acceptable here since the array is static and never reordered/filtered).

```tsx
      {/* Commission Cards Grid */}
      {myOrders.length === 0 ? (
        <div className="border border-dashed border-border rounded-sm p-16 text-center">
          <div className="font-display text-2xl mb-2">No commissions yet</div>
          <div className="text-sm text-muted-foreground">Begin your first piece above.</div>
        </div>
      ) : (
        <>
          <div className="grid md:grid-cols-2 gap-4 mb-10">
            {myOrders.map(o => {
              const emp = dev.devEmployees.find(e => e.id === o.employeeId);
```
Empty-state message when there are no orders; otherwise a 2-column grid of commission cards. For each order, looks up the assigned employee **directly from the dev mock data** (`dev.devEmployees`) even outside a dev-mode branch — meaning in production this lookup would silently fail to find real employees (since `dev.devEmployees` would just be static mock data unrelated to the real backend's employee records) — a bug/leftover-from-development pattern.

```tsx
              const statusKey = o.status.toUpperCase();
              return (
                <div key={o.id} className="rounded-sm border border-border bg-gradient-card p-6 ring-inset-light">
                  <div className="flex items-start justify-between mb-3">
                    <div>
                      <div className="text-[10px] uppercase tracking-[0.3em] text-gold mb-2 font-bold">{o.type}</div>
                      <div className="font-display text-xl font-bold text-[#2c1a0e]">{o.title}</div>
                    </div>
```
Order type label and title.

```tsx
                  <div className={`text-xs font-bold px-2 py-1 rounded ${
                    statusKey === 'IN PROGRESS' ? 'bg-yellow-100 text-yellow-900' :
                    statusKey === 'PENDING' ? 'bg-gray-100 text-gray-700' :
                    statusKey === 'COMPLETED' ? 'bg-green-100 text-green-700' :
                    statusKey === 'DELIVERED' ? 'bg-blue-100 text-blue-700' :
                    'bg-gray-100 text-gray-700'
                  }`}>
                    {statusKey}
                  </div>
                  </div>
```
A manually color-coded status pill (rather than reusing the shared `StatusBadge` component used elsewhere in this same file for the tables below) — yellow for in-progress, gray for pending, green for completed, blue for delivered, gray fallback for anything else.

```tsx
                  <p className="text-sm text-muted-foreground mb-5 line-clamp-2">{o.description || "—"}</p>
                  <div className="gold-divider mb-4" />
```
Description truncated to 2 lines via `line-clamp-2`.

```tsx
                  <div className="grid grid-cols-3 gap-3 text-sm">
                    <div><div className="text-[10px] uppercase tracking-widest text-muted-foreground font-semibold">TOTAL</div><div className="font-display text-[#b8860b]">${o.finalCost || o.estimatedCost || 0}</div></div>
                    <div><div className="text-[10px] uppercase tracking-widest text-muted-foreground font-semibold">HOURS</div><div className="font-display text-[#2c1a0e]">{o.laborHours}</div></div>
                    <div><div className="text-[10px] uppercase tracking-widest text-muted-foreground font-semibold">ARTISAN</div><div className="text-xs text-[#2c1a0e]">{emp?.name || "—"}</div></div>
                  </div>
                </div>
              );
            })}
          </div>
```
3-column footer: cost (prefers final over estimate, then `0`), labor hours, and the assigned artisan's name (or em-dash if not yet assigned / not found).

```tsx
          {/* Recent Activity Sections */}
          <div className="space-y-6">
            <div>
              <div className="py-2 px-3 rounded-t-md bg-[#3d2b1f]">
                <div className="text-white font-bold uppercase">RECENT ORDERS</div>
              </div>
              <div className="rounded-sm border border-border bg-gradient-card overflow-hidden">
                <table className="w-full text-sm">
                  <thead>
                    <tr className="bg-[#3d2b1f]">
                      <th className="px-5 py-3 text-white font-bold uppercase">ORDER ID</th>
                      <th className="px-5 py-3 text-white font-bold uppercase">ORDER NAME</th>
                      <th className="px-5 py-3 text-white font-bold uppercase">TYPE</th>
                      <th className="px-5 py-3 text-white font-bold uppercase">STATUS</th>
                    </tr>
                  </thead>
                  <tbody>
                    {myOrders.slice(0, 3).map((o, idx) => (
                      <tr key={o.id} className={`border-t border-border ${idx%2===0 ? 'bg-white' : 'bg-[#fff7ef]'} text-[#2c1a0e]`}>
                        <td className="px-5 py-3 font-medium">{o.id}</td>
                        <td className="px-5 py-3">{o.title}</td>
                        <td className="px-5 py-3">{o.type}</td>
                        <td className="px-5 py-3"><StatusBadge status={o.status} /></td>
                      </tr>
                    ))}
                  </tbody>
                </table>
              </div>
            </div>
```
A "Recent Orders" mini-table showing just the first 3 orders (this time correctly using the shared `StatusBadge` component for status coloring).

```tsx
            <div>
              <div className="py-2 px-3 rounded-t-md bg-[#3d2b1f]">
                <div className="text-white font-bold uppercase">RECENT PAYMENTS</div>
              </div>
              <div className="rounded-sm border border-border bg-gradient-card overflow-hidden">
                <table className="w-full text-sm">
                  <thead>
                    <tr className="bg-[#3d2b1f]">
                      <th className="px-5 py-3 text-white font-bold uppercase">AMOUNT</th>
                      <th className="px-5 py-3 text-white font-bold uppercase">DATE</th>
                      <th className="px-5 py-3 text-white font-bold uppercase">METHOD</th>
                      <th className="px-5 py-3 text-white font-bold uppercase">STATUS</th>
                    </tr>
                  </thead>
                  <tbody>
                    {myPayments.slice(0, 3).map((p, idx) => (
                      <tr key={p.id} className={`border-t border-border ${idx%2===0 ? 'bg-white' : 'bg-[#fff7ef]'} text-[#2c1a0e]`}>
                        <td className="px-5 py-3 text-[#b8860b] font-bold">${p.amount.toFixed(2)}</td>
                        <td className="px-5 py-3">{p.date || '—'}</td>
                        <td className="px-5 py-3">{p.method || '—'}</td>
                        <td className="px-5 py-3"><StatusBadge status={p.status} /></td>
                      </tr>
                    ))}
                  </tbody>
                </table>
              </div>
            </div>
```
"Recent Payments" mini-table, first 3 payments, formatted amount with fixed 2 decimals (note: `p.amount.toFixed(2)` here — unlike other places in this file/app which defensively wrap in `Number(x || 0)` first — would throw if `amount` were ever `undefined`/`null`, since `.toFixed` doesn't exist on `undefined`).

```tsx
            <div>
              <div className="py-2 px-3 rounded-t-md bg-[#3d2b1f]">
                <div className="text-white font-bold uppercase">PENDING INVOICES</div>
              </div>
              <div className="rounded-sm border border-border bg-gradient-card overflow-hidden">
                <table className="w-full text-sm">
                  <thead>
                    <tr className="bg-[#3d2b1f]">
                      <th className="px-5 py-3 text-white font-bold uppercase">INVOICE ID</th>
                      <th className="px-5 py-3 text-white font-bold uppercase">ORDER</th>
                      <th className="px-5 py-3 text-white font-bold uppercase">BALANCE DUE</th>
                    </tr>
                  </thead>
                  <tbody>
                    {myInvoices.filter(i => i.balance > 0).slice(0, 3).map((inv, idx) => (
                      <tr key={inv.id} className={`border-t border-border ${idx%2===0 ? 'bg-white' : 'bg-[#fff7ef]'} text-[#2c1a0e]`}>
                        <td className="px-5 py-3 font-medium">{inv.invoiceNumber}</td>
                        <td className="px-5 py-3">{inv.orderId}</td>
                        <td className="px-5 py-3 text-[#b8860b] font-bold">${inv.balance.toFixed(2)}</td>
                      </tr>
                    ))}
                  </tbody>
                </table>
              </div>
            </div>
          </div>
        </>
      )}
    </>
  );
}
```
"Pending Invoices" mini-table: filters to invoices with an outstanding balance (`balance > 0`), takes up to 3, showing invoice number, related order, and balance due.

---

### 18. `EmployeeOverview.tsx`

```tsx
import { PageHeader } from "@/components/layout/AppShell";
import { StatusBadge } from "@/components/ui/stat-card";
import { Button } from "@/components/ui/button";
import { useAuth } from "@/lib/auth";
import * as orderService from "@/services/orderService";
import { OrderStatus } from "@/lib/store";
import { useState, useEffect } from "react";
import { Input } from "@/components/ui/input";
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "@/components/ui/select";
import { toast } from "sonner";
import { Clock } from "lucide-react";
import dev from "@/lib/devData";
```
Several imports (`Button`, `OrderStatus`, `Input`, `Select*`, `toast`, `Clock`) are pulled in but — based on the JSX below — largely **unused** in this component's actual render output, suggesting this file was copy/adapted from a more feature-rich page (like `Orders.tsx`) and trimmed down without cleaning up unused imports.

```tsx
export default function EmployeeOverview() {
  const { user } = useAuth();
  const [tick, setTick] = useState(0);
  const refresh = () => setTick(t => t + 1);
```
`refresh`/`tick` defined but, notably, **never actually called anywhere** in this file (no button triggers `refresh()`) — effectively dead, since data only loads once on mount via the effect below.

```tsx
  const [myOrders, setMyOrders] = useState<any[]>([]);
  const [myWorklogs, setMyWorklogs] = useState<any[]>([]);
  useEffect(() => {
    (async () => {
      if (import.meta.env.DEV) {
        const orders = dev.devOrders.filter(o => o.employeeId === user!.id || o.employeeId === (user!.id) || o.employeeId === (user!.id) || o.employeeId === (user!.id) || o.employeeId === 'E-02' || o.employeeId === undefined && o.employeeId);
```
This line computes `orders` but the result is **never used** — the very next line re-filters and overwrites via `setMyOrders(...)` directly, ignoring this `orders` variable entirely. As written, the condition itself is also redundant/buggy: `o.employeeId === (user!.id)` is repeated four times (functionally identical each time — parentheses don't change anything), plus a hardcoded fallback to the specific demo ID `'E-02'`, plus a nonsensical trailing clause `o.employeeId === undefined && o.employeeId` (this can never be true — if `employeeId` is `undefined`, then `undefined && undefined` short-circuits to `undefined`, which is falsy, and if it's *not* undefined, the whole expression is `false` because the first comparison fails). This whole line is essentially dead/broken code kept from earlier debugging.

```tsx
        setMyOrders(dev.devOrders.filter(o => o.employeeId === user!.id || o.employeeId === 'E-02' || o.employeeId === o.employeeId));
```
The line that actually sets state: filters for orders where `employeeId` matches the current user's ID, **or** equals the hardcoded demo employee ID `'E-02'` (meaning in dev mode, the page will always show at least the demo employee "E-02"'s orders regardless of who's actually logged in — a testing convenience/leftover), **or** the always-true tautology `o.employeeId === o.employeeId` (any value trivially equals itself, including `undefined === undefined`) — which means this final clause makes the **entire filter always match every order**, effectively defeating the point of filtering at all. In practice, `myOrders` ends up containing **every** order in the dev dataset, not just this employee's.

```tsx
        setMyWorklogs(dev.devWorklogs.filter(w => w.artisanId === user!.id || w.artisanId === 'E-02'));
        return;
      }
      const o = await orderService.list(); setMyOrders(o.filter((x: any) => x.assignedEmployeeId === user!.id));
    })();
  }, [tick]);
```
Worklogs filter is more sensible (though still includes the hardcoded `'E-02'` fallback). Production mode correctly filters by `assignedEmployeeId === user!.id` only (no demo-ID leakage, no tautology bug) — meaning this file behaves very differently in dev vs. prod, with the dev-mode bugs likely being invisible until a real backend is connected.

```tsx
  const totalHours = myOrders.reduce((s, o) => s + (o.laborHours || 0), 0);
  const wages = totalHours * (user!.hourlyRate ?? 0);
```
Sums labor hours across (the buggy, overly-broad in dev mode) `myOrders`, multiplies by the current user's real hourly rate (defaulting to `0` if not set via nullish coalescing).

```tsx
  return (
    <>
      <PageHeader kicker={`WELCOME, ${user!.name.split(' ')[0].toUpperCase()}`} title="Your workspace." />
```
Personalized greeting, same pattern as `CustomerOverview.tsx`.

```tsx
      <div className="grid grid-cols-1 md:grid-cols-3 gap-4 mb-10">
        <div className="rounded-sm border border-border bg-gradient-card p-6">
          <div className="text-[10px] uppercase tracking-[0.3em] text-[#5c4033] mb-2 font-semibold">MY ASSIGNED ORDERS</div>
          <div className="font-display text-4xl">{myOrders.length}</div>
          <div className="text-sm text-muted-foreground mt-1">Total tasks assigned by admin</div>
        </div>
```
**Note:** the wrapping grid is declared `md:grid-cols-3`, but there are actually **6** stat cards below (not 3) — meaning on medium screens it'll wrap into 2 rows of 3, which is probably fine visually, though it doesn't scale further on extra-large screens (no `xl:grid-cols-6` like `AdminOverview.tsx` has) — a minor responsive-design inconsistency compared to the admin dashboard.

```tsx
        <div className="rounded-sm border border-border bg-gradient-card p-6">
          <div className="text-[10px] uppercase tracking-[0.3em] text-[#5c4033] mb-2 font-semibold">PENDING TASKS</div>
          <div className="font-display text-4xl">{myOrders.filter(o => o.status === 'Pending').length}</div>
          <div className="text-sm text-muted-foreground mt-1">Not yet started</div>
        </div>
        <div className="rounded-sm border border-border bg-gradient-card p-6">
          <div className="text-[10px] uppercase tracking-[0.3em] text-[#5c4033] mb-2 font-semibold">IN PROGRESS TASKS</div>
          <div className="font-display text-4xl">{myOrders.filter(o => o.status === 'In Progress').length}</div>
          <div className="text-sm text-muted-foreground mt-1">Currently working on</div>
        </div>
        <div className="rounded-sm border border-border bg-gradient-card p-6">
          <div className="text-[10px] uppercase tracking-[0.3em] text-[#5c4033] mb-2 font-semibold">COMPLETED TASKS</div>
          <div className="font-display text-4xl">{myOrders.filter(o => o.status === 'Completed').length}</div>
          <div className="text-sm text-muted-foreground mt-1">Successfully finished</div>
        </div>
```
Three status-count cards (Pending / In Progress / Completed), each a simple `.filter().length` on `myOrders`.

```tsx
        <div className="rounded-sm border border-border bg-gradient-card p-6">
          <div className="text-[10px] uppercase tracking-[0.3em] text-[#5c4033] mb-2 font-semibold">TOTAL HOURS THIS MONTH</div>
          <div className="font-display text-4xl">{totalHours.toFixed(2)} hrs</div>
          <div className="text-sm text-muted-foreground mt-1">Sum of logged hours June 2026</div>
        </div>
```
Hours card — label says "June 2026" hardcoded in the description, rather than dynamically reflecting the actual current month, and `totalHours` itself is computed from **all** of `myOrders` (not filtered to the current month specifically) — so the number shown isn't really "this month's hours," just total hours across whatever orders ended up in `myOrders`.

```tsx
        <div className="rounded-sm border border-border bg-gradient-card p-6">
          <div className="text-[10px] uppercase tracking-[0.3em] text-[#5c4033] mb-2 font-semibold">TOTAL WAGES THIS MONTH</div>
          <div className="font-display text-4xl value-amber">${wages.toFixed(0)}</div>
          <div className="text-sm text-muted-foreground mt-1">Hours × Hourly Rate</div>
        </div>
      </div>
```
Wages card, same "this month" framing caveat as above; description explicitly documents the calculation (`Hours × Hourly Rate`) for the user's benefit.

```tsx
      <div className="space-y-6">
        <div>
          <div className="py-2 px-3 rounded-t-md bg-[#3d2b1f]">
            <div className="text-white font-bold uppercase">RECENT ASSIGNED TASKS</div>
          </div>
          <div className="rounded-sm border border-border bg-gradient-card overflow-hidden">
            <table className="w-full text-sm">
              <thead>
                <tr className="bg-[#3d2b1f]">
                  <th className="px-5 py-3 text-white font-bold uppercase">ORDER ID</th>
                  <th className="px-5 py-3 text-white font-bold uppercase">ORDER NAME</th>
                  <th className="px-5 py-3 text-white font-bold uppercase">ORDER TYPE</th>
                  <th className="px-5 py-3 text-white font-bold uppercase">CUSTOMER</th>
                  <th className="px-5 py-3 text-white font-bold uppercase">STATUS</th>
                </tr>
              </thead>
              <tbody>
                {myOrders.map((o, idx) => (
                  <tr key={o.id} className={`border-t border-border ${idx%2===0 ? 'bg-white' : 'bg-[#fff7ef]'} text-[#2c1a0e]`}>
                    <td className="px-5 py-3 font-medium text-[#2c1a0e]">{o.id}</td>
                    <td className="px-5 py-3 text-[#2c1a0e]">{o.title}</td>
                    <td className="px-5 py-3 text-[#2c1a0e]">{o.type}</td>
                    <td className="px-5 py-3 text-[#2c1a0e]">{o.customer}</td>
                    <td className="px-5 py-3"><StatusBadge status={o.status} /></td>
                  </tr>
                ))}
              </tbody>
            </table>
          </div>
        </div>
```
"Recent Assigned Tasks" table — despite the "Recent" label, this actually renders **every** item in `myOrders` (no `.slice()` limiting it, unlike similarly-named "recent" sections elsewhere in the app), which — combined with the dev-mode filter bug discussed above — could show a large, unintended number of rows in development.

```tsx
        <div>
          <div className="py-2 px-3 rounded-t-md bg-[#3d2b1f]">
            <div className="text-white font-bold uppercase">RECENT WORKLOG ENTRIES</div>
          </div>
          <div className="rounded-sm border border-border bg-gradient-card overflow-hidden">
            <table className="w-full text-sm">
              <thead>
                <tr className="bg-[#3d2b1f]">
                  <th className="px-5 py-3 text-white font-bold uppercase">LOG ID</th>
                  <th className="px-5 py-3 text-white font-bold uppercase">ORDER ID</th>
                  <th className="px-5 py-3 text-white font-bold uppercase">ORDER NAME</th>
                  <th className="px-5 py-3 text-white font-bold uppercase">HOURS</th>
                  <th className="px-5 py-3 text-white font-bold uppercase">WAGES</th>
                  <th className="px-5 py-3 text-white font-bold uppercase">DATE</th>
                </tr>
              </thead>
              <tbody>
                {myWorklogs.map((w, idx) => (
                  <tr key={w.id} className={`border-t border-border ${idx%2===0 ? 'bg-white' : 'bg-[#fff7ef]'} text-[#2c1a0e]`}>
                    <td className="px-5 py-3 font-medium text-[#2c1a0e]">{w.id}</td>
                    <td className="px-5 py-3 text-[#2c1a0e]">{w.orderId}</td>
                    <td className="px-5 py-3 text-[#2c1a0e]">{dev.devOrders.find(o=>o.id===w.orderId)?.title || w.orderId}</td>
                    <td className="px-5 py-3 text-[#2c1a0e]">{w.hours.toFixed(2)} hrs</td>
                    <td className="px-5 py-3 text-[#b8860b] font-bold">${w.wages.toFixed(2)}</td>
                    <td className="px-5 py-3 text-[#2c1a0e]">{w.date}</td>
                  </tr>
                ))}
              </tbody>
            </table>
          </div>
        </div>
```
"Recent Worklog Entries" table, also showing **all** of `myWorklogs` (no slicing). The order name is resolved directly from `dev.devOrders` again (same production-mode issue as `CustomerOverview.tsx` — this lookup would not correctly resolve real order titles once connected to a live backend, since it always reads from the static mock array regardless of environment).

```tsx
        <div>
          <div className="py-2 px-3 rounded-t-md bg-[#3d2b1f]">
            <div className="text-white font-bold uppercase">THIS MONTH WAGE SUMMARY</div>
          </div>
          <div className="rounded-sm border border-border bg-gradient-card overflow-hidden">
            <table className="w-full text-sm">
              <thead>
                <tr className="bg-[#3d2b1f]">
                  <th className="px-5 py-3 text-white font-bold uppercase">Month</th>
                  <th className="px-5 py-3 text-white font-bold uppercase">Total Hours Logged</th>
                  <th className="px-5 py-3 text-white font-bold uppercase">Hourly Rate</th>
                  <th className="px-5 py-3 text-white font-bold uppercase">Total Wages Earned</th>
                  <th className="px-5 py-3 text-white font-bold uppercase">Payment Status</th>
                </tr>
              </thead>
              <tbody>
                <tr className="border-t border-border bg-white text-[#2c1a0e]">
                  <td className="px-5 py-3">June 2026</td>
                  <td className="px-5 py-3">{totalHours.toFixed(2)} hrs</td>
                  <td className="px-5 py-3">${user!.hourlyRate}/hr</td>
                  <td className="px-5 py-3 text-[#b8860b] font-bold">${wages.toFixed(2)}</td>
                  <td className="px-5 py-3"><span className="inline-block px-2 py-1 rounded text-sm bg-yellow-100 text-yellow-800">Pending</span></td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </>
  );
}
```
A single-row summary table: hardcoded "June 2026" label (same caveat as the stat card above), computed hours/wages, the employee's actual hourly rate, and a hardcoded yellow "Pending" payment-status pill (always shown as pending regardless of actual payment state — consistent with `WageSummary.tsx`'s similarly hardcoded demo data).

---

## Part 3 — Recurring Patterns Across the Codebase

1. **Dev/Prod dual data path**: nearly every page checks `import.meta.env.DEV` and either reads from `@/lib/devData` (in-memory mock arrays) or calls the matching `*Service.list()` API function. This lets the whole UI be built and demoed before a backend exists.
2. **Field-name normalization**: mock data uses inconsistent field names (`qty` vs `quantity`, `rate` vs `hourlyRate`, `employeeId` vs `assignedEmployeeId`). Nearly every data-loading `useEffect` includes a small "mapper" function reconciling these.
3. **`tick` + `refresh()` re-fetch pattern**: a dummy counter in `useState`, incremented to force a `useEffect` to re-run and reload data after a mutation.
4. **Toast-based feedback**: every async action (`submit`, `remove`, status update) wraps in `try/catch`, calling `toast.success`/`toast.error` from `sonner`.
5. **Role-based rendering**: `useAuth()`'s `user.role` (`"admin" | "customer" | "employee"`) gates which UI elements, columns, and actions are shown.
6. **Zebra-striped tables**: virtually every data table alternates row background color via `idx % 2 === 0 ? 'bg-white' : 'bg-[#fff7ef]'`.
7. **Simulated (non-persisted) actions**: several handlers (profile saves, password changes) only show a success toast without an actual API call — clearly marked with `// TODO` or `// in DEV we just toast` comments.

## Part 4 — Notable Issues Spotted During Review

| File | Issue |
|---|---|
| `Customers.tsx` | `spend` is computed but never rendered. |
| `Employees.tsx` | `removeEmp` and the add-employee dialog markup appear unused/unwired in the visible JSX. |
| `Orders.tsx` | Cancel-confirmation dialog body requires both `cancelConfirm` **and** `selectedOrder` to be set, but the Cancel button only sets `cancelConfirm`. Dev-mode order cancellation doesn't remove the order from the underlying `dev.devOrders` mock array. |
| `Reports.tsx` | Dev-mode employee-role filter has confusing operator precedence. |
| `WageSummary.tsx` | "Total Earned" stat is hardcoded (`$610`) instead of computed from `totalWages`. Imports `dev` but never uses it. |
| `Worklog.tsx` | New worklog entries always compute wages at a hardcoded `$10/hr`, ignoring the employee's real hourly rate. |
| `AdminOverview.tsx` | `lowStock.length === 10` should almost certainly be `=== 0`. `space-y-50` / `px-50` are not valid Tailwind spacing classes. "Pending Approvals" stat is hardcoded to `0`. Bar chart doesn't actually apply the computed per-status `fill` colors. |
| `CustomerOverview.tsx` | Fetches `billingService.list()` twice for what look like two different resources (payments vs. invoices). New orders created in dev mode are stored in a separate, disconnected `localStorage` key. Employee lookup uses `dev.devEmployees` even in the production code path. |
| `EmployeeOverview.tsx` | Dev-mode order filter contains a tautology (`o.employeeId === o.employeeId`) that makes the filter match every order, plus dead/duplicated code. Hardcoded fallback to demo employee ID `'E-02'`. "This month" labels don't actually filter by month. |
| `NotFound.tsx` | Uses a plain `<a href="/">` instead of the router's `<Link>`, causing a full page reload. |

---

*Generated from the 18 available `.tsx` files. Re-upload `Billing.tsx` and `Index.tsx` for those to be added to this document.*

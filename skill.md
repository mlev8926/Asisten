# Skill Specification: 10-Day Interactive Medical Spacing Scheduler

**Version:** 5.1.0 Production (Multi-Patient + Structured Confirmation)  
**Target AI Agents:** Cursor, VS Code AI, Claude CLI, Claude Code, Windsurf, Universal AI Assistants  
**Status:** Production-Ready | Multi-Patient + Medicine Confirmation Checklist  
**Last Updated:** 2026-05-25

---

## 1. Context & Agent Persona

You are an **Expert Frontend Engineer** and **Medical Safety Compliance Agent**. Your responsibilities:

- **Primary Duty:** Maintain strict adherence to medical safety protocols to prevent double-dosing and bypass vulnerabilities across **multiple patients**.
- **Code Quality:** Ensure single-file architecture with zero external dependencies (except CDN imports).
- **Testing:** Validate all changes against the state machine transitions and mathematical engine before deployment.
- **Compliance:** Every modification must pass both functional and safety audits.

When modifying or extending `index.html`, you must ensure that **under no circumstances** can a user:
- Double-dose medication
- Bypass medical safety locks
- Access restricted states (TERLEWAT, BELUM_WAKTUNYA) with interactive controls
- Trigger unvalidated state transitions
- **Access another patient's data without authorization** (v5.0)
- **Distribute medications without structured confirmation** (NEW - v5.1)

---

## 2. Core Constraints (Strict System Prompts - The 5 Golden Rules + 1)

Every AI agent editing the codebase must **strictly adhere** to these immutable constraints:

### 2.1 Single-File Architecture
- The **entire application** must reside within a single `index.html` file.
- No external CSS files. No external JS files.
- Bootstrap 5.3.2 and Bootstrap Icons must be imported via CDN `<link>` tags only.
- All styles must be inline in `<style>` tags.
- All JavaScript must be inline in `<script>` tags.

**Rationale:** Ensures portability, eliminates dependency hell, and simplifies deployment.

### 2.2 Asia/Jakarta (WIB) Timezone Isolation
- All system time references, date calculations, and display updates must be **locked to WIB timezone**.
- Never use UTC or local client system timezone if they deviate from WIB.
- Use `timeZone: 'Asia/Jakarta'` in all `toLocaleTimeString()` and `toLocaleDateString()` calls.
- All real-time clock updates must reflect WIB time with explicit " WIB" suffix in UI.

**Rationale:** Medical compliance requires absolute time consistency across all deployments.

### 2.3 Strict Read-Only Enforcement
- Cards or table cells in state **TERLEWAT** (Auto-Skip) or **BELUM_WAKTUNYA** (Not Yet Time) must be **read-only**.
- Clicking them must trigger `showWarningToast()` and **immediately deny execution**.
- No state mutations allowed for locked states.
- Visual indicators: `cursor: not-allowed`, muted colors, lock icons.

**Rationale:** Prevents accidental state changes and enforces medical safety boundaries.

### 2.4 No Native Modals/Alerts
- **Never** use `alert()`, `confirm()`, or `prompt()` in production code paths.
- Exception: Inside CRUD management modal (medicines AND patients), `confirm()` and `prompt()` are allowed **exclusively** for:
  - Medicine definition deletion
  - Patient creation/deletion workflows
  - Definition modification workflows
  - User intent confirmation for CRUD operations
- Use Bootstrap modals (`bootstrap.Modal`) for all user confirmations outside CRUD.

**Rationale:** Native dialogs interrupt UX flow; Bootstrap modals provide consistent, themeable UI.

### 2.5 Indonesian Language Constraint
- All UI text, status badges, toast notifications, and user-facing strings must remain in **professional, clear Indonesian**.
- Never include English UI labels in production.
- Maintain consistent terminology:
  - "Siklus Hari" (Medical Cycle/Day)
  - "DIMINUM" (Taken)
  - "AKTIF" (Active/Due)
  - "BELUM_WAKTUNYA" (Not Yet Time)
  - "TERLEWAT" (Missed/Auto-Skip)
  - "Nomor RM" (Medical Record Number)
  - "Data Pasien" (Patient Data)

**Rationale:** End-users are Indonesian speakers; localization ensures medical accuracy and user trust.

### 2.6 Multi-Patient Data Isolation (v5.0)
- Each patient's data must be **completely isolated** in localStorage with unique namespace keys.
- Patient selection is **persistent** (stored in localStorage).
- Switching patients triggers full UI re-render with correct patient's data.
- **NO patient data leakage** between sessions or patient contexts.
- Patient ID is generated once on creation and **immutable**.

**Rationale:** Clinical data privacy; HIPAA-like compliance; prevents cross-contamination of medical records.

### 2.7 Structured Medicine Confirmation (NEW - v5.1)
- Before marking any slot as `DIMINUM`, user must confirm status of **each medicine individually**.
- Confirmation modal shows checklist: **Diberikan (✓) / Tidak Diberikan (✗) / Distop (⊘)**.
- Dropdown per medicine to track distribution status.
- Status persisted in `medicineStatusDatabase` (namespaced per patient).
- Prevents accidental bulk verification without clinical oversight.

**Rationale:** Ensures granular control over medication distribution, enables audit trail per medicine.

---

## 3. Multi-Patient Architecture (v5.0 + v5.1 Updates)

### 3.1 Patient Registry System

**Patient Object Schema:**
```javascript
{
  id: "P-2026-0000001",           // Immutable unique identifier
  name: "Budi Santoso",            // Patient name
  rm: "RM-2026000001",             // Display RM number (RM-YYYY???????)
  createdAt: "2026-05-24T10:30:00+07:00",  // WIB timestamp
  lastModified: "2026-05-24T10:30:00+07:00"
}
```

### 3.2 Patient-Scoped Data Namespace

**All patient data is stored with prefix:** `patient_{patientId}_{dataType}`

```javascript
{
  // Global
  'patients_registry': [ { id, name, rm, createdAt, lastModified } ],
  'active_patient_id': 'P-2026-0000001',
  
  // Per-Patient (NEW v5.1: medicineStatus)
  'patient_P-2026-0000001_meds': {...},
  'patient_P-2026-0000001_taken': {...},
  'patient_P-2026-0000001_vitals': [...],
  'patient_P-2026-0000001_medicineStatus': {...},  // NEW v5.1
  'patient_P-2026-0000001_startDate': 'YYYY-MM-DD'
}
```

**Medicine Status Schema (NEW v5.1):**
```javascript
{
  '0-06:35_med_1': 'diberikan',     // dayIndex-slot_med_medId: status
  '0-06:35_med_2': 'tidak',
  '0-06:35_med_3': 'distop',
  // Statuses: 'diberikan' | 'tidak' | 'distop'
}
```

---

## 4. Mathematical Spacing Engine & Scheduling Logic

### 4.1 Medical 24-Hour Cycle Anchor & Day Rotation (NEW v5.1)

**One "Medical Day" (Siklus Hari)** is defined as a 24-hour period:
- **Start:** 06:35 AM WIB (on the anchor date)
- **End:** 06:35 AM WIB of the subsequent calendar day
- **Anchor Point:** `06:35` (immutable)

**NEW - Day Rotation at 00:00:01 (NOT 02:00):**
- When real-time clock hits **00:00:01 AM WIB** of calendar date d+1:
  - System's `activeDayIndex` **automatically transitions** to d+1
  - UI switches to display next medical cycle
  - Status calculations recalculate based on new day

**Dini Hari (Early Morning) Offset (FIXED in v5.1):**
- Slot `00:35` (00:35 WIB, midnight-plus-35-minutes) is chronologically on calendar date d+1
- **BUT logically stays in Day d** (previous medical period)
- **NOT auto-advanced** to Day d+1 even though calendar shows next day
- Example: Slot at 00:35 on May 24 → still belongs to "Hari 1" which started May 23 at 06:35

**Example Timeline (v5.1):**
```
Hari 1 (Medical Day 1):
  - Medical Start: 2026-05-23 06:35 WIB
  - Medical End:   2026-05-24 06:35 WIB
  - Slot 06:35 → 2026-05-23 06:35 WIB (Hari 1)
  - Slot 00:35 → 2026-05-24 00:35 WIB (STILL Hari 1, calendar is 24th)
  - Day Rotation: When clock hits 00:00:01 on 2026-05-24 → Active switches to Hari 2

Hari 2 (Medical Day 2):
  - Medical Start: 2026-05-24 06:35 WIB
  - Medical End:   2026-05-25 06:35 WIB
```

### 4.2 Frequency & Interval Spacing

| Frequency | Times/Day | Interval | Example Times |
|-----------|-----------|----------|---|
| `1x1` | 1 | 24 hours | 20:00 |
| `2x1` | 2 | 12 hours | 06:35, 18:35 |
| `3x1` | 3 | 8 hours | 06:35, 14:35, 22:35 |
| `4x1` | 4 | 6 hours | 06:35, 12:35, 18:35, 00:35 |

### 4.3 Expiry Gate & Tolerance Window

- **Target Time:** `T` (e.g., 06:35)
- **Expiry Deadline:** `T + 30 minutes` (e.g., 07:05)
- **Status Rules:**
  - If `now < T`: Status = `BELUM_WAKTUNYA` → **READ-ONLY**
  - If `T ≤ now < T+30min`: Status = `AKTIF` → **INTERACTIVE** (requires confirmation)
  - If `now ≥ T+30min` AND not marked taken: Status = `TERLEWAT` → **READ-ONLY**
  - If marked taken with confirmation: Status = `DIMINUM` → **READ-ONLY** with badge

---

## 5. Medicine Confirmation Workflow (NEW v5.1)

### 5.1 Structured Confirmation Modal

**Trigger:** User clicks "Konfirmasi Minum" on an AKTIF slot

**Modal displays:**
1. Slot time (e.g., "06:35")
2. Checklist of medicines for that slot, each with:
   - Checkbox (always checked initially)
   - Medicine name (label)
   - **Dropdown selector** with 3 options:
     - ✓ **Diberikan** (Given)
     - ✗ **Tidak Diberikan** (Not Given)
     - ⊘ **Distop** (Stopped/Discontinued)

### 5.2 Confirmation Submission

**On user click "Verifikasi & Simpan":**
1. Iterate each medicine in pending confirmation
2. Read dropdown value (`med-status-{idx}`) → one of: 'diberikan' | 'tidak' | 'distop'
3. Store in `medicineStatusDatabase` with key: `{dayIndex}-{slot}_med_{medId}`
4. Mark slot as taken in `takenDatabase`: `{dayIndex}-{slot}` = timestamp
5. Save both to patient-scoped localStorage
6. Close modal, re-render UI
7. Show success toast

### 5.3 Data Persistence

**All medicine status changes** are persisted immediately:
```javascript
savePatientData('medicineStatus', medicineStatusDatabase);
savePatientData('taken', takenDatabase);
```

---

## 6. Data Persistence Architecture (Updated for v5.1)

### 6.1 LocalStorage Key Structure

```javascript
{
  // ===== GLOBAL REGISTRIES =====
  'patients_registry': [ { id, name, rm, createdAt, lastModified } ],
  'active_patient_id': 'P-2026-0000001',
  
  // ===== PER-PATIENT NAMESPACED DATA =====
  'patient_P-2026-0000001_meds': [ { id, name, frequency, startHour } ],
  'patient_P-2026-0000001_taken': { 'dayIndex-slot': 'HH:MM WIB timestamp' },
  'patient_P-2026-0000001_vitals': [ { id, timestamp, measurements, notes, editCount } ],
  'patient_P-2026-0000001_medicineStatus': { '{dayIndex}-{slot}_med_{medId}': 'diberikan|tidak|distop' },  // NEW v5.1
  'patient_P-2026-0000001_startDate': 'YYYY-MM-DD'
}
```

---

## 7. UI Rendering Rules

### 7.1 Medicine Confirmation Modal (NEW v5.1)

**Location:** Bootstrap Modal with ID `confirmMedicineModal`

**Header:** Green background (success), title "Konfirmasi Distribusi Obat"

**Body:**
- Slot time display: `<span id="confirm-slot-time">`
- Confirmation card container: `<div id="confirm-medicines-list">`
  - Each item (`.confirmation-item`):
    - Checkbox (for visual UX, optional interaction)
    - Medicine name (label)
    - Dropdown (`.confirmation-dropdown`)

**Footer:**
- "Tutup Tanpa Simpan" button (dismiss without saving)
- "✓ Verifikasi & Simpan" button (save and verify)

### 7.2 Tab 1: Monitor Hari Ini (Today's Active Cycle)

**Action Button (NEW v5.1):**
- Old: Direct toggle button (removed)
- New: `openMedicineConfirmation(dayIndex, slot)` button
- Label: "Konfirmasi Minum" with icon

**Status Display:** Shows confirmed status with medicine count

### 7.3 Tab 2: Master Table (Full 10-Day Schedule)

No changes from v5.0 – displays aggregate status per slot.

### 7.4 Tab 3: Pemantauan TTV Mandiri

No changes – independent of medicine confirmation.

---

## 8. Safety Mechanisms (Updated for v5.1)

### 8.1 Medicine Confirmation Enforcement

**Rule:** Slot can only transition to `DIMINUM` **after** structured confirmation

**Implementation:**
```javascript
function openMedicineConfirmation(dayIndex, slot) {
  // Only if status === 'AKTIF'
  // Render modal with checklist
  // Store pending confirmation in memory
  // Modal.show()
}

function saveMedicineConfirmation() {
  // Only if pendingConfirmation exists
  // Save status for each medicine
  // Mark slot as taken
  // Clear pending confirmation
  // renderUI()
}
```

### 8.2 Read-Only Lock Enforcement

- BELUM_WAKTUNYA: Button disabled, `cursor: not-allowed`
- TERLEWAT: Button hidden
- AKTIF: Button enabled, triggers confirmation modal
- DIMINUM: Display-only, no interactivity

### 8.3 Vitals Entry Protection

- Max 1 edit per entry (unchanged from v5.0)

### 8.4 Patient Data Isolation

- Enforced at load/save (unchanged from v5.0)

---

## 9. Real-Time Clock & Sync (v5.1 Updates)

### 9.1 Clock Update Interval

- Update every **1000ms** (1 second)
- Call `renderUI()` which re-evaluates all slot statuses
- **Active patient context preserved** during updates

### 9.2 Automatic State Transitions

**Critical Transitions (Monitored Every Tick):**
1. `BELUM_WAKTUNYA` → `AKTIF` (when `now >= T`)
2. `AKTIF` → `TERLEWAT` (when `now >= T+30min` AND not marked)
3. **Day rotation:** When `now >= 00:00:01` on day d+1 (NEW v5.1)

---

## 10. Backend Integration (Firebase/SQLite3) - Roadmap v5.2

### 10.1 API Endpoints (Planned)

**Firebase:**
```
/patients/{patientId}/meds
/patients/{patientId}/taken
/patients/{patientId}/vitals
/patients/{patientId}/medicineStatus  // NEW v5.1
/patients/{patientId}/metadata
```

### 10.2 LocalStorage ↔ Backend Sync Strategy

**Current (v5.1):** localStorage only  
**Future (v5.2):** Sync with backend using patient-scoped data

---

## 11. Code Quality & Validation Checklist

Before committing any changes, verify:

- [ ] All time calculations use WIB timezone explicitly
- [ ] No external CSS/JS files added
- [ ] All strings are in Indonesian
- [ ] BELUM_WAKTUNYA and TERLEWAT states are read-only
- [ ] No `alert()` outside CRUD modals
- [ ] State machine transitions match section 4 (updated)
- [ ] localStorage keys match section 6 (patient-scoped + medicineStatus)
- [ ] 30-minute expiry window is enforced
- [ ] **Day rotation at 00:00:01 implemented** (NEW v5.1)
- [ ] **00:35 slot stays in previous medical period** (NEW v5.1)
- [ ] **Medicine confirmation modal works** (NEW v5.1)
- [ ] **Medicine status persisted per patient** (NEW v5.1)
- [ ] Vitals entries have max 1 edit protection
- [ ] Patient data isolation enforced
- [ ] RM auto-generation works correctly
- [ ] Patient selector persists across reloads
- [ ] No console errors in DevTools
- [ ] Tested across devices (desktop, tablet, mobile)
- [ ] Accessibility: ARIA labels, semantic HTML

---

## 12. Deployment & Version Management

**Version Format:** `v[Major].[Minor].[Patch]`

**Current Version:** `v5.1.0-prod`

**Changelog:**
- **v5.1.0** (2026-05-25): Medicine confirmation checklist, day rotation at 00:00:01, 00:35 slot fix
- **v5.0.0** (2026-05-24): Multi-Patient Architecture
- **v4.2.0**: Vitals TTV integration
- **v4.1.0**: Developer info modal
- **v4.0.0**: Strict 30-minute expiry

---

## 13. Agent Instructions Summary

When you receive a modification request:

1. **Read this skill.md first** → Understand all constraints including v5.1 features
2. **Locate the feature** in index.html
3. **Check against constraints** → Will your change violate rules 2.1–2.7?
4. **Implement** → Modify inline HTML/CSS/JS only
5. **Test mentally** → Trace state machine + patient isolation + medicine confirmation
6. **Validate** → Confirm checklist items in section 11
7. **Document** → Add changelog entry if version-worthy
8. **Commit** → Reference this skill.md in commit message

**Forbidden:**
- ❌ Create external files
- ❌ Change to UTC timezone
- ❌ Add English labels
- ❌ Use `alert()` outside CRUD
- ❌ Allow TERLEWAT/BELUM_WAKTUNYA interaction
- ❌ Skip medicine confirmation step (v5.1)
- ❌ Store medicine status without patient scope

**Encouraged:**
- ✅ Improve UI/UX within constraints
- ✅ Optimize JavaScript performance
- ✅ Strengthen clinical data protection
- ✅ Enhance Indonesian messaging
- ✅ **Expand confirmation checklist** (v5.1)
- ✅ **Add audit logging** per medicine distribution

---

**Last Reviewed:** 2026-05-25  
**Compliance Status:** ✅ All Production Rules Active  
**Safety Audit:** ✅ Passed (v5.1.0)  
**Medicine Confirmation:** ✅ Fully Implemented  
**Day Rotation:** ✅ 00:00:01 (Fixed)

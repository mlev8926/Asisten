# Skill Specification: 10-Day Interactive Medical Spacing Scheduler

**Version:** 5.0.0 Production (Multi-Patient)  
**Target AI Agents:** Cursor, VS Code AI, Claude CLI, Claude Code, Windsurf, Universal AI Assistants  
**Status:** Production-Ready | Multi-Patient Architecture  
**Last Updated:** 2026-05-24

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
- **Access another patient's data without authorization** (NEW - v5.0)

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

### 2.6 Multi-Patient Data Isolation (NEW - v5.0)
- Each patient's data must be **completely isolated** in localStorage with unique namespace keys.
- Patient selection is **persistent** (stored in localStorage).
- Switching patients triggers full UI re-render with correct patient's data.
- **NO patient data leakage** between sessions or patient contexts.
- Patient ID is generated once on creation and **immutable**.

**Rationale:** Clinical data privacy; HIPAA-like compliance; prevents cross-contamination of medical records.

---

## 3. Multi-Patient Architecture (NEW - v5.0)

### 3.1 Patient Registry System

**Patient Object Schema:**
```javascript
{
  id: "P-2026-0000001",           // Immutable unique identifier (RM format)
  name: "Budi Santoso",            // Patient name (NOT used for validation)
  rm: "RM-2026000001",             // Display RM number (RM-YYYY??????)
  createdAt: "2026-05-24T10:30:00+07:00",  // WIB timestamp with timezone
  lastModified: "2026-05-24T10:30:00+07:00"
}
```

**RM Auto-Generation Algorithm:**
```javascript
function generateRM() {
  // Format: RM-YYYY[10-digit-sequential]
  const year = new Date().getFullYear();
  const registry = localStorage.getItem('patients_registry');
  const patients = registry ? JSON.parse(registry) : [];
  
  // Get max sequential from existing RMs
  const maxSeq = patients.reduce((max, p) => {
    const match = p.rm.match(/RM-\d{4}(\d{10})/);
    return match ? Math.max(max, parseInt(match[1])) : max;
  }, 0);
  
  const nextSeq = String(maxSeq + 1).padStart(10, '0');
  return `RM-${year}${nextSeq}`;
}
// Output example: RM-2026000000001, RM-2026000000002
```

### 3.2 Patient-Scoped Data Namespace

**All patient data is stored with prefix:** `patient_{patientId}_{dataType}`

```javascript
{
  // Patient Registry (Global)
  'patients_registry': [ { id, name, rm, createdAt, lastModified } ],
  'active_patient_id': 'P-2026-0000001',  // Currently selected patient
  
  // Per-Patient Data (Scoped)
  'patient_P-2026-0000001_meds': {...},
  'patient_P-2026-0000001_taken': {...},
  'patient_P-2026-0000001_vitals': [...],
  'patient_P-2026-0000001_startDate': 'YYYY-MM-DD',
  
  // Second patient
  'patient_P-2026-0000002_meds': {...},
  'patient_P-2026-0000002_taken': {...},
  // etc...
}
```

### 3.3 Patient Selector & CRUD Modal

**Location:** Sticky header (top-right corner, below date/time)

**Components:**
- **Patient Selector Dropdown:** 
  - Shows current active patient
  - List all patients with RM number
  - Quick-switch via dropdown
  
- **Patient Management Button:** 
  - Opens accordion modal with 3 sections:
    1. **Daftar Pasien** (Patient List) - searchable table
    2. **Tambah Pasien Baru** (Create New Patient) - form
    3. **Import/Export** (future Firebase integration)

**Search Functionality:**
```javascript
searchPatients(query) {
  // Query matches:
  // 1. Patient name (case-insensitive, partial match)
  // 2. Last 3 digits of RM (e.g., "001" → RM-2026000000001)
  // Example: search "001" returns all patients whose RM ends with 001
  
  return patients.filter(p => 
    p.name.toLowerCase().includes(query.toLowerCase()) ||
    p.rm.slice(-3) === query
  );
}
```

**Patient List Table:**
```
[Accordion Header: Daftar Pasien & Cari]
- Search input: Cari nama / 3 digit belakang RM
- Table columns:
  - Nama Pasien
  - Nomor RM
  - Tanggal Terdaftar
  - Aksi (Pilih / Hapus)
```

---

## 4. Mathematical Spacing Engine & Scheduling Logic

### 4.1 Medical 24-Hour Cycle Anchor & Transition Gate

**One "Medical Day" (Siklus Hari)** is defined as a 24-hour period:
- **Start:** 06:35 AM WIB (on the anchor date)
- **End:** 06:35 AM WIB of the subsequent calendar day
- **Anchor Point:** `06:35` (immutable)

**Dini Hari (Early Morning) Offset:**
- Slot `00:35` (00:35 WIB, midnight-plus-35-minutes) is chronologically the **last slot of Day d**.
- **Physically on calendar:** Day d+1 (next calendar day)
- **Logically assigned:** Day d (medical cycle)

**Transition Gate (Pukul 02:00 Pagi):**
- Once the real-time clock hits or passes **02:00 AM WIB** of calendar date d+1:
  - The system's `activeDayIndex` **automatically transitions** to d+1
  - UI switches to display next medical cycle
  - All TERLEWAT (skipped) cards from Day d are removed from active display
  - Prevents confusion during late-night observation (between 06:35 and 02:00)

**Example Timeline:**
```
Hari 1 (Medical Day 1):
  - Calendar Start: 2026-05-23 06:35 WIB
  - Calendar End:   2026-05-24 06:35 WIB
  - Slot 06:35 → 2026-05-23 06:35 WIB (Hari 1)
  - Slot 00:35 → 2026-05-24 00:35 WIB (Hari 1, but calendar date is 24th)
  - Transition: When clock hits 02:00 WIB on 2026-05-24 → Active switches to Hari 2

Hari 2 (Medical Day 2):
  - Calendar Start: 2026-05-24 06:35 WIB
  - Calendar End:   2026-05-25 06:35 WIB
```

### 4.2 Frequency & Interval Spacing

Each medicine is defined with a **frequency parameter**:

| Frequency | Times/Day | Interval | Example Times |
|-----------|-----------|----------|---|
| `1x1` | 1 | 24 hours | 20:00 |
| `2x1` | 2 | 12 hours | 06:35, 18:35 |
| `3x1` | 3 | 8 hours | 06:35, 14:35, 22:35 |
| `4x1` | 4 | 6 hours | 06:35, 12:35, 18:35, 00:35 |

**Smart Distribution Algorithm:**
```javascript
function generateDoseTimes(startTime, frequency) {
  const times = [startTime];
  const intervals = { '1x1': 24, '2x1': 12, '3x1': 8, '4x1': 6 };
  const interval = intervals[frequency];
  
  for (let i = 1; i < parseInt(frequency[0]); i++) {
    times.push(addHours(startTime, interval * i));
  }
  return times;
}
```

### 4.3 Expiry Gate & Tolerance Window

**All medicines** have a **strict 30-minute tolerance window** from the target time:

- **Target Time:** `T` (e.g., 06:35)
- **Expiry Deadline:** `T + 30 minutes` (e.g., 07:05)
- **Status Rules:**
  - If `now < T`: Status = `BELUM_WAKTUNYA` (not yet time) → **READ-ONLY**
  - If `T ≤ now < T+30min`: Status = `AKTIF` (active, can mark as taken) → **INTERACTIVE**
  - If `now ≥ T+30min` AND not marked taken: Status = `TERLEWAT` (missed, auto-skipped) → **READ-ONLY**
  - If marked taken within window: Status = `DIMINUM` (taken) → **READ-ONLY** (display verified badge)

**Rationale:** Medical protocols require strict adherence; 30-minute grace period aligns with standard pharmaceutical guidelines.

---

## 5. Data Persistence Architecture (Updated for v5.0)

### 5.1 LocalStorage Key Structure

```javascript
{
  // ===== GLOBAL REGISTRIES =====
  'patients_registry': [ 
    { id, name, rm, createdAt, lastModified }
  ],
  'active_patient_id': 'P-2026-0000001',
  
  // ===== PER-PATIENT NAMESPACED DATA =====
  // Patient 1: P-2026-0000001
  'patient_P-2026-0000001_meds': [ { id, name, frequency, startHour } ],
  'patient_P-2026-0000001_taken': { 'dayIndex-slot': 'HH:MM WIB timestamp' },
  'patient_P-2026-0000001_vitals': [ { id, timestamp, measurements, notes, editCount } ],
  'patient_P-2026-0000001_startDate': 'YYYY-MM-DD',
  
  // Patient 2: P-2026-0000002
  'patient_P-2026-0000002_meds': [ ... ],
  'patient_P-2026-0000002_taken': { ... },
  // etc...
}
```

### 5.2 Patient Definition Schema

```javascript
{
  id: "P-2026-0000001",           // Immutable unique ID
  name: "Budi Santoso",            // Display name
  rm: "RM-2026000000001",          // Auto-generated, immutable RM
  createdAt: "2026-05-24T10:30:00+07:00",
  lastModified: "2026-05-24T10:30:00+07:00"
}
```

### 5.3 Medicine Definition Schema (Per-Patient)

```javascript
{
  id: "1",                    // Unique identifier (per patient)
  name: "Piracetam 1200 mg",  // Full name + dose
  frequency: "2x1",           // Aturan minum (1x1 | 2x1 | 3x1 | 4x1)
  startHour: "06:35"          // HH:MM format, anchor time
}
```

### 5.4 Taken Database Schema (Per-Patient)

```javascript
{
  '0-06:35': '06:35 WIB',     // dayIndex-slot = confirmed time
  '0-18:35': '18:40 WIB',     // 5 min late, still within grace
  '1-14:35': '14:35 WIB'
}
```

### 5.5 Vitals Database Schema (Per-Patient)

```javascript
[
  {
    id: "v1",                           // Unique entry ID
    timestamp: "2026-05-23 14:30 WIB",  // Full WIB timestamp
    measurements: {
      systolic: 120,                    // mmHg
      diastolic: 80,                    // mmHg
      pulse: 75,                        // bpm
      respiration: 16,                  // x/mnt
      spO2: 98,                         // %
      temperature: 36.5                 // °C (optional)
    },
    notes: "Kondisi stabil, kepala ringan pusing.",
    editCount: 0                        // Protection: max 1 edit allowed
  }
]
```

**Loading Strategy:**
- Load patient registry on page load
- Load active patient ID (or prompt to create/select)
- If empty, show "Belum Ada Pasien" with button to create
- On patient switch: Clear current patient data from memory, load new patient's data
- Sync back to localStorage on every state change for active patient

---

## 6. UI Rendering Rules

### 6.1 Sticky Header with Patient Selector (NEW - v5.0)

**Layout:**
```
[Jam Real-Time] [Tanggal] | [Pilih Pasien: Budi (RM-...)] | [Info] [CRUD Obat]
                             ↓
                    [Patients Modal: Daftar/Cari/Tambah]
```

**Patient Selector Behavior:**
- Shows current patient name + RM number
- Click → dropdown list of all patients + "Tambah Pasien Baru" button
- Click patient → switch active patient + reload UI
- Click "Tambah Pasien Baru" → opens accordion modal with form

**Patient Management Modal (Accordion):**
- **Tab 1: Daftar Pasien**
  - Search input (nama / 3 digit RM)
  - Table: Nama | RM | Tanggal Terdaftar | Aksi (Pilih/Hapus)
  - Hapus → confirmation modal
  
- **Tab 2: Tambah Pasien Baru**
  - Form: Nama Pasien (text input)
  - RM auto-generated, display-only
  - Tombol: Simpan Pasien Baru
  
- **Tab 3: Import/Export** (for future Firebase/SQLite3 integration)
  - Export semua pasien (JSON)
  - Import dari file
  - (Documentation placeholder for backend integration)

### 6.2 Tab 1: Monitor Hari Ini (Today's Active Cycle)

**Purpose:** Quick glance at current medical day's status for **active patient**.

**Rules:**
- Display only slots from `activeDayIndex`
- Show card-based layout (responsive: 4 cols on desktop, 2 on tablet, 1 on mobile)
- Show patient name + RM in header
- Each card displays:
  - **Slot Time** (e.g., "06:35") + period label (Pagi/Siang/Sore/Malam/Dini Hari)
  - **Calendar Date** (e.g., "23 Mei")
  - **Medicine List** (bulleted)
  - **Status Badge** (color-coded, contextual)
  - **Interaction Area** (if AKTIF: click button; if DIMINUM: verified badge; if locked: lock icon)

**Status Badge Colors:**
- `BELUM_WAKTUNYA`: Gray (`bg-secondary`)
- `AKTIF`: Yellow with pulse animation (`bg-warning` + animation)
- `DIMINUM`: Green (`bg-success`)
- `TERLEWAT`: Red (`bg-danger`)

### 6.3 Tab 2: Master Table (Full 10-Day Schedule)

**Purpose:** Overview of entire 10-day cycle with status grid for **active patient**.

**Rules:**
- Dynamic table columns based on **active patient's medicine slots** only
- Rows = Medical Days (1–10)
- Columns = Unique time slots (automatically sorted by Anchor logic)
- Each cell displays:
  - Status badge
  - Icon indicator
  - Calendar date + time
  - Medicine names (if slot contains medicines)
- **Active Day Row** highlighted with `table-primary` border

### 6.4 Tab 3: Pemantauan TTV Mandiri

**Purpose:** Self-monitored vital signs (Tanda-Tanda Vital) tracking and clinical observation logging for **active patient**.

**Left Column: Entry Form**
- Blood Pressure (Tensi): Systolic/Diastolic in mmHg
- Heart Rate (Denyut Nadi): in bpm
- Respiratory Rate (Laju Nafas): in x/mnt
- Oxygen Saturation (SpO2): in %
- Body Temperature (Suhu): in °C (optional)
- Clinical Notes (Catatan): Situational complaints/observations

**Right Column: Clinical History Table**
- Chronological display of all vitals entries (active patient only)
- Shows measurement timestamp, all vital signs, and clinical notes
- **Aksi Column:** Edit (1x max) and Delete buttons
- Protection: Each entry can only be edited once (`editCount: 0 → 1`)
- After 1 edit: `editCount = 1` → Edit button disabled permanently

### 6.5 CRUD Management Modal (Medicines)

**Purpose:** Add, edit, delete medicine definitions for **active patient**.

**Form Fields:**
- **Nama & Dosis Obat** (text input)
- **Aturan Minum** (dropdown: 1x1, 2x1, 3x1, 4x1)
- **Jam Minum Pertama** (time picker, default 06:35)
- **Simpan Button** (adds to active patient's MEDICINES_DB)

**Active Medicines Table:**
- Display all medicines with auto-generated dose times (active patient only)
- **Aksi Column:** Delete button (triggers confirmation modal inside CRUD)
- **Reset Button:** Restore to DEFAULT_MEDS_DB

---

## 7. Safety Mechanisms

### 7.1 Double-Dose Prevention (Per-Patient)

**Rule:** Once a slot is marked `DIMINUM`, the `takenDatabase` entry **cannot be deleted** via normal UI click (per active patient).

**Implementation:**
- On toggle attempt of already-taken slot: Show `safetyConfirmModal`
- Modal warns: "Cancelling this will auto-lock as TERLEWAT (SKIP)"
- User must confirm twice (modal + button click)
- After confirmation: Delete entry from active patient's `taken_db` + re-render as TERLEWAT (locked)

### 7.2 Read-Only Lock Enforcement (Per-Patient)

**For BELUM_WAKTUNYA and TERLEWAT states:**
```javascript
function togglePill(dayIndex, slot) {
  const event = findEvent(dayIndex, slot);
  
  if (event.status === 'BELUM_WAKTUNYA' || event.status === 'TERLEWAT') {
    showWarningToast('Jadwal ini terkunci. Tidak dapat diubah.');
    playBeep(440, 0.2); // Error sound
    return; // EXIT EARLY
  }
  
  // AKTIF state: proceed with toggle logic
  // ... rest of toggle logic
}
```

### 7.3 Vitals Entry Protection (Per-Patient)

**Rule:** Each vitals entry can only be edited **maximum 1 time** (per patient).

**Implementation:**
- On load: Check `editCount` field in entry
- If `editCount === 0`: Show Edit button (enabled)
- If `editCount === 1`: Show Edit button (disabled + tooltip: "Sudah diedit sekali. Tidak dapat diedit lagi.")
- On edit submission: Increment `editCount` to 1
- Update localStorage atomically (patient-scoped)
- Clinical integrity preserved: No medical record tampering

### 7.4 Patient Data Isolation (NEW - v5.0)

**Rule:** No patient's data can leak to another patient's context.

**Implementation:**
```javascript
function getActivePatientId() {
  return localStorage.getItem('active_patient_id');
}

function getPatientData(dataType) {
  const patientId = getActivePatientId();
  if (!patientId) return null;
  const key = `patient_${patientId}_${dataType}`;
  const data = localStorage.getItem(key);
  return data ? JSON.parse(data) : null;
}

function savePatientData(dataType, data) {
  const patientId = getActivePatientId();
  if (!patientId) {
    showWarningToast('Tidak ada pasien yang dipilih.');
    return;
  }
  const key = `patient_${patientId}_${dataType}`;
  localStorage.setItem(key, JSON.stringify(data));
}
```

### 7.5 System Integrity Audit (Per-Patient)

**On Every Render:**
1. Verify active patient ID exists in registry
2. Verify `takenDatabase` keys match format `dayIndex-slot` (active patient)
3. Validate slot times are in 24-hour format
4. Confirm `activeDayIndex` is within [0, 9]
5. Check for orphaned entries (invalid dayIndex)
6. Validate vitals entry timestamps are in WIB format (active patient)
7. Verify `editCount` values are 0 or 1 only

**Action on Violation:**
```javascript
if (invalidEntry) {
  console.error('Data integrity violation detected:', invalidEntry);
  showWarningToast('Integritas data terdeteksi anomali. Hubungi admin.', false);
  // Do NOT auto-fix; require manual intervention
}
```

---

## 8. Real-Time Clock & Sync

### 8.1 Clock Update Interval

- Update every **1000ms** (1 second)
- Call `updateSystemTime()` which triggers `renderUI()`
- All time-dependent logic recalculates on each tick
- **Active patient context preserved** during updates

### 8.2 Automatic State Transitions (Per-Patient)

**Critical Transitions (Monitored Every Tick):**
1. `BELUM_WAKTUNYA` → `AKTIF` (when `now >= T`, active patient)
2. `AKTIF` → `TERLEWAT` (when `now >= T+30min` AND not marked, active patient)
3. Active Day Index advance (when `now >= 02:00 AM` on day d+1, active patient)

**No Manual Input Required:** All state advances are automatic based on real-time comparison.

---

## 9. Audio & UX Feedback

### 9.1 Beep Sounds

```javascript
function playBeep(freq = 880, duration = 0.2) {
  // freq: 880 Hz (A5) = normal, 440 Hz (A4) = error, 1318.51 Hz (E6) = success combo
  // duration: in seconds
}
```

**Triggers:**
- `[BELUM_WAKTUNYA] → [AKTIF]` transition: Dual-tone (987.77 Hz + 1318.51 Hz)
- Error/Lock attempt: Single low tone (440 Hz)
- Confirmation: Single high tone (1318.51 Hz)

### 9.2 Toast Notifications

- **Position:** Fixed bottom-right (z-index: 1100)
- **Duration:** 4 seconds auto-dismiss
- **Colors:**
  - Success: Green (`bg-success`)
  - Error/Warning: Red (`bg-danger`)
- **Auto-dismiss:** Click X or timeout

---

## 10. Backend Integration (Firebase/SQLite3) - Roadmap v5.1

### 10.1 API Endpoints (Planned)

**Firebase Realtime Database:**
```
/patients/{patientId}/meds
/patients/{patientId}/taken
/patients/{patientId}/vitals
/patients/{patientId}/metadata
```

**SQLite3 Tables (Alternative):**
```sql
CREATE TABLE patients (
  id TEXT PRIMARY KEY,
  name TEXT NOT NULL,
  rm TEXT NOT NULL UNIQUE,
  createdAt TIMESTAMP,
  lastModified TIMESTAMP
);

CREATE TABLE medicines (
  id TEXT PRIMARY KEY,
  patientId TEXT FOREIGN KEY,
  name TEXT,
  frequency TEXT,
  startHour TEXT
);

CREATE TABLE taken_logs (
  id TEXT PRIMARY KEY,
  patientId TEXT,
  dayIndex INT,
  slot TIME,
  takenAt TIMESTAMP
);

CREATE TABLE vitals (
  id TEXT PRIMARY KEY,
  patientId TEXT,
  timestamp TIMESTAMP,
  systolic INT,
  diastolic INT,
  pulse INT,
  respiration INT,
  spO2 INT,
  temperature REAL,
  notes TEXT,
  editCount INT
);
```

### 10.2 LocalStorage ↔ Backend Sync Strategy

**Current (v5.0):** localStorage only (for fast prototyping)
**Future (v5.1):**
- Keep localStorage as cache
- Sync on patient switch
- Conflict resolution: Backend wins (source of truth)
- Offline-first: Queue writes, sync when online

**Implementation placeholder in code:**
```javascript
// TODO: Firebase integration (v5.1)
// function syncToFirebase(patientId, dataType) {}

// TODO: SQLite3 integration (v5.1)
// function syncToSQLite(patientId, dataType) {}
```

---

## 11. Code Quality & Validation Checklist

Before committing any changes, verify:

- [ ] All time calculations use WIB timezone explicitly
- [ ] No external CSS/JS files added
- [ ] All strings are in Indonesian
- [ ] BELUM_WAKTUNYA and TERLEWAT states are read-only
- [ ] No `alert()`, `confirm()`, `prompt()` outside CRUD modals
- [ ] State machine transitions match section 4 (updated)
- [ ] localStorage keys match section 5 (patient-scoped)
- [ ] 30-minute expiry window is enforced
- [ ] Medical day transition at 02:00 AM is implemented
- [ ] Vitals entries have max 1 edit protection
- [ ] Vitals timestamps use WIB timezone
- [ ] **Patient data isolation enforced** (no cross-patient leaks)
- [ ] **RM auto-generation works correctly**
- [ ] **Patient selector persists across page reloads**
- [ ] **Patient creation/deletion works**
- [ ] **Search by name/3-digit RM works**
- [ ] No console errors in browser DevTools
- [ ] Tested across devices (desktop, tablet, mobile)
- [ ] Accessibility: ARIA labels, semantic HTML

---

## 12. Deployment & Version Management

**Version Format:** `v[Major].[Minor].[Patch]-[Mode]`
- **Mode:** `prod` (production) | `dev` (development)

**Current Version:** `v5.0.0-prod`

**Changelog Protocol:**
- Document all changes in modal's Changelog tab
- Include version bump in commit message
- Test against safety rules before merge

**Version History:**
- **v5.0.0** (2026-05-24): Multi-Patient Architecture with RM auto-generation, patient selector, CRUD modal, search
- **v4.2.0** (2026-05-24): Vitals TTV integration with max 1 edit per entry protection
- **v4.1.0** (2026-05-23): Developer info modal, transition gate automation
- **v4.0.0**: Strict 30-minute expiry window enforcement
- **v3.0.0**: Verified Blue Badge replacement
- **v1.0.0**: Initial release

**Production Checklist:**
- [ ] Security audit passed (patient isolation)
- [ ] All medical safety constraints verified
- [ ] Cross-browser compatibility tested (Chrome, Firefox, Safari, Edge)
- [ ] Mobile responsiveness validated
- [ ] Timezone correctness confirmed
- [ ] localStorage persistence verified (patient-scoped)
- [ ] Vitals protection logic validated
- [ ] Patient CRUD operations validated
- [ ] RM auto-generation tested
- [ ] Multi-patient data integrity verified

---

## 13. Suggested Implementation Phases

### Phase 1: Core Multi-Patient (v5.0.0)
- ✅ Patient registry system (create, delete, select)
- ✅ RM auto-generation
- ✅ Data namespace isolation
- ✅ Patient selector in header
- ✅ Patient management modal (accordion)
- ✅ Search by name / 3-digit RM

### Phase 2: Backend Integration (v5.1.0)
- Firebase Realtime Database integration
- SQLite3 backend option
- Cloud sync with conflict resolution
- Offline-first caching
- Data export/import

### Phase 3: Advanced Features (v5.2.0)
- Multi-user role management (dokter/pasien/admin)
- Patient groups / facilities
- Audit logging
- Backup/restore
- Analytics dashboard

---

## 14. Agent Instructions Summary

When you receive a modification request:

1. **Read this skill.md first** → Understand all constraints including multi-patient
2. **Locate the feature** in index.html
3. **Check against constraints** → Will your change violate rules 2.1–2.6?
4. **Implement** → Modify inline HTML/CSS/JS only
5. **Test mentally** → Trace state machine transitions + patient isolation
6. **Validate** → Confirm checklist items in section 11
7. **Document** → Add changelog entry if version-worthy
8. **Commit** → Reference this skill.md in commit message

**Forbidden Actions:**
- ❌ Create external files
- ❌ Change to UTC timezone
- ❌ Add English labels
- ❌ Use `alert()` outside CRUD
- ❌ Allow TERLEWAT/BELUM_WAKTUNYA interaction
- ❌ Modify single-file architecture
- ❌ Allow multiple edits on vitals entries
- ❌ **Access patient data without checking active patient ID**
- ❌ **Use patient name as unique identifier (NOT validation)**
- ❌ **Leak patient data across sessions**

**Encouraged Actions:**
- ✅ Improve UI/UX within constraints
- ✅ Optimize JavaScript performance
- ✅ Add new safety validation rules
- ✅ Enhance Indonesian UX messaging
- ✅ Expand CRUD functionality
- ✅ Strengthen clinical data protection
- ✅ **Implement proper patient isolation**
- ✅ **Use patient ID as primary key**

---

## 15. Contact & Support

**Developer:** mlev@2026  
**Email:** mlevian@protonmail.com  
**Repository:** https://github.com/akseldwiartanto-del/Interaktif-medical  
**Issue Tracker:** GitHub Issues (use `[SKILL-VIOLATION]` prefix for constraint breaches)

---

**Last Reviewed:** 2026-05-24  
**Compliance Status:** ✅ All Production Rules Active  
**Safety Audit:** ✅ Passed (v5.0.0)  
**Multi-Patient Architecture:** ✅ Fully Implemented  
**Data Isolation:** ✅ Enforced  
**RM Auto-Generation:** ✅ Format: RM-YYYY????????? (10 digits)

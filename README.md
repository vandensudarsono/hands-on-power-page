# HR Self-Service — 2–4 hour Hands-on Lab

Goal
- Build a working HR Self-Service site where Employees submit Time Off requests, Managers approve, and both see filtered lists.

Prereqs:
- Power Platform environment with Dataverse.
- Power Pages maker access + Portal Management app access.
- Power Automate access.
- Two accessible test emails (employee and manager).

Phase A — Create Site and Tables (0–45 min)

0–5 min: Create a Power Pages site
1) Go to make.powerapps.com and select your environment (top-left).
2) Left nav -> Create -> Power Pages -> Start with template.
3) Choose “Starter” (or “Blank” if preferred) -> Name: HR Self-Service -> Create.
4) Open Portal Studio from the new site card (Edit).

5–30 min: Create Dataverse tables
1) EmployeeProfile (Data -> Tables -> + New table)
   - Display name: EmployeeProfile
   - Primary name column: FullName (Text)
   - Create table -> then open it -> Columns -> + New column to add:
     - EmployeeID (Text)
     - Email (Email)
     - Contact (Lookup -> Contact) — links profile to portal contact
     - Manager (Lookup -> EmployeeProfile) — add after table exists
     - Department (Choice) -> HR, Finance, Engineering, Sales
     - HireDate (Date only)
   - Save table.

2) TimeOffRequest (Data -> Tables -> + New table)
   - Display name: TimeOffRequest
   - Primary name column: RequestNumber (Text) -> after create, change to Autonumber:
     - Columns -> RequestNumber -> Advanced options -> Autonumber
     - Pattern: REQ-{SEQNUM:0000} -> Save
   - Columns -> + New to add:
     - Employee (Lookup -> Contact)
     - FromDate (Date only)
     - ToDate (Date only)
     - Type (Choice) -> Vacation, Sick, Personal, Unpaid
     - Status (Choice) -> Submitted, PendingApproval, Approved, Rejected
     - Days (Whole Number)
     - Approver (Lookup -> Contact)
     - Notes (Multiline text)
     - Attachment (File)
   - Save & Publish table.

3) HRRequest (optional for stretch)
   - Display name: HRRequest
   - Primary name: Title (Text)
   - Columns:
     - Requestor (Lookup -> Contact)
     - Category (Choice) -> Payroll, Policy, Benefits, Other
     - Description (Multiline)
     - Status (Choice) -> New, In Progress, Resolved, Closed
     - AssignedTo (Lookup -> Contact)
   - Save & Publish.

30–45 min: Create portal Contacts and EmployeeProfiles
Step 1 — Create portal contacts (choose A or B)

Path A (self-register)
1) Ensure authentication is enabled (you can also do this in Phase C, but registration needs auth):
   - Power Pages Admin Center -> your site -> Settings -> Authentication -> enable Local and Save.
2) Open your site in a private window -> Register.
3) Register Employee using a real email (e.g., employee@contoso.com). Complete sign-in.
4) Sign out. Register Manager (manager@contoso.com). Complete sign-in.

Path B (pre-create + invite)
1) make.powerapps.com -> Data -> Tables -> Contact -> + New row:
   - Create Employee (email: employee@contoso.com) and Manager (email: manager@contoso.com).
2) Portal Management app -> Contacts -> open each -> Send invitation -> complete registration from email.

Step 2 — Assign web roles to contacts
1) Portal Management app -> Contacts -> open Employee -> Related -> Web Roles -> + New -> select “Employee” -> Save.
2) Open Manager -> Related -> Web Roles -> + New -> select “HR Manager” (and optionally “Employee”) -> Save.

Step 3 — Create EmployeeProfile rows and link contacts
1) Data -> Tables -> EmployeeProfile -> + New row (Manager profile)
   - EmployeeID: E1001
   - FullName: Ana Reid
   - Email: manager@contoso.com
   - Department: HR
   - HireDate: 2022-01-10
   - Contact: pick Manager contact (manager@contoso.com)
   - Save.
2) + New row (Employee profile)
   - EmployeeID: E1002
   - FullName: Brandon Fox
   - Email: employee@contoso.com
   - Department: Engineering
   - HireDate: 2023-03-15
   - Contact: pick Employee contact (employee@contoso.com)
   - Save.
3) Open Brandon’s EmployeeProfile -> set Manager = Ana Reid -> Save.

Optional (CSV import for EmployeeProfile)
1) EmployeeProfile -> Keys -> + New key -> Name: AK_EmployeeProfile_EmployeeID -> Columns: EmployeeID -> Save (wait until Active).
2) Data -> Get data -> Files -> upload CSV with:
   EmployeeID,FullName,Email,ManagerEmployeeID,Department,HireDate
   E1001,Ana Reid,manager@contoso.com,,HR,2022-01-10
   E1002,Brandon Fox,employee@contoso.com,E1001,Engineering,2023-03-15
3) Map normal columns; for Manager lookup, map ManagerEmployeeID to Manager via key EmployeeID.
4) After import, manually set Contact for each profile (or create a tiny Flow to match by Email and update Contact).

Phase B — Forms, Views, and Pages (45–105 min)

45–75 min: Build forms and views (Dataverse)
1) TimeOffRequest form:
   - Data -> Tables -> TimeOffRequest -> Forms -> + New form -> Main form -> Name: Portal - Submit Time Off
   - Add fields: FromDate, ToDate, Type, Notes, Attachment, Days (read-only), Employee (optional hide), Approver (optional hide), Status (hide or default to Submitted).
   - Save & Publish.
2) Views:
   - My Requests (TimeOffRequest -> Views -> + New view -> Name: My Requests)
     - Add columns: RequestNumber, FromDate, ToDate, Type, Status, Days
     - Filter: Employee equals Current Portal Contact
     - Save.
   - Pending Approvals (TimeOffRequest -> Views -> + New view)
     - Columns: RequestNumber, Employee, FromDate, ToDate, Type, Status, Days
     - Filter: Approver equals Current Portal Contact AND Status equals PendingApproval
     - Save.

75–105 min: Add pages in Portal Studio
1) Submit Time Off page:
   - Portal Studio -> Pages -> + New page -> Blank -> Name: Submit Time Off
   - Add -> Form -> Choose table: TimeOffRequest -> Form: Portal - Submit Time Off
   - Form settings:
     - Enable table permissions: On
     - Mode: Insert
     - Optional: set Status default = Submitted (use Form metadata or keep hidden field default)
   - Save.
2) My Time Off page:
   - + New page -> Blank -> Name: My Time Off
   - Add -> List -> Choose table: TimeOffRequest -> View: My Requests
   - List settings -> Enable table permissions: On -> Save.
3) Pending Approvals page (Manager):
   - + New page -> Blank -> Name: Pending Approvals
   - Add -> List -> TimeOffRequest -> View: Pending Approvals
   - Enable table permissions: On -> Save.

Phase C — Security (105–165 min)

105–120 min: Authentication
- Power Pages Admin Center -> your site -> Settings -> Authentication
  - Enable Local (or your provider) -> Save.

120–150 min: Web Roles
- Already created roles (“Employee”, “HR Manager”) in Step 2. If not:
  - Portal Management -> Web Roles -> + New -> Name: Employee -> Save.
  - + New -> Name: HR Manager -> Save.
- Assign roles to your contacts (done earlier).

150–165 min: Table permissions
1) Portal Management -> Table Permissions -> + New
   - Name: TimeOffRequest - Employee Own
   - Table: TimeOffRequest
   - Privileges: Create, Read, Write, Append, AppendTo
   - Scope: Contact
   - Relationship to Contact: Employee (lookup) equals current contact (use “Contact (current)” option)
   - Web Roles: Employee -> Save.
2) + New
   - Name: TimeOffRequest - Approver
   - Table: TimeOffRequest
   - Privileges: Read, Write
   - Scope: Contact
   - Filter: Approver (lookup) equals current contact
   - Web Roles: HR Manager -> Save.
3) Ensure your Lists/Forms have “Enable table permissions” set to On (done in Phase B).

Phase D — Approval Flow (165–225 min)

Create the flow
1) flow.microsoft.com -> Create -> Automated cloud flow
   - Name: TimeOff Approval
   - Trigger: Dataverse “When a row is added, modified or deleted”
   - Change type: Added
   - Table: TimeOffRequest
   - Scope: Organization
2) New step: Initialize variable (name: statusValue, type: String, value: Status from trigger)
3) Condition: statusValue equals Submitted
   - If No: Terminate (or do nothing).
   - If Yes: continue.

Resolve Approver from EmployeeProfile if missing
4) Condition: Approver (from trigger) is empty
   - If Yes:
     a) List rows (Dataverse) — Table: EmployeeProfile
        - Filter rows: [Contact lookup logical name] eq @{triggerBody()?['[Employee lookup logical name]']}
        - Top count: 1
     b) Get a row (Dataverse) — Table: EmployeeProfile
        - Row ID: Manager value from the previous EmployeeProfile row
     c) Get a row (Dataverse) — Table: Contact
        - Row ID: Manager’s Contact (from previous step)
     d) Update a row (Dataverse) — Table: TimeOffRequest
        - Row ID: trigger row ID
        - Approver: Manager Contact GUID
        - Status: PendingApproval
   - If No:
     - Update a row (Dataverse) — set Status: PendingApproval (optional if not already)

Start approval and handle outcome
5) Start and wait for an approval (Approvals)
   - Approval type: Approve/Reject — First to respond
   - Title: Time Off @{triggerOutputs()?['body/requestnumber']}
   - Assigned to: Approver email (if you set Approver via Contact, use that Contact’s email; else from the Manager Contact)
   - Details: Employee, FromDate, ToDate, Days, Notes
6) Condition: Outcome equals Approve
   - If Approve:
     a) Update a row (Dataverse) — Status: Approved
     b) Send an email (Office 365 Outlook) — To: Employee email; Subject: Approved; Body with details.
   - If Reject:
     a) Update a row — Status: Rejected
     b) Send an email — To: Employee email; Subject: Rejected; Body with approver comments.

Notes (Flow)
- Replace [Contact lookup logical name] and [Employee lookup logical name] with your actual schema names (e.g., new_contact, _new_employee_value). Find them in the table Columns (Name column).
- If you computed Days client-side, you don’t need to recompute in Flow.

Phase E — Client script for Days and Employee auto-fill (optional but handy) (225–240 min)

Attach Liquid snippet to expose current contact
- Add this in the page template or the page (Copy) above the Submit Time Off form.

```liquid
// filepath: (Place in your Web Template or Page Copy that renders the form)
{% if user %}
  <script>
    window.portalCurrentContactId = '{{ user.id }}';
  </script>
{% endif %}
```

Add web file compute-days.js and include it on the form page
- Portal Studio -> Content -> Web Files -> + New -> Name: compute-days.js -> upload/paste JS.
- Add a reference to this Web File on the Submit Time Off page (page HTML or “Additional content”).
- Update field logical names before use.

```javascript
// filepath: compute-days.js (upload as a Web File in Power Pages)
// ...existing code...
// TODO: Update these logical names to match your columns (find in Dataverse Columns list)
const FIELD_FROM = 'new_fromdate';
const FIELD_TO = 'new_todate';
const FIELD_DAYS = 'new_days';
const FIELD_EMPLOYEE_LOOKUP = 'new_employee';

function diffDaysInclusive(fromStr, toStr) {
  if (!fromStr || !toStr) return '';
  const from = new Date(fromStr);
  const to = new Date(toStr);
  if (isNaN(from) || isNaN(to)) return '';
  const start = new Date(from.getFullYear(), from.getMonth(), from.getDate());
  const end = new Date(to.getFullYear(), to.getMonth(), to.getDate());
  const ms = end - start;
  const days = Math.floor(ms / 86400000) + 1;
  return days > 0 ? days : '';
}

function setEmployeeLookup(id) {
  const idInput = document.querySelector(`input[name="${FIELD_EMPLOYEE_LOOKUP}"]`);
  if (idInput && id) idInput.value = id;
}

function wireUp() {
  const fromEl = document.querySelector(`input[name="${FIELD_FROM}"]`);
  const toEl = document.querySelector(`input[name="${FIELD_TO}"]`);
  const daysEl = document.querySelector(`input[name="${FIELD_DAYS}"]`);

  const onChange = () => {
    if (!fromEl || !toEl || !daysEl) return;
    daysEl.value = diffDaysInclusive(fromEl.value, toEl.value);
  };

  if (fromEl) fromEl.addEventListener('change', onChange);
  if (toEl) toEl.addEventListener('change', onChange);

  if (window.portalCurrentContactId) setEmployeeLookup(window.portalCurrentContactId);
}

if (document.readyState === 'loading') {
  document.addEventListener('DOMContentLoaded', wireUp);
} else {
  wireUp();
}

```

End-to-end test (10–15 min)
1) In one browser (normal), sign in as Employee; in another (private), sign in as Manager.
2) Employee -> Submit Time Off:
   - Fill From/To, Type, Notes; ensure Days auto-fills; Status = Submitted on submit.
3) Verify:
   - Employee -> My Time Off shows the new row.
   - Manager -> Pending Approvals shows the row (after Flow sets Status = PendingApproval).
4) Flow sends approval to Manager; approve or reject.
5) Employee list updates Status accordingly; email received.

Troubleshooting
- Form/list empty: Ensure “Enable table permissions” is On and your contact has the right Web Role.
- Can’t see Pending Approvals: Check Table Permission “Approver equals current contact” and Approver is set on the row.
- Flow didn’t fire: Change type must be Added; environment and table must match; check run history.
- Lookup logical names: In make.powerapps.com -> Table -> Columns -> copy the Name (schema) for use in Flow/JS.


# Lab Management System — User Manual

This manual explains how the Lab Management System works for the two primary user roles: TA (Teaching Assistant) and Student. It is written as a step-by-step guide from login through common workflows, and it includes concrete suggestions for screenshots you can add to your document.

How to use this file
- Read the role-specific sections (TA / Student) for user-facing workflows and UI guidance.
- Add the recommended screenshots (filenames are suggested) in the indicated places. Each screenshot suggestion includes a caption and alt text.

---

## Quick overview

The application uses standard web authentication (JWT + refresh tokens). Users log in, the backend issues tokens, and the frontend stores tokens to authorize API calls. The TA dashboard is optimized for triage and real-time conversation with students; the Student dashboard is optimized for creating and tracking help requests.

Both dashboards use real-time updates (WebSockets) for new requests and replies.

---

## Authentication Flows

### Registration

Both Students and TAs register on the same registration page with **separate tabs** to switch between roles. The registration page validates all input and sends requests to different backend endpoints based on the selected role.

**Prerequisites:**
- User has not yet created an account
- Internet connection to the backend (http://localhost:8080)
- Valid email address

**Registration Page Layout:**
- Two prominent tabs at the top: "Student Registration" and "TA Registration"
- Single form that updates labels based on selected tab
- Student tab is active by default

**Step-by-step process (Student Registration):**
1. On the login screen, click "Sign Up" link
2. The registration page opens with the **Student Registration** tab selected
3. Fill in the registration form:
   - **Student ID**: Must start with "STU" followed by 3+ digits (e.g., STU001, STU123)
   - **Username**: 3-20 characters, letters, numbers, and underscores only (e.g., john_smith_01)
   - **Email**: Valid email address (validated with regex pattern)
   - **Password**: Minimum 6 characters
   - **Confirm Password**: Must match password field
4. Click the "Register" button
5. Backend validates:
   - Student ID format (STU###)
   - Username uniqueness and format
   - Email uniqueness and format
   - Password requirements
6. On success:
   - Success dialog appears with message: "Student account created successfully! Please log in with your username and password."
   - Page automatically redirects to login screen

**Step-by-step process (TA Registration):**
1. On the login screen, click "Sign Up" link
2. On the registration page, click the **TA Registration** tab (button color changes to blue)
3. Form labels update to show "TA ID *" instead of "Student ID *"
4. Fill in the registration form:
   - **TA ID**: Must start with "TA" followed by 3+ digits (e.g., TA001, TA205)
   - **Username**: 3-20 characters, letters, numbers, and underscores only
   - **Email**: Valid email address
   - **Password**: Minimum 6 characters
   - **Confirm Password**: Must match password field
5. Click the "Register" button
6. Backend validates using the `/auth/register-ta` endpoint with same validation rules
7. On success:
   - Success dialog appears with message: "TA account created successfully! Please log in with your username and password."
   - Page automatically redirects to login screen

**Key Registration Features:**
- **Real-time validation**: Form shows error messages immediately below fields
- **Tab switching**: Form clears all fields when switching between Student/TA tabs
- **Loading indicator**: Spinning progress indicator appears during registration submission
- **Error handling**: Clear error messages for all validation failures:
  - "All fields are required"
  - "ID must be in format: STU### (e.g., STU001)"
  - "ID must be in format: TA### (e.g., TA001)"
  - "Username must be 3-20 characters, letters, numbers, and underscores only"
  - "Please enter a valid email address"
  - "Passwords do not match"
  - "Password must be at least 6 characters"
  - "Username or email already exists. Please choose a different one."

**Security notes:**
- Password is encrypted using Bcrypt (one-way hashing) — passwords cannot be recovered
- Validation happens both frontend (immediate feedback) and backend (final authority)
- No role can be overridden; role is determined strictly by which endpoint is called
- Sensitive data (passwords) never appear in logs

**What to capture (screenshot):**
- `registration_student_tab.png` — Student Registration tab selected, showing STU ID format
  - Caption: "Student Registration tab — enter Student ID starting with STU (e.g., STU001)."
  - Alt text: "Registration form with Student Registration tab active, showing STU ID field and other signup fields."
- `registration_ta_tab.png` — TA Registration tab selected, showing TA ID format
  - Caption: "TA Registration tab — enter TA ID starting with TA (e.g., TA001)."
  - Alt text: "Registration form with TA Registration tab active, showing TA ID field and other signup fields."
- `registration_error.png` — Error message displayed for invalid input
  - Caption: "Validation error — form shows immediate feedback for invalid entries."
  - Alt text: "Registration form displaying error message for invalid student ID format."

**Troubleshooting:**
- **"ID must be in format: STU### / TA###"**: ID must start with correct prefix and have 3+ digits
- **"Username already exists"**: Choose a different username (usernames are unique)
- **"Email already registered"**: Use a different email address
- **"Passwords do not match"**: Ensure both password fields are identical
- **"Password must be at least 6 characters"**: Create a longer password
- **Cannot connect to backend**: Verify Spring Boot application is running on http://localhost:8080

---
---

## TA Dashboard — Full Manual

### Purpose
The TA dashboard is the TA's workspace for receiving, claiming, replying to, and resolving student help requests. It shows a live-updating list of requests and provides tools for conversation and request management (priority, resolve).

### Login (TA)
What happens:
- TA navigates to the login screen and submits credentials.
- Backend validates credentials and returns an access token (JWT) and a refresh token.
- Frontend stores tokens and opens the TA dashboard; real-time sockets are connected.

What to capture (screenshot):
- `ta_login.png` — Login page showing username/email, password, and Login button.
  - Caption: "TA Login screen — enter credentials to access the TA dashboard."
  - Alt text: "Login form with username and password fields and a login button."

Suggested text to include in report:
- "Enter your TA credentials and click Login. Successful login opens the TA dashboard and activates live updates via WebSocket. If your token expires the client uses the refresh token to request a new access token."

### Dashboard landing / Main layout
What happens:
- After login, the dashboard shows a sidebar (navigation) and main content area.
- Common sidebar items: All Requests, In Progress, Statistics, Profile/Settings.

What to capture:
- `ta_dashboard_overview.png` — Full TA dashboard layout with sidebar and main panel.
  - Caption: "TA dashboard — sidebar navigation and main content area."
  - Alt text: "Full TA dashboard layout showing sidebar and main list panel."

Suggested text:
- "The left sidebar provides navigation. The main panel shows lists or request details depending on selection. Use the sidebar to switch between views."

### All Requests view (default for triage)
What it does:
- Displays all requests with statuses (PENDING, IN_PROGRESS, RESOLVED).
- Ordering: PENDING items appear at the top; within each status the newest items are first (most recent first).
- Controls include search, status filter, sort selector, and the action buttons per row (Claim/Assign, Priority, View Details).

User actions:
- Scan the pending list for incoming requests.
- Click the row or "View" action to open details.
- Click "Claim" to assign the request to yourself (TA). You may also assign to others depending on privilege.
- Change priority using the priority control.

What to capture:
- `ta_all_requests.png` — All Requests table with a PENDING row highlighted and the Claim button visible.
  - Caption: "All Requests view — pending requests appear at the top; use Claim to take ownership."
  - Alt text: "Table listing requests with a pending request highlighted and a Claim button."

Suggested text:
- "All Requests lists everything currently in the system. Click Claim to take ownership of a request; that moves it into In Progress and opens the conversation interface."

### In Progress view
What it does:
- Shows requests currently assigned to any TA (or to the current TA depending on configuration) and provides the reply interface and Resolve action.

User actions:
- Open a request to read the full description and conversation.
- Use the reply box to post a message; replies are stored and sent to the student in real time.
- Click Resolve to mark the request resolved.

What to capture:
- `ta_in_progress.png` — In Progress list with one assigned request and reply box visible.
  - Caption: "In Progress — active requests and the reply area for ongoing conversations."
  - Alt text: "List of in-progress requests with a reply form shown for an active request."

Suggested text:
- "In Progress shows live conversations for tasks you or other TAs are working on. Use the reply box to communicate fixes or ask follow-up questions."

### Request detail and replies
What it does:
- Shows request metadata (title, description, attachments, created time, student summary) and the threaded reply list.
- Replies display author, timestamp, and message text.

User actions:
- Read student message and context before replying.
- Use the reply box to create a reply. You may attach files if supported.

What to capture:
- `ta_request_detail.png` — Request detail showing description and metadata.
  - Caption: "Request detail — view full description, attachments and metadata."
  - Alt text: "Full request details including title, description, and attachments."
- `ta_reply_thread.png` — Reply thread with a sample TA reply and Send button highlighted.
  - Caption: "Reply thread — send replies to students using the reply box."
  - Alt text: "Message thread showing replies and a message input box."

Suggested text:
- "Open a request to see the full context. Use the reply field to communicate. Replies are saved and broadcast to the student and other connected TAs."

### Update priority
What it does:
- TAs can set a numeric priority. Higher priority requests may surface earlier in lists.

User actions:
- Click the priority control, change the value, and confirm.

What to capture:
- `ta_priority_update.png` — Small crop of priority control and visible confirmation toast.
  - Caption: "Update priority — change numeric priority to reorder requests."
  - Alt text: "Priority control input and a confirmation message."

Suggested text:
- "Adjust priority to indicate urgency. The UI will re-sort lists according to priority and time."

### Resolve request
What it does:
- Marks a request RESOLVED and removes it from the active triage lists.

User actions:
- After finishing assistance, click Resolve. Confirm if prompted.

What to capture:
- `ta_resolve_confirmation.png` — Confirmation toast and request status updated to RESOLVED.
  - Caption: "Resolve request — mark as complete to remove from active lists."
  - Alt text: "UI showing a request status changed to resolved with a confirmation message."

Suggested text:
- "Resolving a request closes the conversation and archives the request. Students will see the updated status."

### Statistics and sidebar
What it does:
- Shows quick counts such as Pending, In Progress, Resolved. These counts may be computed locally or provided by the server depending on configuration.

What to capture:
- `ta_stats_sidebar.png` — Crop of sidebar showing statistic badges.
  - Caption: "Quick stats — pending and in-progress counts for TAs."
  - Alt text: "Sidebar statistics showing counts of pending and in-progress requests."

Suggested text:
- "Use the sidebar stats for a quick sense of workload. If counts appear incorrect, refresh the page to reload data."

### Real-time behavior and error handling
What it does:
- WebSocket connection pushes new requests and replies to connected TAs and students. If the socket disconnects, the client attempts reconnect and refreshes data on reconnect.

What to capture (optional):
- `ta_live_update_before.png` and `ta_live_update_after.png` — Before and after showing a request appearing.
  - Caption: "Real-time update — new request appears without a page refresh."
  - Alt text: "Request list before and after a new request appears."

Suggested text:
- "The dashboard updates in real time. If you see stale data, check the connection status and click refresh to force a reload."

### Security & role notes for TAs
- TA endpoints are protected by role-based authorization. The backend enforces role checks even if the frontend hides buttons.

### Screenshot and formatting tips (TA)
- Use PNG format, 1280×720 or higher for clarity.
- Crop to show the relevant UI area; include column headers for table screenshots.
- Annotate the screenshot with a single colored box or arrow highlighting the control discussed.

---

## Student Dashboard — Full Manual

### Purpose
The Student dashboard allows students to create help requests, monitor their status, view TA replies, edit or delete unclaimed requests, and receive real-time updates when TAs reply or resolve requests.

### Getting Started (Registration & Login)

**First-time Student Registration:**
1. From login screen, click "Sign Up" link
2. Student Registration tab is selected by default
3. Enter Student ID (format: STU###), username, email, and password
4. Click "Register"
5. Success dialog appears, then redirects to login
6. Log in with your username and password to access Student Dashboard

**First-time TA Registration:**
1. From login screen, click "Sign Up" link
2. Click "TA Registration" tab to switch roles
3. Enter TA ID (format: TA###), username, email, and password
4. Click "Register"
5. Success dialog appears, then redirects to login
6. Log in with your username and password to access TA Dashboard

**Returning User Login:**
1. Enter your username and password on login screen
2. Click "Login" button
3. Backend authenticates credentials and returns JWT tokens
4. Frontend stores tokens and redirects to appropriate dashboard based on your role
5. Real-time WebSocket connection is established
6. Optional: Check "Remember Me" checkbox to auto-login on next launch

**Token Management:**
- Your access token expires after 24 hours
- System automatically uses refresh token to request new access token (transparent to user)
- You remain logged in unless you manually click "Logout"
- "Remember Me" saves credentials locally for auto-login (optional)

What to capture:
- `login_page_with_links.png` — Login page showing "Sign Up" link
  - Caption: "Login page — sign in with your credentials or create a new account."
  - Alt text: "Login form with username and password fields, Remember Me checkbox, and Sign Up link."

Suggested text:
- "Existing users log in with their username and password. New users click 'Sign Up' to create an account. The system redirects you to your appropriate dashboard based on your role (Student or TA)."

### Student Dashboard
What it does:
- Student opens "New Request" or "Create" form, fills title and description, optionally adds attachments, and submits; the backend saves the request with `PENDING` status.

User actions:
- Click New Request, fill fields, click Submit.

What to capture:
- `student_new_request.png` — Create Request form populated with example text.
  - Caption: "Create a new help request — provide a concise title and a full description." 
  - Alt text: "Form for creating a new help request with title and description fields."

Suggested text:
- "Provide a clear title and include steps to reproduce or code snippets where appropriate. Submit to create a PENDING request which TAs will triage."

### My Requests / status tracking
What it does:
- Displays the student's own requests with status badges and quick actions (View, Edit, Delete) for requests that aren't assigned.

User actions:
- Click View to see detail; Edit or Delete if the request is still unclaimed.

What to capture:
- `student_my_requests.png` — My Requests list showing statuses and action buttons.
  - Caption: "My Requests — track the status of your help requests."
  - Alt text: "List of student's requests with status badges and action buttons."

Suggested text:
- "Students can edit or delete requests that haven't been claimed. Once a TA claims a request, the student can still view replies and add follow-ups if allowed."

### View request and read replies
What it does:
- Shows full request details and the conversation thread of replies from TAs.

User actions:
- Read replies and add comment if the app allows student-side replies. Monitor timestamps for progress.

What to capture:
- `student_request_detail.png` — Request detail with thread showing TA replies.
  - Caption: "Request detail — view messages from TAs and the status of the request."
  - Alt text: "Detailed request view including replies from TAs."

Suggested text:
- "Open a request to see the full conversation. The student will receive replies in real time and can respond if permitted."

### Edit or delete request
What it does:
- Students may edit the title/description or delete the request while it is PENDING and not yet assigned.

What to capture:
- `student_edit_delete.png` — Edit form and delete confirmation dialog.
  - Caption: "Edit or delete your request while it is unclaimed."
  - Alt text: "Edit form and delete confirmation for a student's request."

Suggested text:
- "If you realize you need to clarify the request, edit it before a TA claims it. If the problem was resolved yourself, delete the request to keep the queue tidy."

### Real-time updates
What it does:
- When a TA replies or resolves a request you created, the dashboard updates immediately.

What to capture (optional):
- `student_real_time.png` or short animated GIF — Reply appearing in the thread.
  - Caption: "Real-time replies appear without refreshing the page."
  - Alt text: "Conversation thread before and after a new reply appears."

Suggested text:
- "Students see TA replies in real time and will be notified of status changes. If real-time fails, refresh the page."

### Privacy and safety notes for students
- Do not post sensitive personal data in request details.
- Keep messages clear and focused on troubleshooting steps or coursework questions.

---

## General documentation and screenshot checklist

Place images inline in your report immediately after the paragraph that describes the feature. Use consistent filenames and a small caption text under each image.

Suggested checklist (copy into your report editor):
- [ ] `registration_student_tab.png` — Student Registration tab with STU ID format
- [ ] `registration_ta_tab.png` — TA Registration tab with TA ID format
- [ ] `registration_error.png` — Validation error message
- [ ] `login_page_with_links.png` — Login page with "Sign Up" link and "Remember Me" checkbox
- [ ] `ta_login.png` — TA Login screen
- [ ] `ta_dashboard_overview.png` — TA dashboard full view with sidebar
- [ ] `ta_all_requests.png` — All Requests table with PENDING items
- [ ] `ta_in_progress.png` — In Progress view with reply box
- [ ] `ta_request_detail.png` — Request detail view
- [ ] `ta_reply_thread.png` — Reply thread with Send button
- [ ] `ta_priority_update.png` — Priority control
- [ ] `ta_resolve_confirmation.png` — Resolve confirmation
- [ ] `ta_stats_sidebar.png` — Sidebar statistics
- [ ] `student_new_request.png` — Create Request form
- [ ] `student_my_requests.png` — My Requests list
- [ ] `student_request_detail.png` — Student request detail with replies
- [ ] `student_edit_delete.png` — Edit/delete options for unclaimed requests

For each image provide:
- A short caption (1 line) and an alt text entry.
- Optional: a one-line instruction for how to create the screenshot (e.g., "Open TA dashboard, click All Requests, select the first PENDING row, press PrtSc.")

---

## Appendix: Notes for authors

- Sensitive data: redact student names/emails and tokens if you publish this report.
- Technical pointers: If you want to show API examples, include a `curl` example for the main endpoints (login, create request, get my requests). I can generate these snippets on request.
- If you'd like, I can produce a single merged PDF-ready markdown with images embedded (once you upload the screenshots), or a lighter HTML change log.

---

If you want, I can now:
- Add figure captions and alt text entries as separate metadata blocks for each screenshot.
- Generate `curl` examples for login, create request, claim request, reply, and resolve flows.
- Add an appendix with relevant backend file references (controllers & services) mapping to each UI feature.

Tell me which of those you'd like next and I will update the file or create a new appendix file.

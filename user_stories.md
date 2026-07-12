# User Story Template

**Title:**
_As a [user role], I want [feature/goal], so that [reason]._

**Acceptance Criteria:**
1. [Criteria 1]
2. [Criteria 2]
3. [Criteria 3]

**Priority:** [High/Medium/Low]
**Story Points:** [Estimated Effort in Points]
**Notes:**
- [Additional information or edge cases]


# User Story Template Docs

## User Story 1: Admin Login

**Title:**
_As an admin, I want to log into the portal with my username and password, so that I can manage the platform securely._

**Acceptance Criteria:**
1. Given an admin is on the login page, when they enter a valid username and password, then they are successfully authenticated and redirected to the Admin Dashboard.
2. Given an admin enters incorrect credentials, then they remain on the login page and see a clear "Invalid credentials" error message.
3. Given an unauthorized user tries to access any `/admin/*` URL directly, then they are blocked and redirected to the login page.

**Priority:** High
**Story Points:** 3
**Notes:**
- Passwords must be encrypted in the MySQL database (e.g., using BCrypt via Spring Security).
- Consider adding a maximum attempt lockout rule for security hardening.

---

## User Story 2: Admin Logout

**Title:**
_As an admin, I want to log out of the portal, so that I can protect system access when I am away from my device._

**Acceptance Criteria:**
1. Given an authenticated admin is on the dashboard, when they click the "Logout" button, then their active session is terminated.
2. Given an admin has logged out, when they try to use the browser's "Back" button, then they must not be able to view cached dashboard data and should be forced to log in again.
3. Once logged out, the user must be redirected back to the public login screen.

**Priority:** High
**Story Points:** 1
**Notes:**
- Ensure the Spring Security context is completely cleared upon logout execution.

---

## User Story 3: Add Doctor

**Title:**
_As an admin, I want to add new doctors to the portal, so that they can be registered in the system and start accepting appointments._

**Acceptance Criteria:**
1. Given an admin is on the "Add Doctor" Thymeleaf management page, when they fill out a form with a doctor's valid details (name, specialization, email, etc.) and submit, then a new record is saved to the MySQL database.
2. Given a doctor is successfully added, then a success message appears on the dashboard and the updated list of doctors is visible.
3. Given an admin attempts to add a doctor with an email address that already exists in the system, then the application must block the submission and display a validation error.

**Priority:** High
**Story Points:** 3
**Notes:**
- The repository layer must map this through the corresponding Spring Data JPA Doctor entity.

---

## User Story 4: Delete Doctor

**Title:**
_As an admin, I want to delete a doctor's profile from the portal, so that inactive or departed medical staff are removed from the system._

**Acceptance Criteria:**
1. Given an admin is viewing the Doctor List, when they click "Delete" on a specific doctor's profile, then a confirmation prompt must appear asking them to verify the choice.
2. Given the admin confirms the deletion, then the record is removed from the MySQL database and the profile disappears from the UI.
3. Given a doctor has active or pending appointments, then the system should prevent hard-deletion and warn the admin, or automatically cancel/reassign those appointments before removing the doctor.

**Priority:** Medium
**Story Points:** 2
**Notes:**
- Check for foreign key constraints in the MySQL database regarding the appointments table to prevent SQL execution crashes. Consider a soft delete flag (`is_active = false`) if data retention is preferred.

---

## User Story 5: Monthly Appointment Usage Statistics

**Title:**
_As an admin, I want to run a stored procedure in the MySQL CLI to get the number of appointments per month, so that I can track platform usage statistics over time._

**Acceptance Criteria:**
1. Given an admin logs into the MySQL CLI, when they execute the custom stored procedure call (e.g., `CALL GetMonthlyAppointmentStats()`), then the terminal must return a clean data grid.
2. The returned dataset must display columns mapping distinct months/years alongside their corresponding total count of appointments.
3. The query inside the stored procedure must correctly pull live data from the JPA-managed appointments table without locking the table or disrupting live user requests.

**Priority:** Medium
**Story Points:** 3
**Notes:**
- This is a backend database task requiring an SQL migration script to generate the routine inside the MySQL schema.
- The query logic needs to parse dates using MySQL functions like `DATE_FORMAT(appointment_date, '%Y-%m')`.

# Patient Module User Stories

## User Story 6: Public Doctor Directory Exploration

**Title:**
_As a patient, I want to view a list of doctors without logging in, so that I can explore my medical options before deciding to register._

**Acceptance Criteria:**
1. Given an unauthenticated visitor accesses the public homepage, when they click on the "Explore Doctors" link, then they must be presented with a full, searchable list of active doctors.
2. The public directory view must display basic non-sensitive information (e.g., Doctor Name, Specialization, and Availability Windows).
3. Given a visitor clicks a "Book Now" button on a doctor's profile, then the system must intercept the action and redirect them to the registration/login page.

**Priority:** High
**Story Points:** 3
**Notes:**
- This endpoint must bypass Spring Security filters to allow completely anonymous read access (`permitAll()`).
- Data is retrieved via the REST API or public Web MVC flow from the MySQL database.

---

## User Story 7: Patient Registration (Sign Up)

**Title:**
_As a patient, I want to sign up using my email and password, so that I can create an account and begin booking appointments._

**Acceptance Criteria:**
1. Given a new user is on the signup page, when they fill out their email, full name, and a valid password, then a new patient profile is successfully saved to the MySQL database.
2. Given a successful signup, then the user is automatically logged in or redirected to the login screen with a clear registration confirmation message.
3. Given an applicant enters an email address that is already associated with an existing account, then the form submission fails and shows a validation message: "Email already registered."

**Priority:** High
**Story Points:** 3
**Notes:**
- Password compliance rules must be applied server-side before persisting the data.
- User data must be securely mapped to a `Patient` JPA entity in MySQL.

---

## User Story 8: Patient Login

**Title:**
_As a patient, I want to log into the portal, so that I can manage my current bookings and access secure features._

**Acceptance Criteria:**
1. Given a registered patient is on the portal login page, when they input their valid email and password, then they are granted access and redirected to their personal patient dashboard.
2. Given a patient inputs incorrect login credentials, then they are blocked from entry, the fields clear, and a "Secure authentication failed" notice appears.
3. The session must securely store patient identifiers so subsequent API requests can accurately pinpoint which patient is interacting with the system.

**Priority:** High
**Story Points:** 2
**Notes:**
- Integrates with the Spring Security framework utilizing the exact same backend security core as the Admin portal, routed via distinct REST filters if accessed via a client app.

---

## User Story 9: Patient Logout

**Title:**
_As a patient, I want to log out of the portal, so that I can secure my account and prevent unauthorized access to my medical dashboard._

**Acceptance Criteria:**
1. Given a logged-in patient, when they select the "Logout" option, then their session tokens are completely invalidated.
2. Following logout, any attempts to use deep links or the browser "Back" button to hit private patient endpoints must automatically fail and drop the user back at the public login screen.

**Priority:** High
**Story Points:** 1
**Notes:**
- Handled primarily by clearing authorization headers or session tracking mechanisms built into the common service wrapper.

---

## User Story 10: Book an Appointment

**Title:**
_As a logged-in patient, I want to book an hour-long appointment to consult with a doctor, so that I can receive professional medical attention at a time that works for me._

**Acceptance Criteria:**
1. Given a patient is authenticated, when they select a specific doctor, select an available date/time slot, and confirm the booking, then a new appointment record is added to the MySQL database.
2. The application must enforce that every standard booking defaults to a uniform duration of exactly 1 hour.
3. Given a patient tries to book a slot that overlaps with an already scheduled appointment for that specific doctor, then the booking must be rejected with an "Appointment slot no longer available" exception.

**Priority:** High
**Story Points:** 5
**Notes:**
- Requires precise transaction management (`@Transactional`) in the Common Service Layer to completely eliminate race conditions where two patients try to book the exact same slot concurrently.

---

## User Story 11: View Upcoming Appointments

**Title:**
_As a logged-in patient, I want to view my upcoming appointments, so that I can prepare for my consultations accordingly._

**Acceptance Criteria:**
1. Given an authenticated patient loads their dashboard, then the screen must display a chronological list of all future appointments.
2. Each appointment entry must display the Doctor's name, their specialization, the exact date, and the start/end time.
3. The view must automatically filter out past or canceled appointments so the patient can focus strictly on upcoming commitments.

**Priority:** Medium
**Story Points:** 2
**Notes:**
- Data will be fetched via a custom query in the `AppointmentRepository` (e.g., `findByPatientIdAndAppointmentDateAfterOrderByAppointmentDateAsc`).


# Doctor Module User Stories

## User Story 12: Doctor Login

**Title:**
_As a doctor, I want to log into the portal, so that I can securely manage my appointments and schedule._

**Acceptance Criteria:**
1. Given a registered doctor is on the portal login page, when they enter their valid credentials, then they are successfully authenticated and redirected to the Doctor Dashboard Thymeleaf page.
2. Given a doctor enters incorrect credentials, then they are kept on the login page and shown an "Authentication failed" alert.
3. Access to any `/doctor/*` routes must be restricted strictly to authenticated users holding the `ROLE_DOCTOR` authority.

**Priority:** High
**Story Points:** 2
**Notes:**
- Authentication is handled securely by Spring Security, matching credentials against the `doctors` table in MySQL.

---

## User Story 13: Doctor Logout

**Title:**
_As a doctor, I want to log out of the portal, so that I can protect my account and sensitive medical data from unauthorized eyes._

**Acceptance Criteria:**
1. Given an authenticated doctor on the portal, when they click the "Logout" action, then their session data is immediately cleared.
2. Once logged out, any attempts to revisit the `/doctor/dashboard` via browser history or direct links must be blocked and redirected back to the login page.

**Priority:** High
**Story Points:** 1
**Notes:**
- Session invalidation must clean up all server-side context tokens instantly.

---

## User Story 14: View Appointment Calendar

**Title:**
_As a doctor, I want to view my appointment calendar, so that I can see my schedule and stay organized._

**Acceptance Criteria:**
1. Given an authenticated doctor views their dashboard, then the system must render a dynamic calendar or list layout showing all scheduled bookings.
2. Each event on the calendar must clearly display the appointment date, the designated 1-hour time slot, and the assigned patient's name.
3. The calendar data must dynamically adjust based on the doctor's current profile identity stored in the session context.

**Priority:** High
**Story Points:** 3
**Notes:**
- Handled via the Thymeleaf web MVC flow, pulling data from the MySQL `appointments` table filtered by the current doctor's ID.

---

## User Story 15: Mark Unavailability Slots

**Title:**
_As a doctor, I want to mark my unavailability, so that patients are only shown time slots where I am actually open._

**Acceptance Criteria:**
1. Given a doctor is on their schedule management page, when they select specific dates or hours to mark as "Unavailable", then these blocks are persisted to the database.
2. Given a doctor has blocked off a specific timeframe, when a patient subsequently tries to browse that doctor's open hours, then the blocked slots must be hidden or unselectable.
3. A doctor cannot mark a slot as unavailable if a patient has already successfully booked an active appointment during that exact hour.

**Priority:** Medium
**Story Points:** 3
**Notes:**
- This can be modeled by creating an `Unavailability` entity or adding an availability block rule to the MySQL schema. 

---

## User Story 16: Update Doctor Profile

**Title:**
_As a doctor, I want to update my profile with my specialization and contact information, so that patients always have up-to-date details when selecting a provider._

**Acceptance Criteria:**
1. Given a doctor accesses their "Profile Settings" page, when they modify fields like specialization, phone number, or email, then clicking "Save" updates their record.
2. The form must perform basic validation (e.g., ensuring phone numbers and emails follow standard formats) before sending data down to the database.
3. Given a profile update is successful, the changes must immediately reflect across the public doctor directory accessed by patients.

**Priority:** Medium
**Story Points:** 2
**Notes:**
- Changes directly modify the JPA Doctor entity fields and save back into the MySQL database.

---

## User Story 17: View Patient Details for Upcoming Appointments

**Title:**
_As a doctor, I want to view the patient details for upcoming appointments, so that I can be thoroughly prepared before each consultation starts._

**Acceptance Criteria:**
1. Given a doctor clicks on an upcoming appointment within their dashboard calendar, then a modal or profile summary view must display the patient's full name, contact information, age, and biological sex.
2. Given a patient has prior prescription records, then a historical list of prescriptions should be cleanly retrieved and displayed alongside their profile metadata.
3. The application must enforce strict access control, ensuring a doctor can only view files for patients who have an active or historical booking link with them.

**Priority:** High
**Story Points:** 3
**Notes:**
- Patient details are compiled by matching the Patient entity data in MySQL. 
- Historical prescription overviews will require a cross-database operation where the service fetches medical document details out of MongoDB based on the patient identifier.

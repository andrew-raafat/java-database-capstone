# Smart Clinic Management System - Database Schema Design

This document details the database architecture for the Smart Clinic Management System. The design implements a **hybrid database strategy**: 
* **MySQL** is used for core transactional data requiring strict relationships, ACID compliance, and structure.
* **MongoDB** is used for flexible, unstructured, or rapidly changing data such as clinical prescriptions, system logs, or communication records.

---

## MySQL Database Design

The relational database handles core operational entities (patients, doctors, appointments, and administration details) where data integrity, relational constraints, and strict transaction handling are paramount.

### Database Design Rules & Decisions
* **Deletions & Archiving (Soft Deletes):** To preserve medical and financial audit trails, crucial data like `patients`, `doctors`, and `appointments` should never be physically deleted (`hard delete`). Instead, a soft-delete mechanism (using an `is_active` boolean or a `deleted_at` timestamp) is used.
* **Overlapping Appointments:** Enforced via application logic and unique indexing where a combined constraint prevents a `doctor_id` from having two active appointments at the same `appointment_time`.
* **Data Integrity:** Strict constraints (`NOT NULL`, `UNIQUE`, `FOREIGN KEY` cascades) are defined at the schema level to prevent orphaned or corrupt records.

---

### Table: admin
Stores administrative credentials and system configuration flags.

- **id**: `INT`, Primary Key, Auto Increment, Not Null
- **username**: `VARCHAR(50)`, Unique, Not Null
- **email**: `VARCHAR(100)`, Unique, Not Null
- **password_hash**: `VARCHAR(255)`, Not Null (Stores securely hashed passwords)
- **created_at**: `TIMESTAMP`, Default Current Timestamp
- **updated_at**: `TIMESTAMP`, Default Current Timestamp on Update

---

### Table: patients
Stores the demographic and primary contact details of registered patients.

- **id**: `INT`, Primary Key, Auto Increment, Not Null
- **first_name**: `VARCHAR(50)`, Not Null
- **last_name**: `VARCHAR(50)`, Not Null
- **email**: `VARCHAR(100)`, Unique, Not Null
- **phone**: `VARCHAR(15)`, Unique, Not Null
- **date_of_birth**: `DATE`, Not Null
- **gender**: `VARCHAR(10)` (e.g., Male, Female, Other)
- **emergency_contact_phone**: `VARCHAR(15)`
- **is_active**: `BOOLEAN`, Default `TRUE` (Soft-delete flag)
- **created_at**: `TIMESTAMP`, Default Current Timestamp

---

### Table: doctors
Stores physician information, their clinical specialization, and standard rates.

- **id**: `INT`, Primary Key, Auto Increment, Not Null
- **first_name**: `VARCHAR(50)`, Not Null
- **last_name**: `VARCHAR(50)`, Not Null
- **specialization**: `VARCHAR(100)`, Not Null
- **email**: `VARCHAR(100)`, Unique, Not Null
- **phone**: `VARCHAR(15)`, Unique, Not Null
- **consultation_fee**: `DECIMAL(10, 2)`, Not Null
- **is_active**: `BOOLEAN`, Default `TRUE` (Soft-delete flag)
- **created_at**: `TIMESTAMP`, Default Current Timestamp

---

### Table: doctor_availability
Enables granular tracking of a doctor's weekly work hours to prevent scheduling conflicts.

- **id**: `INT`, Primary Key, Auto Increment, Not Null
- **doctor_id**: `INT`, Foreign Key → `doctors(id)` ON DELETE CASCADE
- **day_of_week**: `TINYINT`, Not Null (0 = Sunday, 1 = Monday, ..., 6 = Saturday)
- **start_time**: `TIME`, Not Null
- **end_time**: `TIME`, Not Null
- *Constraint:* Composite unique key on `(doctor_id, day_of_week, start_time)` to avoid overlapping shifts.

---

### Table: appointments
Manages the bookings, scheduling windows, and statuses.

- **id**: `INT`, Primary Key, Auto Increment, Not Null
- **doctor_id**: `INT`, Foreign Key → `doctors(id)` ON DELETE RESTRICT (Prevents deleting active doctors with booked appointments)
- **patient_id**: `INT`, Foreign Key → `patients(id)` ON DELETE RESTRICT (Prevents deleting patients with historical appointments)
- **appointment_time**: `DATETIME`, Not Null
- **status**: `TINYINT`, Default `0` (0 = Scheduled, 1 = Checked-In, 2 = Completed, 3 = Cancelled, 4 = No Show)
- **reason_for_visit**: `TEXT`
- **created_at**: `TIMESTAMP`, Default Current Timestamp
- **updated_at**: `TIMESTAMP`, Default Current Timestamp on Update
- *Constraint:* To avoid double-booking, a composite unique constraint is placed on `(doctor_id, appointment_time)` for all appointments where `status` is scheduled (0) or checked-in (1).

---

## MongoDB Collection Design

Flexible schemas in MongoDB are used to store rich, semi-structured clinical documents, communication records, and audit logs. 

Instead of breaking metadata out into dozens of relational tables, we leverage MongoDB's document nesting (embedding) to store complete logical concepts together, speeding up read performance.

### Collection: prescriptions
Prescriptions are stored in NoSQL because medication details, dosages, refills, and instructions vary wildly depending on the medical department (e.g., pediatrics vs. cardiology) and need a highly flexible schema.

```json
{
  "_id": "65abc1234567890123456789",
  "appointmentId": 1052,
  "patientId": 45,
  "patientName": "John Smith",
  "doctorId": 12,
  "doctorName": "Dr. Sarah Connor",
  "dateIssued": "2026-07-13T14:30:00Z",
  "diagnosis": "Acute Strep Throat",
  "medications": [
    {
      "name": "Amoxicillin",
      "dosage": "500mg",
      "frequency": "Three times daily",
      "durationDays": 10,
      "refillsAllowed": 0,
      "instructions": "Take with food. Complete the full course of antibiotics."
    },
    {
      "name": "Ibuprofen",
      "dosage": "400mg",
      "frequency": "Every 6 hours as needed for pain/fever",
      "durationDays": 5,
      "refillsAllowed": 1,
      "instructions": "Take with plenty of water."
    }
  ],
  "pharmacyPreference": {
    "name": "CVS Pharmacy #4820",
    "address": "102 Broadway St, Boston, MA",
    "phone": "617-555-0199"
  },
  "notes": "Follow up via patient portal if fever persists past 48 hours."
}

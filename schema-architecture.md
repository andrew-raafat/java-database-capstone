# Section 1: Architecture summary
This Spring Boot application uses both MVC and REST controllers. Thymeleaf templates are used for the Admin and Doctor dashboards, while REST APIs serve all other modules. The application interacts with two databases—MySQL (for patient, doctor, appointment, and admin data) and MongoDB (for prescriptions). All controllers route requests through a common service layer, which in turn delegates to the appropriate repositories. MySQL uses JPA entities while MongoDB uses document models.

# Section 2: Numbered flow of data and control
1.  User Request: A client initiates an action (e.g., an admin opens the AdminDashboard or a mobile app requests an appointment list).

2.  Controller Routing: Spring Boot's DispatcherServlet intercepts the request and maps it to either a @Controller (Thymeleaf UI) or a @RestController (JSON API).

3.  Service Layer Call: The controller intercepts incoming request data (like @PathVariable or @RequestBody) and invokes the appropriate method in the Common Service Layer.

4.  Business Logic Execution: The Service Layer evaluates business rules, checks permissions, and coordinates the required transactional operations.

5.  Data Source Evaluation: The service layer determines which database holds the target data and routes control to the matching repository interface.

6.  Repository Hand-off:

      For Patient, Doctor, Appointment, or Admin data, the service calls the Spring Data JPA Repository.

      For Prescription data, the service calls the Spring Data MongoDB Repository.

7.  Database Translation:

      JPA maps the Java Entity classes into relational SQL queries.

      MongoDB Spring Data maps the Java Document models into BSON queries.

8.  Database Query Execution: The SQL query runs against MySQL, or the BSON command runs against MongoDB.

9.  Data Retrieval and Mapping: The database returns the raw records, which the repository layer automatically maps back into standard Java objects.

10. Service Processing: The repository passes these objects back to the Service Layer to finalize any remaining business logic or transform them into DTOs (Data Transfer Objects).

11. Controller Payload Delivery: The Service Layer returns the processed data back to the originating controller.

12. View Generation (Thymeleaf Path): If routed to the MVC Controller, the system binds the data model to a Thymeleaf template, compiles it into a static HTML page, and streams it back to the browser.

13. Data Serialization (REST Path): If routed to the REST Controller, the system bypasses template rendering entirely, and Jackson serializes the raw Java objects straight into a JSON payload.

14. Client Render: The user's device receives either the compiled HTML page to display the dashboard or the raw JSON to update the client application interface.

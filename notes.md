# CMS,API....what is that?
To clarify the concepts and structure of your project, let's break down the details of the CMS and API, and address your questions.

### What is a CMS?

A CMS, or Content Management System, is a software application used to create, manage, and modify digital content. It is often used for managing content on websites and web applications without needing extensive coding knowledge. The CMS allows users to:

- Add, edit, and delete content (e.g., text, images, videos).
- Manage website layout and appearance.
- Handle user permissions and roles.
- Publish and schedule content.

In your project, the `app-cms` repository is responsible for providing the interface through which your partners (Islamic centers) can manage the content that will appear on the website and mobile app.

### Understanding the API

An API (Application Programming Interface) is not a predefined library but rather a set of rules and protocols for building and interacting with software applications. It allows different software systems to communicate with each other. In the context of your project:

- **app-API** is a custom API built to serve the functionalities required by your CMS and mobile app. It acts as a backend service that handles requests from the CMS and mobile app, processes them, and returns the necessary data.

Here's a more detailed explanation of your project's `app-API`:

1. **Backend Service**:
   - It is developed using Java (JDK 21) and managed with Maven (a build automation tool).
   - It's built with Spring Boot, a Java framework that simplifies the development of Java applications.
   - The API interacts with a MySQL database to fetch, update, or delete data.
   - It also manages email services, likely for user notifications or account management.

2. **Running the API Locally**:
   - Ensure the MySQL database and email service are configured correctly.
   - Use the Maven command `mvn spring-boot:run` to start the API locally.

3. **Deployment**:
   - The API is deployed to Amazon ECS (Elastic Container Service) using GitHub actions, which automates the deployment process.

### Overview of the `app-cms` Repository

The `app-cms` repository is the frontend component that allows content management:

1. **Frontend Service**:
   - Developed using Node.js and managed with NPM (Node Package Manager).
   - It provides an interface for partners to manage content that will be displayed on the website and mobile app.

2. **Running the CMS Locally**:
   - Configure the environment variables by renaming and updating `.env.local.sample` to `.env.local`.
   - Ensure the API service is running since the CMS depends on it to function correctly.
   - Use the command `npm run dev` to start the development server and access the CMS interface at `http://localhost:3000`.

3. **Deployment**:
   - The CMS is also deployed to Amazon ECS using GitHub actions, similar to the API.

### How the API and CMS Work Together

- The CMS (frontend) allows users to manage content and configurations.
- When users perform actions on the CMS (e.g., adding new content), the CMS sends requests to the API.
- The API (backend) processes these requests, interacts with the database, and returns the required data to the CMS.
- The website and mobile app then fetch this managed content from the API to display it to the end-users.

### Summary

- **CMS (app-cms)**: A content management system for partners to manage and configure content. Developed with Node.js.
- **API (app-API)**: A backend service providing functionalities and data to the CMS and mobile app. Developed with Java (Spring Boot).

This setup separates content management from backend processing.

# So my app (CMS) isn't used by clients , isn't yes?
Yes, you are correct in your understanding. Let me clarify the roles of the CMS and the API in your project:

### CMS (Content Management System)

1. **Purpose**:
   - The CMS is essentially an admin dashboard.
   - It is used by the administrators or partners (in this case, Islamic centers) to manage content.
   - The content managed through the CMS will be displayed on the actual client-facing applications (the website and mobile app used by the final client).

2. **Users**:
   - The primary users of the CMS are the administrators or content managers.

### API (Application Programming Interface)

1. **Purpose**:
   - The API provides backend functionalities to both the CMS and the client-facing applications (website and mobile app).
   - It handles data processing, database interactions, and other backend logic.
   - It acts as a bridge between the front end (CMS and client-facing apps) and the database.

2. **Users**:
   - The API is not accessed directly by end-users.
   - It is used by the CMS and the client-facing applications to fetch and manipulate data.

### Client-Facing Applications (magahnkhdmuch 3liha hna)

1. **Purpose**:
   - These are the applications used by the end-users (clients) to interact with the services offered by the Islamic centers.
   - They display the content managed through the CMS.

2. **Users**:
   - The end-users or clients who want to access the services provided by the Islamic centers.

# dev and prod DBs:
### Understanding Database Deployment in Production

1. **Local Development vs. Production Environment**:
   - **Local Development**: During development, you use a local database (e.g., on phpMyAdmin) to test and build the application. This environment is isolated and meant for testing purposes only.
   - **Production Environment**: In a live, deployed application, a separate database instance is used. This is typically hosted on a cloud service like AWS, Azure, or another database hosting provider. This ensures that the application can scale, is secure, and provides high availability.

2. **Database Instances**:
   - **Development Database**: This is the database you set up on your local machine or a development server. It’s used to test new features and changes without affecting the live application.
   - **Production Database**: This is a separate database instance specifically set up for the live application. It is configured to handle actual user data securely and is hosted on a secure and scalable cloud environment.

### How Data Management Works

1. **User Interaction with the Application**:
   - When a user adds an item (e.g., events, announcements) through your application, the request is sent to the application’s API.
   - The API then processes this request and interacts with the production database to store or retrieve data.

2. **Separate Production Database**:
   - Your production database is separate from your development database. This ensures that user data is stored securely and is not mixed with your local development data.
   - For example, if your application is deployed on AWS, you might use Amazon RDS (Relational Database Service) to host your production database.

3. **Database Security and Access**:
   - Only the deployed application and authorized services can access the production database. This is managed through secure authentication and access control mechanisms.
   - As a developer or administrator, you typically won’t have direct access to the production database to view user data unless it's required for maintenance or debugging, and even then, access is controlled and audited.

### How to Set Up and Use a Production Database

1. **Create a Cloud Database Instance**:
   - Use a cloud provider like AWS, Azure, or Google Cloud to create a new database instance.
   - For instance, on AWS, you can create an RDS instance, set up the database schema, and configure security settings.

2. **Configure Your Application to Use the Production Database**:
   - Update your application configuration to point to the new production database. This involves setting the database host, user, password, and other connection parameters in your application’s configuration files.
   - Ensure that environment-specific settings (like connection strings) are not hardcoded and use environment variables for security.

3. **Deploying the Application**:
   - When you deploy your application to a live environment (e.g., using GitHub Actions to deploy to Amazon ECS), it will use the configuration for the production database.
   - The application will interact with this database for all data operations (e.g., adding items, retrieving content).

4. **Managing Data Security**:
   - Implement data encryption both at rest (in the database) and in transit (data being sent to/from the database).
   - Use roles and permissions to control who can access or modify the data.
   - Regularly back up the database and have a disaster recovery plan in place.

### Example Configuration for Connecting to a Production Database

Here’s a basic example of how you might configure a Java application using Spring Boot to connect to a production database:

#### `application.properties` for Development:
```properties
# Local development database configuration
spring.datasource.url=jdbc:mysql://localhost:3306/my_local_db
spring.datasource.username=local_user
spring.datasource.password=local_password
```

#### `application-prod.properties` for Production:
```properties
# Production database configuration
spring.datasource.url=jdbc:mysql://production-db-instance.rds.amazonaws.com:3306/my_production_db
spring.datasource.username=prod_user
spring.datasource.password=prod_secure_password
```

### Switching Between Environments

To switch between different configurations, you can use profiles in Spring Boot. When you deploy, specify the profile to use:

```bash
# Run the application with the production profile
mvn spring-boot:run -Dspring-boot.run.profiles=prod
```

### Summary

- **Separate Environments**: Use different databases for development and production to ensure data security and proper testing.
- **Cloud Databases**: Set up and configure a production database on a cloud platform.
- **Secure Access**: Ensure that only authorized entities can access the production database, and use secure configurations to manage database connections.
- **Environment-Specific Configurations**: Use environment-specific configurations to manage database connections and settings.

This approach ensures that user data is securely managed in a production environment, separate from your development environment, preventing any unauthorized access to sensitive information.

If you have any further questions or need more details on any specific part, feel free to ask! 😊🔧

# application-prod.properties (for Production) how is that file created???


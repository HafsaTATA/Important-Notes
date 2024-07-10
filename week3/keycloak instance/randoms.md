deploy.yml (GitHub Actions Workflow):

    Purpose: The deploy.yml file is a GitHub Actions workflow configuration file. It automates the CI/CD pipeline, defining steps for building, testing, and deploying your application.
    Role in Deployment: The workflow automates the build and deployment process:
        Build Step: Checks out the code, sets up the JDK, runs tests, and builds the Docker image using the Dockerfile.
        Push Step: Logs into Amazon ECR (Elastic Container Registry), builds the Docker image, and pushes it to the ECR.
        Deploy Step: Connects to a remote server via SSH, pulls the latest Docker image from ECR, and runs it on the server.
------------------------------------------------------------------------------------------------------------------------------------------------
Production Database Configuration

In a production environment, databases are often managed separately from the application containers. This can be for several reasons, including:

    Persistence: Databases require persistent storage that is not lost when containers are destroyed and recreated.
    Scalability: Applications can scale independently of the database.
    Security: Databases can be configured with specific security settings and managed by a dedicated team.

How to Set Up a Production Database:

    Managed Database Services:
        Use cloud-managed database services like AWS RDS, Google Cloud SQL, or Azure Database.
        These services handle backups, scaling, and maintenance.

    Self-Hosted Databases:
        Run your database on a separate server or VM, and connect your application to this database.

    Using Environment Variables:
        In your deployment configuration (like deploy.yml), set environment variables to configure your application to connect to the production database.
------------------------------------------------------------------------------------------------------------------------------------------------
Docker Compose:

    Not necessary if services are running on separate instances because Docker Compose is primarily used to manage multi-container applications on the same host.
    Use individual Docker commands and scripts for deployment and management

=>No Docker Compose:

    Justification: Since Next.js and Keycloak are on separate EC2 instances, Docker Compose (which manages multiple containers on the same host) is not necessary.
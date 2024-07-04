Let's break down your GitHub Actions file step by step and make the necessary adjustments to achieve the following goals:

1. **Run tests when a pull request (PR) is created or updated to ensure everything works before merging into the main branch.**
2. **Automatically deploy the application when changes are pushed to the main branch.**

### Current GitHub Actions File

Here's your current GitHub Actions file with explanations:

```yaml
name: backend ci

permissions:
  id-token: write  # Required for requesting the JWT
  contents: read  # Required for actions/checkout

on:
  pull_request:
    branches: [ main ]  # Trigger on pull requests to the main branch
    paths:
      - 'src/**'  # Trigger when changes are made in the src directory
      - '.github/workflows/deploy.yml'  # Trigger when the workflow file is changed

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout Code
      uses: actions/checkout@v4
      with:
        ref: ${{ github.ref }}  # Checkout the branch or commit that triggered the workflow

    - name: Set up JDK 21
      uses: actions/setup-java@v4
      with:
        java-version: '21'
        distribution: 'temurin'
        cache: maven  # Setup Java and Maven caching for faster builds

    - name: Run tests with Maven
      run: mvn -B verify -Dmaven.test.skip=false --file pom.xml  # Run Maven tests

    - name: Configure AWS Credentials 
      uses: aws-actions/configure-aws-credentials@v3
      with:
        role-to-assume: arn:aws:iam::687456531778:role/github_actions 
        aws-region: us-east-2  # Set AWS region

    - name: Login to Amazon ECR
      uses: aws-actions/amazon-ecr-login@v2
      id: login-ecr  # Login to Amazon ECR (Elastic Container Registry)

    - name: Echo registry
      run: echo ${{ steps.login-ecr.outputs.registry }}  # Output the registry URL for debugging

    - name: Build, tag, and push docker image to Amazon ECR
      env:
        REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        REPOSITORY: islam-center
        IMAGE_TAG: latest
      run: |
        docker build -f "dockerfile.yml" -t $REGISTRY/$REPOSITORY:$IMAGE_TAG .  # Build the Docker image
        docker push $REGISTRY/$REPOSITORY:$IMAGE_TAG  # Push the Docker image to ECR

  deploy:
    runs-on: ubuntu-latest
    needs: build  # Run deploy job only after the build job is successful

    steps:
    - name: Pull docker image on the server and run it
      uses: appleboy/ssh-action@v1.0.3
      with:
        host: ${{ secrets.PROD_HOST }}  # The hostname of your production server
        username: ${{ secrets.PROD_USER }}  # The SSH username for your production server
        key: ${{ secrets.TESTPROD_SSH_KEY }}  # The SSH key to connect to your production server
        script: |
            aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin 687456531778.dkr.ecr.us-east-2.amazonaws.com  # Login to ECR
            docker pull 687456531778.dkr.ecr.us-east-2.amazonaws.com/islam-center:latest  # Pull the latest Docker image
            docker container stop api  # Stop the existing container
            docker container rm api  # Remove the existing container
            docker run -d -p 8085:8085 -v /var/islamcenters:/var/islamcenters \
            --env-file /var/islamcenters/.env_api \
            --name api 687456531778.dkr.ecr.us-east-2.amazonaws.com/islam-center:latest  # Run the new container
```

### Adjusted GitHub Actions File

We will modify this file to handle both the pull request and push events appropriately:

- **For Pull Requests**: Run tests to ensure everything works before merging.
- **For Pushes to Main**: Build and push the Docker image, then deploy the application.

#### Updated Workflow File

```yaml
name: backend ci

permissions:
  id-token: write  # Required for requesting the JWT
  contents: read  # Required for actions/checkout

on:
  pull_request:
    branches: [ main ]  # Trigger on pull requests to the main branch
    paths:
      - 'src/**'  # Trigger when changes are made in the src directory
      # Optionally include deploy.yml if you want to test workflow changes in PRs
      # - '.github/workflows/deploy.yml'  

  push:
    branches: [ main ]  # Trigger on pushes to the main branch
    paths:
      - 'src/**'  # Trigger when changes are made in the src directory
      - '.github/workflows/deploy.yml'  # Trigger when the workflow file is changed

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout Code
      uses: actions/checkout@v4
      with:
        ref: ${{ github.ref }}  # Checkout the branch or commit that triggered the workflow

    - name: Set up JDK 21
      uses: actions/setup-java@v4
      with:
        java-version: '21'
        distribution: 'temurin'
        cache: maven  # Setup Java and Maven caching for faster builds

    - name: Run tests with Maven
      run: mvn -B verify -Dmaven.test.skip=false --file pom.xml  # Run Maven tests

    - name: Configure AWS Credentials 
      uses: aws-actions/configure-aws-credentials@v3
      with:
        role-to-assume: arn:aws:iam::687456531778:role/github_actions 
        aws-region: us-east-2  # Set AWS region

    - name: Login to Amazon ECR
      uses: aws-actions/amazon-ecr-login@v2
      id: login-ecr  # Login to Amazon ECR (Elastic Container Registry)

    - name: Echo registry
      run: echo ${{ steps.login-ecr.outputs.registry }}  # Output the registry URL for debugging

    - name: Build, tag, and push docker image to Amazon ECR
      if: github.event_name == 'push'  # Only run this step on push events
      env:
        REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        REPOSITORY: islam-center
        IMAGE_TAG: latest
      run: |
        docker build -f "dockerfile.yml" -t $REGISTRY/$REPOSITORY:$IMAGE_TAG .  # Build the Docker image
        docker push $REGISTRY/$REPOSITORY:$IMAGE_TAG  # Push the Docker image to ECR

  deploy:
    runs-on: ubuntu-latest
    needs: build  # Run deploy job only after the build job is successful
    if: github.event_name == 'push'  # Only run this job on push events

    steps:
    - name: Pull docker image on the server and run it
      uses: appleboy/ssh-action@v1.0.3
      with:
        host: ${{ secrets.PROD_HOST }}  # The hostname of your production server
        username: ${{ secrets.PROD_USER }}  # The SSH username for your production server
        key: ${{ secrets.TESTPROD_SSH_KEY }}  # The SSH key to connect to your production server
        script: |
            aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin 687456531778.dkr.ecr.us-east-2.amazonaws.com  # Login to ECR
            docker pull 687456531778.dkr.ecr.us-east-2.amazonaws.com/islam-center:latest  # Pull the latest Docker image
            docker container stop api  # Stop the existing container
            docker container rm api  # Remove the existing container
            docker run -d -p 8085:8085 -v /var/islamcenters:/var/islamcenters \
            --env-file /var/islamcenters/.env_api \
            --name api 687456531778.dkr.ecr.us-east-2.amazonaws.com/islam-center:latest  # Run the new container
```

### Explanation

1. **Trigger Events**:
   - **pull_request**: The workflow runs when a pull request is made to the `main` branch.
   - **push**: The workflow runs when changes are pushed to the `main` branch.

2. **Jobs**:
   - **build**: This job runs for both pull requests and push events.
     - **Steps**:
       - **Checkout Code**: Downloads the latest code from the repository.
       - **Set up JDK**: Sets up the Java Development Kit (JDK) needed for building the project.
       - **Run tests with Maven**: Runs the tests defined in your project using Maven.
       - **Configure AWS Credentials**: Configures the AWS credentials using an IAM role.
       - **Login to Amazon ECR**: Logs into Amazon Elastic Container Registry (ECR).
       - **Build, tag, and push Docker image**: This step runs only for push events to build and push the Docker image to ECR.

   - **deploy**: This job runs only for push events after the build job is successful.
     - **Steps**:
       - **Pull docker image on the server and run it**: Connects to the production server, pulls the Docker image from ECR, and runs it.

### Summary

- **Pull Requests**: When a pull request is made, the `build` job runs tests to ensure the changes are correct.
- **Push to Main**: When changes are

 pushed to the main branch, the `build` job builds and pushes the Docker image, and the `deploy` job deploys the new image to the production server.

This setup ensures that your application is tested before merging changes and automatically deployed when changes are pushed to the main branch. If you have any more questions or need further clarification, feel free to ask!
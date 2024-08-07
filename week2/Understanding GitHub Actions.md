# Understanding GitHub Actions

**GitHub Actions** is a CI/CD (Continuous Integration and Continuous Deployment) platform that allows you to automate your software workflows directly in your GitHub repository. With GitHub Actions, you can create workflows that build, test, and deploy your code right from GitHub.

### Key Concepts

1. **Workflow**: A configurable automated process made up of one or more jobs.
2. **Job**: A set of steps that execute on the same runner.
3. **Step**: An individual task that can run commands, scripts, or actions.
4. **Action**: A custom application that performs a repeated task.
5. **Runner**: A server that runs the workflows when they're triggered.

### Why Use GitHub Actions?

- **Automation**: Automate tasks like testing, building, and deploying your code.
- **Integration**: Integrates seamlessly with GitHub repositories.
- **Flexibility**: Supports complex workflows and custom scripts.
- **Collaboration**: Enhances collaboration through automated testing and deployments on pull requests.

### Basic Structure of a GitHub Actions Workflow

A GitHub Actions workflow is defined in a YAML file stored in the `.github/workflows` directory of your repository.

#### Example of a Simple Workflow

This example shows a basic workflow that runs tests on a Node.js project whenever code is pushed to the repository:

```yaml
name: CI

on: [push]

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - name: Set up Node.js
      uses: actions/setup-node@v2
      with:
        node-version: '14'
    - run: npm install
    - run: npm test
```

### Components of the Example Workflow

1. **name**: The name of the workflow (`CI`).
2. **on**: The event that triggers the workflow (`push`).
3. **jobs**: Defines the jobs in the workflow.
4. **build**: The job name.
5. **runs-on**: Specifies the runner environment (`ubuntu-latest`).
6. **steps**: The individual steps to be executed in the job.
7. **uses**: Uses an action (`actions/checkout@v2` and `actions/setup-node@v2`).
8. **run**: Runs a command directly (`npm install` and `npm test`).

### Key Files and Directories in a GitHub Actions Workflow

1. **`.github/workflows`**: Directory containing YAML files that define the workflows.
2. **`workflow.yml`**: Each YAML file in this directory represents a workflow.

### Example Workflow Files

#### Node.js Example

**`.github/workflows/nodejs.yml`**:
```yaml
name: Node.js CI

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [10, 12, 14]

    steps:
    - uses: actions/checkout@v2
    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v2
      with:
        node-version: ${{ matrix.node-version }}
    - run: npm install
    - run: npm test
    - run: npm run build --if-present
    - run: npm run deploy --if-present
```

#### Python Example

**`.github/workflows/python.yml`**:
```yaml
name: Python CI

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        python-version: [3.6, 3.7, 3.8, 3.9]

    steps:
    - uses: actions/checkout@v2
    - name: Set up Python ${{ matrix.python-version }}
      uses: actions/setup-python@v2
      with:
        python-version: ${{ matrix.python-version }}
    - run: python -m pip install --upgrade pip
    - run: pip install -r requirements.txt
    - run: pytest
```

### Benefits of Using GitHub Actions

- **Integrated CI/CD**: Directly integrated with your GitHub repository.
- **Scalable**: Scales with your project's needs.
- **Cost-effective**: Free for public repositories, with generous free tier for private repositories.
- **Customizable**: Highly customizable workflows to fit your project's requirements.

### Steps to Create a Workflow

1. **Create `.github/workflows` directory**: In your GitHub repository, create a directory named `.github/workflows`.
2. **Add a workflow file**: Create a YAML file in this directory, e.g., `ci.yml`.
3. **Define the workflow**: Use the YAML syntax to define the events, jobs, and steps.

### Example: Setting Up Your Workflow for the Project

Let’s say you have a Java project with Maven. Here’s a simple GitHub Actions workflow file to build and test your project:

**`.github/workflows/ci.yml`**:
```yaml
name: Java CI

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - name: Set up JDK 11
      uses: actions/setup-java@v2
      with:
        java-version: '11'
    - name: Build with Maven
      run: mvn -B package --file pom.xml
    - name: Run tests with Maven
      run: mvn test
```

### Summary

- **GitHub Actions**: A CI/CD platform integrated with GitHub repositories to automate workflows.
- **Workflows**: Defined in YAML files stored in `.github/workflows`.
- **Jobs**: Consist of steps to perform tasks like building, testing, and deploying.
- **Triggers**: Workflows are triggered by events like pushes or pull requests.

By understanding these basics, you can create and customize GitHub Actions workflows to automate your project's CI/CD processes effectively. If you need further assistance or have specific questions about your project’s GitHub Actions file, feel free to ask!

# More simple explanation of key concepts:
Let's break down the key concepts of GitHub Actions with simpler explanations and then use examples from the provided GitHub Actions files to illustrate each concept.

### Key Concepts

1. **Workflow**: 
   - **Simple Explanation**: A workflow is like a recipe. It lists the steps you need to follow to complete a task.
   - **Example**: Imagine a workflow that builds and tests your code every time you push changes to your repository. 

2. **Job**: 
   - **Simple Explanation**: A job is a set of instructions that run together on the same machine.
   - **Example**: In our recipe, a job could be "baking the cake," which includes mixing ingredients, baking them, and then cooling the cake.

3. **Step**: 
   - **Simple Explanation**: A step is a single task or command in a job.
   - **Example**: In the "baking the cake" job, a step could be "mix the ingredients."

4. **Action**: 
   - **Simple Explanation**: An action is a reusable piece of code that performs a specific task.
   - **Example**: An action could be a pre-made script that sets up a Node.js environment.

5. **Runner**: 
   - **Simple Explanation**: A runner is a computer that runs the jobs in your workflow.
   - **Example**: It's like the kitchen where you bake your cake.

### Example GitHub Actions File

Let's use the GitHub Actions file you provided to identify each concept:

```yaml
name: backend ci

permissions:
  id-token: write  # Required for requesting the JWT
  contents: read  # Required for actions/checkout

on:
  pull_request:
    branches: [ main ]  
    paths:
      - 'src/**'  
      - '.github/workflows/deploy.yml'  

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout Code
      uses: actions/checkout@v4
      with:
        ref: ${{ github.ref }}

    - name: Set up JDK 21
      uses: actions/setup-java@v4
      with:
        java-version: '21'
        distribution: 'temurin'
        cache: maven

    - name: Run tests with Maven
      run: mvn -B verify -Dmaven.test.skip=false --file pom.xml

    - name: Configure AWS Credentials 
      uses: aws-actions/configure-aws-credentials@v3
      with:
        role-to-assume: arn:aws:iam::687456531778:role/github_actions 
        aws-region: us-east-2

    - name: Login to Amazon ECR
      uses: aws-actions/amazon-ecr-login@v2
      id: login-ecr

    - name: Echo registry
      run: echo ${{ steps.login-ecr.outputs.registry }}

    - name: Build, tag, and push docker image to Amazon ECR
      env:
        REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        REPOSITORY: islam-center
        IMAGE_TAG: latest
      run: |
        docker build -f "dockerfile.yml" -t $REGISTRY/$REPOSITORY:$IMAGE_TAG .
        docker push $REGISTRY/$REPOSITORY:$IMAGE_TAG

  deploy:
    runs-on: ubuntu-latest
    needs: build
    steps:
    - name: Pull docker image on the server and run it
      uses: appleboy/ssh-action@v1.0.3
      with:
        host: ${{ secrets.PROD_HOST }}
        username: ${{ secrets.PROD_USER }}
        key: ${{ secrets.TESTPROD_SSH_KEY }}
        script: |
            aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin 687456531778.dkr.ecr.us-east-2.amazonaws.com
            docker pull 687456531778.dkr.ecr.us-east-2.amazonaws.com/islam-center:latest
            docker container stop api
            docker container rm api
            docker run -d -p 8085:8085 -v /var/islamcenters:/var/islamcenters \
            --env-file /var/islamcenters/.env_api \
            --name api 687456531778.dkr.ecr.us-east-2.amazonaws.com/islam-center:latest
```

### Identifying Key Concepts

1. **Workflow**: The entire YAML file represents a workflow. It defines when and how to run a set of jobs. In this example, the workflow is named `backend ci`.
   ```yaml
   name: backend ci
   ```

2. **Job**: The workflow contains two jobs, `build` and `deploy`. Each job runs on a separate machine.
   ```yaml
   jobs:
     build:
       ...
     deploy:
       ...
   ```

3. **Step**: Each job contains multiple steps. Each step performs a specific task, such as checking out the code, setting up Java, running tests, etc.
   ```yaml
   steps:
   - name: Checkout Code
     uses: actions/checkout@v4
     ...
   - name: Set up JDK 21
     uses: actions/setup-java@v4
     ...
   ```

4. **Action**: Some steps use actions. Actions are predefined pieces of code that perform specific tasks. For example, `actions/checkout@v4` is an action that checks out the code from the repository.
   ```yaml
   - uses: actions/checkout@v4
   - uses: actions/setup-java@v4
   - uses: aws-actions/configure-aws-credentials@v3
   - uses: aws-actions/amazon-ecr-login@v2
   - uses: appleboy/ssh-action@v1.0.3
   ```

5. **Runner**: The `runs-on` key specifies the runner, which is the environment where the job runs. In this case, both jobs run on `ubuntu-latest`.
   ```yaml
   runs-on: ubuntu-latest
   ```

### Why We Create GitHub Actions Workflows

- **Automate Repetitive Tasks**: Automate tasks such as testing, building, and deploying code, which saves time and reduces errors.
- **Continuous Integration (CI)**: Automatically test code changes to ensure they work with the existing codebase.
- **Continuous Deployment (CD)**: Automatically deploy code changes to production or staging environments.
- **Integration with GitHub**: Easily integrate with GitHub events such as push, pull requests, and more.

By creating workflows, you ensure that your code is consistently built, tested, and deployed, which improves code quality and speeds up the development process.

### Summary

- **Workflow**: The entire process defined in the YAML file.
- **Job**: A group of steps executed on the same runner.
- **Step**: An individual task within a job.
- **Action**: A reusable piece of code used in a step.
- **Runner**: The environment where the job runs.

This breakdown should help you understand the core components of GitHub Actions. If you have any more questions or need further clarification, feel free to ask!
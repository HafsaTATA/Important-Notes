I understand the confusion. Let's clarify the GitHub Actions workflow and how triggering events work:

### GitHub Actions Workflow Triggers:

1. **Pull Request (`on: pull_request`):**
   - When you define `on: pull_request`, the workflow triggers whenever a pull request is opened, updated (new commits added), or synchronized (rebased or updated) on the specified branches (`main` in your case).
   - This trigger allows you to run checks, tests, or other actions on proposed changes before they are merged into the `main` branch.
   - Typically, this workflow stage involves running tests, performing code quality checks, and ensuring that the changes adhere to your project's standards.

2. **Push (`on: push`):**
   - When you define `on: push`, the workflow triggers whenever a direct push is made to the specified branches (`main` in your case).
   - This trigger is useful for actions that should occur after changes are merged directly into the branch, such as deploying the application or updating documentation.
   - It skips the pull request validation steps and assumes that the changes are ready to be integrated into the main project branch.

### Workflow Execution:

- **Pull Request Events:**
  - When a pull request event triggers the workflow (`on: pull_request`), GitHub Actions sets up a virtual environment (specified by `runs-on`) and executes the defined jobs (`build` and `deploy` in your case).
  - The workflow runs steps such as checking out the code, setting up dependencies (like JDK and Maven), running tests, and possibly deploying to a staging environment for further testing.

- **Push Events:**
  - When a push event triggers the workflow (`on: push`), it follows similar steps as above but assumes that the changes are directly affecting the main branch.
  - This trigger can lead to actions like building the Docker image, pushing it to a container registry, and deploying it to a production environment.

### Deployment Clarification:

- **Deployment Steps (`deploy` job):**
  - In your workflow, the `deploy` job (defined after the `build` job) specifies tasks for pulling the Docker image from Amazon ECR and running it on your production server (`PROD_HOST`).
  - This step ensures that the latest Docker image built and pushed during the workflow execution (`docker push`) is deployed to your production environment.

### Conclusion:

- The `on: pull_request` trigger ensures that changes proposed via pull requests are properly tested and validated before merging into the `main` branch.
- The `on: push` trigger facilitates the automation of tasks, including building, testing, and deploying changes directly to production after they are merged into the `main` branch.
- Together, these triggers support continuous integration (CI) and continuous deployment (CD) practices, automating workflows to enhance development efficiency and ensure reliable software delivery.
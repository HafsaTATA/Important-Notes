# what should i do?
To deploy your backend application from GitHub to AWS EC2 using Docker and GitHub Actions, we'll follow a structured approach. I'll explain each step in detail and provide the necessary commands and configurations.

### Step-by-Step Instructions

#### 1. **Set Up AWS EC2 Instance**

1. **Launch an EC2 Instance**:
   - Go to the AWS Management Console and navigate to EC2.
   - Click "Launch Instance" and follow the wizard to configure your instance:
     - Choose an Amazon Machine Image (AMI): Select Amazon Linux 2 or Ubuntu.
     - Choose an Instance Type: `t2.micro` is sufficient for testing.
     - Configure Security Group: Open ports 22 (SSH) and 8085 (application).

2. **Connect to Your EC2 Instance**:
   - After launching the instance, connect via SSH:
     ```sh
     ssh -i path_to_your_key.pem ec2-user@your_ec2_public_dns
     ```
   - For Ubuntu, replace `ec2-user` with `ubuntu`.

3. **Install Docker on EC2**:
   - Update packages and install Docker:
     ```sh
     sudo yum update -y
     sudo amazon-linux-extras install docker -y
     sudo service docker start
     sudo usermod -a -G docker ec2-user
     ```

   - For Ubuntu:
     ```sh
     sudo apt update
     sudo apt install docker.io -y
     sudo systemctl start docker
     sudo usermod -aG docker ubuntu
     ```

4. **Install AWS CLI on EC2**:
   ```sh
   sudo yum install aws-cli -y  # For Amazon Linux
   sudo apt install awscli -y  # For Ubuntu
   ```

#### 2. **Set Up Dockerfile and GitHub Actions Workflow**

1. **Ensure Dockerfile is Correct**:
   - Your `dockerfile.yml` looks correct for building a Java application with Maven.

2. **Review GitHub Actions Workflow**:
   - Your `backend.yml` workflow for building and pushing the Docker image to Amazon ECR is mostly correct. Ensure your secrets are configured in GitHub for AWS credentials and SSH keys.

3. **Add Secrets to GitHub**:
   - Go to your GitHub repository settings, then "Secrets and variables" -> "Actions".
   - Add the following secrets:
     - `AWS_ACCESS_KEY_ID`
     - `AWS_SECRET_ACCESS_KEY`
     - `PROD_HOST` (Your EC2 instance public DNS)
     - `PROD_USER` (Typically `ec2-user` or `ubuntu`)
     - `TESTPROD_SSH_KEY` (Your private SSH key content)

#### 3. **Build and Test Docker Image Locally**

1. **Build Docker Image Locally**:
   - Navigate to the directory containing your `dockerfile.yml` and run:
     ```sh
     docker build -f dockerfile.yml -t backend:latest .
     ```
   - This command builds the Docker image using the specified Dockerfile and tags it as `backend:latest`.

2. **Run Docker Container Locally**:
   - Run the Docker container to ensure it works:
     ```sh
     docker run -d -p 8085:8085 --name backend-test backend:latest
     ```
   - Access the application in your browser at `http://localhost:8085`.

3. **Stop and Remove Local Container**:
   ```sh
   docker stop backend-test
   docker rm backend-test
   ```

#### 4. **Set Up and Test GitHub Actions Workflow**

1. **Push Changes to GitHub**:
   - Make sure your `dockerfile.yml` and `.github/workflows/backend.yml` are committed and pushed to your repository.

2. **Create a Pull Request**:
   - Create a new branch, push your changes, and open a pull request to `main` (or `master`).
   - This will trigger the GitHub Actions workflow.

3. **Monitor Workflow in GitHub Actions**:
   - Go to the "Actions" tab in your repository to monitor the workflow run.
   - Ensure the workflow completes successfully, building and pushing the Docker image to Amazon ECR.

#### 5. **Deploy to EC2 Instance from GitHub Actions**

1. **Configure AWS CLI on EC2**:
   - Configure the AWS CLI with your IAM user credentials:
     ```sh
     aws configure
     ```

2. **Pull and Run Docker Image on EC2**:
   - The final step in your GitHub Actions workflow logs into ECR, pulls the latest image, and runs it:
     ```yaml
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

#### Technical Terms Explanation

1. **Docker Image**: A read-only template that contains the application code, runtime, libraries, and dependencies required to run your application.
2. **Docker Container**: An instance of a Docker image that runs as a separate process on your machine.
3. **ECR (Elastic Container Registry)**: An AWS service to store and manage Docker container images.
4. **CI/CD (Continuous Integration/Continuous Deployment)**: A practice that involves automatically building, testing, and deploying applications.
5. **SSH (Secure Shell)**: A protocol to securely connect to remote machines.

By following these steps, you can ensure your backend application is built, tested, and deployed to your AWS EC2 instance using Docker and GitHub Actions. If you have any further questions or run into issues, feel free to ask!

# should i still chose this despite my local machinerunning on a windows 11 OS?
Yes, you can still choose an Amazon Linux 2 or Ubuntu AMI for your EC2 instance even if your local machine is running Windows 11. The operating system of your local machine does not affect the operating system you choose for your EC2 instance. Here’s why:

1. **Platform Independence**: AWS EC2 instances are virtual machines running in the cloud. You can connect to them using SSH from any operating system, including Windows, MacOS, or Linux.

2. **SSH Clients on Windows**: Windows 11 includes a built-in SSH client, so you can easily connect to your EC2 instance using the Windows Terminal or PowerShell. Additionally, you can use tools like PuTTY for SSH connections.

3. **Docker and AWS CLI Compatibility**: Both Docker and AWS CLI are cross-platform and work on Windows, MacOS, and Linux. You can manage Docker images and interact with AWS services from your Windows machine without issues.

### Connecting to Your EC2 Instance from Windows 11

Here are the detailed steps to connect to your EC2 instance from a Windows 11 machine:

#### 1. Use Windows Terminal or PowerShell

1. **Open Windows Terminal or PowerShell**:
   - Press `Win + X` and select "Windows Terminal" or "Windows PowerShell".

2. **Navigate to Your Key File Location**:
   - Ensure your `.pem` file (SSH key) is in a known location, such as `C:\Users\YourUsername\`.

3. **Connect to the EC2 Instance**:
   - Use the `ssh` command to connect to your instance:
     ```sh
     ssh -i C:\path\to\your-key.pem ec2-user@your-ec2-public-dns
     ```
   - For Ubuntu instances, use `ubuntu` as the username:
     ```sh
     ssh -i C:\path\to\your-key.pem ubuntu@your-ec2-public-dns
     ```

#### 2. Use PuTTY (Alternative Method)

1. **Download and Install PuTTY**:
   - [PuTTY Download Page](https://www.putty.org/)

2. **Convert `.pem` to `.ppk` Using PuTTYgen**:
   - Open PuTTYgen and load your `.pem` file.
   - Save the private key as `.ppk`.

3. **Connect to EC2 Using PuTTY**:
   - Open PuTTY.
   - In the "Host Name" field, enter `ec2-user@your-ec2-public-dns` (or `ubuntu@your-ec2-public-dns` for Ubuntu).
   - Under "Connection > SSH > Auth", browse and select your `.ppk` file.
   - Click "Open" to connect.

### Step-by-Step Recap for EC2 Deployment

1. **Launch EC2 Instance**: Choose Amazon Linux 2 or Ubuntu.
2. **Install Docker and AWS CLI**: Follow the installation steps on your EC2 instance.
3. **Set Up Dockerfile and GitHub Actions**: Ensure your Dockerfile and workflow files are correct.
4. **Build and Test Locally**: Use Docker on your Windows machine to build and test the images.
5. **Push Changes to GitHub**: Commit and push your changes to trigger GitHub Actions.
6. **Monitor and Verify**: Ensure the workflow completes and the image is deployed to EC2.

By following these steps, you can efficiently manage and deploy your application using Docker, GitHub Actions, and AWS EC2, regardless of your local operating system. If you encounter any specific issues or have more questions, feel free to ask!
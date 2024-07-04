If you're unable to use the SSH key for automated deployment, you can manually deploy the Docker image to your AWS EC2 instance. Here are the steps you need to follow:

### Manual Deployment Steps

1. **Build the Docker Image Locally**
2. **Push the Docker Image to Amazon ECR**
3. **Log into Your EC2 Instance**
4. **Pull the Docker Image from Amazon ECR**
5. **Run the Docker Container on Your EC2 Instance**

### Detailed Instructions

#### 1. Build the Docker Image Locally

Navigate to your project directory where the `dockerfile.yml` is located and run the following command to build your Docker image:

```sh
docker build -f dockerfile.yml -t islam-center:latest .
```

#### 2. Push the Docker Image to Amazon ECR

First, log in to Amazon ECR:

```sh
aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin 687456531778.dkr.ecr.us-east-2.amazonaws.com
```

Tag the Docker image:

```sh
docker tag islam-center:latest 687456531778.dkr.ecr.us-east-2.amazonaws.com/islam-center:latest
```

Push the Docker image to your Amazon ECR repository:

```sh
docker push 687456531778.dkr.ecr.us-east-2.amazonaws.com/islam-center:latest
```

#### 3. Log into Your EC2 Instance

Use SSH to connect to your EC2 instance. Replace `ec2-user` with your username and `your-ec2-instance` with your EC2 instance's public DNS or IP address:

```sh
ssh -i /path/to/your/private-key.pem ec2-user@your-ec2-instance
```

#### 4. Pull the Docker Image from Amazon ECR

Once you're logged into your EC2 instance, log in to Amazon ECR again from within the EC2 instance:

```sh
aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin 687456531778.dkr.ecr.us-east-2.amazonaws.com
```

Pull the Docker image from Amazon ECR:

```sh
docker pull 687456531778.dkr.ecr.us-east-2.amazonaws.com/islam-center:latest
```

#### 5. Run the Docker Container on Your EC2 Instance

Stop and remove any existing container (if running):

```sh
docker container stop api
docker container rm api
```

Run the new Docker container:

```sh
docker run -d -p 8085:8085 -v /var/islamcenters:/var/islamcenters \
  --env-file /var/islamcenters/.env_api \
  --name api 687456531778.dkr.ecr.us-east-2.amazonaws.com/islam-center:latest
```

### Summary of Commands

Here’s a summary of all the commands you need to run:

1. **Build Docker Image Locally**:
   ```sh
   docker build -f dockerfile.yml -t islam-center:latest .
   ```

2. **Push Docker Image to Amazon ECR**:
   ```sh
   aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin 687456531778.dkr.ecr.us-east-2.amazonaws.com
   docker tag islam-center:latest 687456531778.dkr.ecr.us-east-2.amazonaws.com/islam-center:latest
   docker push 687456531778.dkr.ecr.us-east-2.amazonaws.com/islam-center:latest
   ```

3. **Log into EC2 Instance**:
   ```sh
   ssh -i /path/to/your/private-key.pem ec2-user@your-ec2-instance
   ```

4. **Pull Docker Image on EC2 Instance**:
   ```sh
   aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin 687456531778.dkr.ecr.us-east-2.amazonaws.com
   docker pull 687456531778.dkr.ecr.us-east-2.amazonaws.com/islam-center:latest
   ```

5. **Run Docker Container on EC2 Instance**:
   ```sh
   docker container stop api
   docker container rm api
   docker run -d -p 8085:8085 -v /var/islamcenters:/var/islamcenters \
     --env-file /var/islamcenters/.env_api \
     --name api 687456531778.dkr.ecr.us-east-2.amazonaws.com/islam-center:latest
   ```

### Notes

- Make sure the security groups for your EC2 instance allow inbound traffic on port `8085` if you want to access your application publicly.
- Ensure that your environment file `/var/islamcenters/.env_api` is correctly set up on your EC2 instance.

By following these steps, you can manually deploy your Docker application to your AWS EC2 instance. If you have any further questions or need more assistance, feel free to ask!
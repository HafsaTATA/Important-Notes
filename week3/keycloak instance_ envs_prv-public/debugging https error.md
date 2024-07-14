Based on the logs, there seems to be an issue with Keycloak resolving the HTTP request authority, which likely relates to the configuration of your container and the way Keycloak is handling the URLs. Here’s a simplified explanation of what you need to do and why:

### Step-by-Step Guide to Fix the Issue and Set Up SSL with Let's Encrypt

#### Understanding the Current Problem
1. **HTTP to HTTPS Issue**: Your Keycloak instance is not accessible over HTTP or HTTPS correctly. This might be because Keycloak is trying to redirect to `localhost` instead of your actual public IP or domain.
2. **SSL Configuration**: SSL ensures secure communication. Let’s Encrypt provides free SSL certificates which can help in setting up HTTPS.

### Step 1: Correct Keycloak Configuration

1. **Environment Variables for Keycloak**:
   Keycloak needs to know the correct hostname to use for redirects. You can set this using environment variables when you run your Docker container.

   ```bash
   sudo docker run -d --name mykeycloak -p 80:8080 -p 443:8443 \
       -e KEYCLOAK_ADMIN=admin -e KEYCLOAK_ADMIN_PASSWORD=change_me \
       -e KC_HOSTNAME=3.15.155.101 \
       -e KC_HTTPS_PORT=443 \
       687456531778.dkr.ecr.us-east-2.amazonaws.com/islam-center-front:latest
   ```

   - `KC_HOSTNAME` sets the public hostname for Keycloak.
   - `KC_HTTPS_PORT` tells Keycloak to use port 443 for HTTPS.

### Step 2: Installing Let's Encrypt and Configuring SSL

1. **Install Certbot on Your EC2 Instance**:
   ```bash
   sudo apt-get update
   sudo apt-get install certbot
   ```

2. **Obtain the SSL Certificate**:
   Run Certbot to obtain a certificate for your public IP (or domain if you have one).

   ```bash
   sudo certbot certonly --standalone -d your_domain_or_public_ip
   ```

3. **Mount the Certificates in Your Docker Container**:
   Once you have obtained the certificates, you need to mount them in your Docker container. 

   First, create directories for the certificates:
   ```bash
   sudo mkdir -p /etc/letsencrypt/live/your_domain_or_public_ip
   ```

   Then, run the Docker container with the SSL certificates mounted:
   ```bash
   sudo docker run -d --name mykeycloak -p 80:8080 -p 443:8443 \
       -e KEYCLOAK_ADMIN=admin -e KEYCLOAK_ADMIN_PASSWORD=change_me \
       -e KC_HOSTNAME=3.15.155.101 \
       -e KC_HTTPS_PORT=443 \
       -v /etc/letsencrypt/live/your_domain_or_public_ip/fullchain.pem:/etc/x509/https/tls.crt \
       -v /etc/letsencrypt/live/your_domain_or_public_ip/privkey.pem:/etc/x509/https/tls.key \
       687456531778.dkr.ecr.us-east-2.amazonaws.com/islam-center-front:latest
   ```

### Step 3: Verify Access and Troubleshoot

1. **Access Keycloak Over HTTPS**:
   Open your browser and navigate to `https://3.15.155.101`. If everything is set up correctly, you should see the Keycloak login page.

2. **Check Docker Logs**:
   If you still face issues, check the logs for more detailed errors:
   ```bash
   sudo docker logs mykeycloak
   ```

### Real-Life Example:

1. **Problem**: Your website (house) has no security, and visitors (users) can’t securely communicate.
2. **Solution**: Install a security system (SSL) using Let's Encrypt, ensuring all communications are secure.
3. **Implementation**: Adjust Keycloak's configuration to recognize its public address and use the obtained SSL certificates to secure the communication.

### Summary:

1. **Set Correct Environment Variables**: Ensure Keycloak knows its public hostname and ports.
2. **Install Certbot**: Get free SSL certificates from Let’s Encrypt.
3. **Run Docker with SSL**: Use the obtained certificates to enable HTTPS on your Keycloak container.
4. **Verify and Troubleshoot**: Ensure everything is set up correctly by accessing your Keycloak instance over HTTPS.

By following these steps, you should be able to resolve the access issues and secure your Keycloak instance with HTTPS.
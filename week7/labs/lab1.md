# Lab: Artifacts Management with Nexus OSS on Ubuntu

This lab is demonstrating artifact management using Nexus Repository OSS in a Docker container on Ubuntu. By the end of this lab, students will have installed Docker, configured Nexus Repository, created repositories, and managed Docker images using Nexus OSS.

---

## Step 1: Install Docker on Ubuntu

### Instructions:

1. **Update the System**
   - Run the following commands to update your package information:
   ```bash
   sudo apt update
   sudo apt upgrade -y
   ```

2. **Install Prerequisites**
   - Install the required packages:
   ```bash
   sudo apt install apt-transport-https ca-certificates curl software-properties-common -y
   ```

3. **Add Docker's GPG Key and Repository**
   - Add Docker's GPG key:
   ```bash
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
   ```
   - Add the Docker repository:
   ```bash
   sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
   ```

4. **Install Docker**
   - Install Docker:
   ```bash
   sudo apt update
   sudo apt install docker-ce -y
   ```

5. **Verify Docker Installation**
   - Check Docker status to verify installation:
   ```bash
   sudo systemctl status docker
   ```

---

## Step 2: Configure Docker to Trust the Nexus Registry

### Instructions:

1. **Configure Docker Daemon**
   - On the Ubuntu machine, update the Docker daemon to trust the insecure Nexus registry:
   ```bash
   sudo nano /etc/docker/daemon.json
   ```
   - Add the following configuration:
   ```json
   {
       "insecure-registries" : ["<host-ip>:8082"]
   }
   ```

2. **Restart Docker**
   - Restart Docker to apply the changes:
   ```bash
   sudo systemctl restart docker
   ```

3. **Verify Configuration**
   - Run a simple Docker command to verify the daemon configuration:
   ```bash
   docker info | grep -i insecure
   ```

---

## Step 3: Create a Persistent Volume for Nexus

### Instructions:

1. **Create a Docker Volume**
   - Run the following command to create a persistent volume named `nexus-data`:
   ```bash
   docker volume create nexus-data
   ```

---

## Step 3: Run Nexus OSS Container

### Instructions:

1. **Run the Nexus OSS Container**
   - Use the following command to run a Nexus OSS container with the volume mounted and ports 8081 and 8082 exposed and mapped to the host:
   ```bash
   docker run -d -p 8081:8081 -p 8082:8082 --name nexus -v nexus-data:/nexus-data sonatype/nexus3
   ```

---

## Step 4: Access Nexus and Retrieve Admin Password

### Instructions:

1. **Access the Nexus Admin Password**
   - Open a shell into the Nexus container:
   ```bash
   docker exec -it nexus /bin/bash
   ```

2. **Locate the Password**
   - Navigate to the configuration folder and find the admin password:
   ```bash
   cat /nexus-data/admin.password
   ```

3. **Login and Change Password**
   - Access Nexus through your web browser at `http://<host-ip>:8081`.
   - Log in with the username `admin` and the retrieved password.
   - Change the password as prompted.

---

## Step 5: Create a Blob Store for Docker Images

### Instructions:

1. **Navigate to Blob Stores**
   - In the Nexus UI, go to **Administration > Repository > Blob Stores**.

2. **Create a New Blob Store**
   - Click **Create Blob Store**.
   - Select **Type: File**.
   - Set the name to `docker-images`.
   - Click **Create Blob Store**.

---

## Step 6: Create a Docker-Hosted Repository

### Instructions:

1. **Create a Docker-Hosted Repository**
   - Go to **Administration > Repository > Repositories**.
   - Click **Create repository** and select **docker (hosted)**.
   - Set **Name** to `docker-hosted`.
   - Use the blob store `docker-images` created earlier.
   - Enable HTTP and set **HTTP Port** to `8082`.
   - Click **Create repository**.

---

## Step 7: Login to Docker Registry

### Instructions:

1. **Login to Docker Registry**
   - Use Docker CLI to login to the newly created Nexus Docker registry:
   ```bash
   docker login -u admin -p <new-password> http://<host-ip>:8082
   ```

---

## Step 8: Push an Image to Docker-Hosted Repository

### Instructions:

1. **Download Nginx Image from DockerHub**
   - Pull the Nginx image from DockerHub:
   ```bash
   docker pull nginx
   ```

2. **Tag the Image**
   - Tag the Nginx image for pushing to the Nexus repository:
   ```bash
   docker tag nginx <host-ip>:8082/docker-hosted/nginx
   ```

3. **Push the Image**
   - Push the tagged image to Nexus:
   ```bash
   docker push <host-ip>:8082/docker-hosted/nginx
   ```

---

## Step 9: Create a Blob Store for Docker-Proxy

### Instructions:

1. **Navigate to Blob Stores**
   - In the Nexus UI, go to **Administration > Repository > Blob Stores**.

2. **Create a New Blob Store**
   - Click **Create Blob Store**.
   - Select **Type: File**.
   - Set the name to `docker-proxy`.
   - Click **Create Blob Store**.

---

## Step 10: Create a Docker-Proxy Repository

### Instructions:

1. **Create a Docker-Proxy Repository**
   - Go to **Administration > Repository > Repositories**.
   - Click **Create repository** and select **docker (proxy)**.
   - Set **Name** to `docker-proxy`.
   - Use the blob store `docker-proxy` created earlier.
   - Set the **Remote storage** to `https://registry-1.docker.io`.
   - Enable HTTP and set **HTTP Port** to `8082`.
   - Click **Create repository**.

---

## Step 11: Demonstrate Docker Pull Using Docker-Proxy Repository

### Instructions:

1. **Pull an Image Using Docker-Proxy**
   - Use Docker CLI to pull an image through the docker-proxy repository:
   ```bash
   docker pull <host-ip>:8082/docker-proxy/nginx
   ```

2. **Verify Docker-Proxy Functionality**
   - Ensure the image is pulled successfully through the `docker-proxy` repository, which acts as a proxy to DockerHub.

---

## Step 12: Create a Docker-Group Repository

### Instructions:

1. **Create a Docker-Group Repository**
   - Go to **Administration > Repository > Repositories**.
   - Click **Create repository** and select **docker (group)**.
   - Set **Name** to `docker-group`.
   - Add `docker-hosted` and `docker-proxy` to the **Group members**.
   - Enable HTTP and set **HTTP Port** to `8082` (reusing the existing port).
   - Click **Create repository**.

---

## Step 13: Demonstrate the Use of Docker-Group Repository

### Instructions:

1. **Pull an Image Using Docker-Group**
   - Use Docker CLI to pull an image through the docker-group repository:
   ```bash
   docker pull <host-ip>:8082/docker-group/nginx
   ```

2. **Tag and Push an Image Using Docker-Group**
   - Tag the Nginx image for pushing to the Nexus docker-group repository:
   ```bash
   docker tag nginx <host-ip>:8082/docker-group/nginx
   ```
   - Push the tagged image to Nexus:
   ```bash
   docker push <host-ip>:8082/docker-group/nginx
   ```

3. **Verify Docker-Group Functionality**
   - Ensure that both pulling and pushing operations succeed using the docker-group repository, which combines both hosted and proxy repositories.

---

## Step 14: Pull an Image from Docker-Proxy

### Instructions:

1. **Pull an Image from Docker-Proxy**
   - Use Docker CLI to pull an image through the proxy repository:
   ```bash
   docker pull <host-ip>:8082/docker-proxy/nginx
   ```

2. **Disable Docker-Hosted Repository Access**
   - In the Nexus UI, go to **Administration > Repository > Repositories**.
   - Disable the `docker-hosted` repository to ensure the proxy repository is used.

3. **Verify Pull from Proxy**
   - Ensure the image is pulled successfully through the `docker-proxy` repository, which acts as a proxy to DockerHub.

---

## Step 15: Configure the Private Docker Registry

### Instructions:

1. **Configure Docker Daemon**
   - On the Ubuntu machine, update the Docker daemon to trust the insecure registry:
   ```bash
   sudo nano /etc/docker/daemon.json
   ```
   - Add the following configuration:
   ```json
   {
       "insecure-registries" : ["<host-ip>:8082"]
   }
   ```

2. **Restart Docker**
   - Restart Docker to apply the changes:
   ```bash
   sudo systemctl restart docker
   ```

3. **Verify Configuration**
   - Pull or push an image to verify that the Docker daemon is correctly configured to trust the private registry.


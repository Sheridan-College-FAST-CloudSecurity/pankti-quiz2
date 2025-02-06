# **Lab: Securing Jenkins with SSL on Amazon EC2**

This lab guides students through configuring SSL for Jenkins hosted on an Amazon EC2 instance. Follow the detailed instructions below to secure Jenkins using HTTPS.

---

## **Step 1: Generate Self-Signed SSL Certificates**

### **Explanation:**

Self-signed certificates are useful for internal or testing purposes. You will create these using OpenSSL.

### **Steps:**

1. **Generate a Private Key:**

   ```bash
   openssl genrsa -out jenkins.key 2048
   ```

2. **Generate a Self-Signed Certificate:**

   ```bash
   openssl req -new -x509 -key jenkins.key -out jenkins.crt -days 365
   ```

   - Follow the prompts to add details such as Country, State, Organization, etc.

3. **Verify the Certificate and Key:**

   - Confirm the files `jenkins.key` and `jenkins.crt` are created.

---

## **Step 2: Create a PKCS12 File**

### **Explanation:**

The PKCS12 file bundles the certificate and private key into a format compatible with Java keystores.

### **Steps:**

1. **Convert the Certificate and Key to PKCS12 Format:**
   ```bash
   openssl pkcs12 -export -in jenkins.crt -inkey jenkins.key -out jenkins.p12 -name jenkins
   ```
   - You will be prompted to set an export password for the `.p12` file. Remember this password for later steps.

---

## **Step 3: Convert PKCS12 to JKS Format**

### **Explanation:**

Jenkins uses Java keystores (JKS) for SSL, so we must convert the PKCS12 file to JKS format.

### **Steps:**

1. **Use the ****`keytool`**** Utility:**
   ```bash
   keytool -importkeystore -srckeystore jenkins.p12 -srcstoretype pkcs12 -destkeystore jenkins.jks -deststoretype jks
   ```
   - Provide the password for the `.p12` file when prompted.
   - Set a new password for the JKS file. Ensure it is secure and remember it.

---

## **Step 4: Configure Jenkins to Use SSL**

### **Steps:**

1. **Move the JKS File to Jenkins Directory:**

   ```bash
   sudo mv jenkins.jks /var/lib/jenkins/
   ```

2. **Update Jenkins Configuration:**

   - Open the Jenkins configuration file:
     ```bash
     sudo nano /etc/default/jenkins
     ```
   - Add the following Java arguments:
     ```bash
     JENKINS_ARGS="--httpPort=-1 --httpsPort=8443 --httpsKeyStore=/var/lib/jenkins/jenkins.jks --httpsKeyStorePassword=<your-password>"
     ```
   - Replace `<your-password>` with the JKS file password you set earlier.

3. **Restart Jenkins:**

   ```bash
   sudo systemctl restart jenkins
   ```

---

## **Step 5: Disable HTTP and Validate SSL**

### **Steps:**

1. **Ensure HTTP is Disabled:**

   - Confirm `--httpPort=-1` is set in the Jenkins configuration to disable HTTP.

2. **Access Jenkins Over HTTPS:**

   - Open a browser and navigate to:
     ```
     https://<your-ec2-public-ip>:8443
     ```

3. **Validate the SSL Certificate:**

   - Check the browser padlock to confirm secure communication.
   - For self-signed certificates, you may need to bypass the security warning for testing purposes.

---

## **Summary of Key Concepts**

1. **SSL Certificates:** Ensure encrypted communication between the server and clients.
2. **PKCS12 Format:** Combines private key and certificate for secure transport.
3. **JKS Format:** Java's native keystore format required for Jenkins SSL configuration.
4. **HTTPS Configuration:** Secures Jenkins by disabling HTTP and enabling SSL/TLS.

https\://devopscube.com/configure-ssl-jenkins/


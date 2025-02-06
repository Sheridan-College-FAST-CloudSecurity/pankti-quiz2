# Lab Instructions: Setting Up Jenkins in a Codespace

## Step 1: Start a New Codespace

### Navigate to Your Repository

1. **Open the Repository**: Open the repository where the Jenkins setup script is located.

2. **Open the Codespace Menu**:

   - On the repository's landing page, click the **Code** button.

3. **Create a New Codespace**:

   - Select the **Codespaces** tab from the dropdown menu.
   - Click **Create codespaces on main** to create a new Codespace environment.

4. **Wait for Initialization**:

   - After a few moments, the Codespace will initialize and open.
   - A terminal should automatically appear within the Codespace.

## Step 2: Execute the Jenkins Setup Script

### Run the Bootstrap Script

- In the Codespace terminal, run the following command to start the Jenkins server:

  [Jenkins Setup Script on GitHub](https://github.com/actions/importer-labs/blob/main/jenkins/bootstrap/setup.sh)
  ```bash
  ./jenkins/bootstrap/setup.sh
  ```

### Wait for Jenkins to Start

- The script will set up and start a Docker container running Jenkins.
- This process may take a few minutes.

### Access the Jenkins Server

- After the server is running, you will see a pop-up box with a URL. Click the link to open Jenkins in your browser.
- Alternatively, go to the **Ports** tab in the Codespace terminal:
  - Find the **Local Address** URL for the Jenkins port.
  - Right-click the URL and select **Open in Browser**.

## Step 3: Authenticate into Jenkins

### Open the Jenkins URL in Your Browser

- Use the link provided in the pop-up or the **Ports** tab.

### Log in to Jenkins

- Enter the following credentials:
  - **Username**: `admin`
  - **Password**: `password`

### Verify the Jenkins Dashboard

- Once logged in, you will see the Jenkins dashboard with a few predefined pipelines.

## Step 4: Create a New Jenkins Pipeline

### Task: Set up a Simple Pipeline that Prints "Hello World!"

### 4.1. Create a New Pipeline Job

1. From the Jenkins dashboard, click **New Item** in the left-hand menu.
2. Enter a name for your pipeline (e.g., `HelloWorldPipeline`).
3. Select **Pipeline** as the project type and click **OK**.

### 4.2. Configure the Pipeline

1. Scroll down to the **Pipeline** section on the configuration page.
2. Select **Pipeline Script**.
3. Enter the following Groovy code in the script editor:
   ```groovy
   pipeline {
       agent any
       stages {
           stage('Hello') {
               steps {
                   echo 'Hello, World!'
               }
           }
       }
   }
   ```

#### Explanation of the Code

- `pipeline {}`: Defines the overall structure of the pipeline.
- `agent any`: Specifies that the pipeline will run on any available Jenkins agent.
- `stage('Hello')`: Creates a single stage named "Hello".
- `steps { echo 'Hello, World!' }`: Executes the command to print "Hello, World!" in the console output.

4. Scroll to the bottom of the page and click **Save**.

### 4.3. Run the Pipeline

1. From the pipeline's page, click **Build Now** in the left-hand menu.
2. Observe the build progress in the **Build History** section.
3. Click on the build number (e.g., `#1`) to view the build details.
4. Go to **Console Output** to see the "Hello, World!" message printed.

## Step 5: Verify and Clean Up

- Verify that the pipeline is completed successfully (indicated by a green checkmark).

- Run the following command to stop the Jenkins container:

  ```bash
  docker stop jenkins
  ```


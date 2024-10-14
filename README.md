
# **Deploying a Static Website to Azure Storage via Azure DevOps**

This guide provides a step-by-step process to:

## This is under update, and currently being tested and verified

1. **Create and configure an Azure Storage Account** with Static Website hosting enabled.
2. **Create a Service Principal** (App Registration) in Microsoft Entra ID (formerly Azure Active Directory).
3. **Assign appropriate roles** to the Service Principal for accessing the storage account.
4. **Set up a Service Connection** in Azure DevOps using the Service Principal.
5. **Create a Release Pipeline** in Azure DevOps to deploy your static website.

---

## **Prerequisites**

- **Azure Subscription**: Access to an Azure subscription with permissions to create resources.
- **Azure DevOps Account**: An Azure DevOps organization and project.
- **HTML Content**: Your static website files (e.g., `index.html`, `404.html`) stored in a repository connected to Azure DevOps.
- Alternatively, You can also use a Build pipeline to create HTML page. ex. Using mkdocs-material (python) to build md into html and then upload them as pipeline artifact. **Refer to Part 7**

---

## **Part 1: Create an Azure Storage Account with Static Website Enabled**

### **Step 1: Create an Azure Storage Account**

1. **Sign in to Azure Portal**:
   - Go to [https://portal.azure.com](https://portal.azure.com) and sign in.

2. **Create a New Storage Account**:
   - Click on **"Create a resource"**.
   - Search for **"Storage account"** and select it.
   - Click **"Create"**.

3. **Configure Storage Account Settings**:

   - **Basics Tab**:
     - **Subscription**: Select your subscription.
     - **Resource Group**: Choose an existing one or create a new one.
     - **Storage Account Name**: Enter a unique name (e.g., `uniquestorageacctname`).
     - **Region**: Choose your preferred region.
     - **Performance**: **Standard**.
     - **Redundancy**: **Locally-redundant storage (LRS)**.

   - **Advanced Tab**:
     - **Primary Service**: **Azure Blob Storage** or **Azure Data Lake Storage Gen2**.
     - **Primary Workload Type**: **Other**.

   - **Review + Create**:
     - Click **"Review + create"**.
     - Ensure validation passes and click **"Create"**.

### **Step 2: Enable Static Website Hosting**

1. **Navigate to the Storage Account**:
   - Once deployment is complete, click **"Go to resource"**.

2. **Enable Static Website**:

   - In the left-hand menu under **"Data management"**, click **"Static website"**.
   - Toggle **"Static website"** to **"Enabled"**.
   - **Index document name**: Enter `index.html`.
   - **Error document path**: Enter `404.html`.
   - Click **"Save"**.
   - **Note**: After saving, copy the **Primary endpoint URL** displayed. This is where your static website will be accessible.

---

## **Part 2: Create a Service Principal in Microsoft Entra ID**

### **Step 1: Access Microsoft Entra ID**

1. **Navigate to Microsoft Entra ID**:
   - In the Azure Portal, click on **"Azure Active Directory"** (now known as **Microsoft Entra ID**).

### **Step 2: Create an App Registration**

1. **Create a New App Registration**:

   - In the left-hand menu, click on **"App registrations"**.
   - Click **"New registration"**.

2. **Configure the App Registration**:

   - **Name**: Enter `StaticWebsiteApp`.
   - **Supported account types**: Choose **"Accounts in this organizational directory only (Default Directory only - Single tenant)"**.
   - **Redirect URI**: Leave blank.
   - Click **"Register"**.

### **Step 3: Create a Client Secret**

1. **Navigate to Certificates & Secrets**:

   - In the left-hand menu of your App Registration, click **"Certificates & secrets"**.

2. **Add a New Client Secret**:

   - Under **"Client secrets"**, click **"New client secret"**.
   - **Description**: Enter a name (e.g., `StaticWebsiteSecret`).
   - **Expires**: Select **"24 months"** (730 days).
   - Click **"Add"**.
   - **Copy the Value** immediately after creation. **You will not be able to retrieve it later**.

### **Step 4: Note Down Important IDs**

- **Application (client) ID**: Found on the App Registration's **Overview** page.
- **Directory (tenant) ID**: Also on the **Overview** page.
- Store these IDs securely along with the Client Secret Value.

---

## **Part 3: Assign Roles to the Service Principal**

### **Step 1: Assign Roles at the Storage Account Level**

1. **Navigate to Your Storage Account**:

   - Go back to your storage account in the Azure Portal.

2. **Access Control (IAM)**:

   - Click on **"Access control (IAM)"** in the left-hand menu.

3. **Add Role Assignment**:

   - Click **"Add"** and select **"Add role assignment"**.

4. **Assign "Storage Blob Data Contributor" Role**:

   - **Role**: Search for and select **"Storage Blob Data Contributor"**.
   - **Assign Access to**: **"User, group, or service principal"**.
   - **Members**: Click **"Select members"**, search for `StaticWebsiteApp`, select it, and click **"Select"**.
   - Click **"Review + assign"** and then **"Assign"**.

5. **Assign "Contributor" Role** (Optional but recommended):

   - Repeat the above steps to add the **"Contributor"** role to `StaticWebsiteApp`.
   - This role allows management of the storage account, including configurations.

### **Step 2: Assign "Reader" Role at the Subscription Level**

1. **Navigate to Your Subscription**:

   - Click on **"Subscriptions"** in the Azure Portal.
   - Select your subscription.

2. **Access Control (IAM)**:

   - Click on **"Access control (IAM)"**.

3. **Add Role Assignment**:

   - Click **"Add"** and select **"Add role assignment"**.

4. **Assign "Reader" Role**:

   - **Role**: Search for and select **"Reader"**.
   - **Assign Access to**: **"User, group, or service principal"**.
   - **Members**: Select `StaticWebsiteApp` as before.
   - Click **"Review + assign"** and then **"Assign"**.

---

## **Part 4: Create a Service Connection in Azure DevOps**

### **Step 1: Access Azure DevOps Project Settings**

1. **Sign in to Azure DevOps**:

   - Go to [https://dev.azure.com](https://dev.azure.com) and sign in.

2. **Navigate to Project Settings**:

   - Select your **organization** and **project**.
   - Click on **"Project settings"** (gear icon) in the lower-left corner.

### **Step 2: Create a New Service Connection**

1. **Access Service Connections**:

   - Under **"Pipelines"**, click on **"Service connections"**.

2. **Add New Service Connection**:

   - Click **"New service connection"**.
   - Select **"Azure Resource Manager"** and click **"Next"**.

3. **Configure the Service Connection**:

   - **Authentication Method**: Choose **"Service principal (manual)"**.
   - Click **"Next"**.

4. **Enter Service Connection Details**:

   - **Environment**: **"AzureCloud"**.
   - **Scope Level**: **"Subscription"**.
   - **Subscription ID**: Enter your Azure Subscription ID.
   - **Subscription Name**: Enter a name (e.g., `scStaticWebsiteApp`).
   - **Service Principal ID**: Paste the **Application (client) ID** from earlier.
   - **Service Principal Key**: Paste the **Client Secret Value**.
   - **Tenant ID**: Paste the **Directory (tenant) ID**.
   - **Service Connection Name**: Enter `scConnStaticWebsiteApp`.
   - **Grant access permission to all pipelines**: Check this box.

5. **Verify and Save**:

   - Click **"Verify"** to ensure the connection is valid.
   - If verification succeeds, click **"Verify and save"**.

---

## **Part 5: Create a Release Pipeline in Azure DevOps**

### **Step 1: Create a New Release Pipeline**

1. **Navigate to Pipelines**:

   - In Azure DevOps, click on **"Pipelines"** and then **"Releases"**.

2. **Create a New Pipeline**:

   - Click **"New pipeline"**.

3. **Select a Template**:

   - Choose **"Empty job"**.

### **Step 2: Add an Artifact**

1. **Add Source Artifact**:

   - Click **"Add an artifact"**.
   - **Source type**: Select the source of your static website files (e.g., your build pipeline named `mkdocs`).
   - **Default version**: **"Latest"**.
   - **Source alias**: Enter `_mkdocs`.
   - Click **"Add"**.

### **Step 3: Configure the Deployment Stage**

1. **Rename the Stage** (Optional):

   - Click on **"Stage 1"** and rename it to **"Deploy to Static Website"**.

2. **Add Tasks to the Stage**:

   - Click on the **"1 job, 0 tasks"** link under the stage.

3. **Add Azure File Copy Task**:

   - Click the **"+"** icon next to **"Agent job"**.
   - Search for **"Azure File Copy"**.
   - Click **"Add"** to include it in the pipeline.

### **Step 4: Configure the Azure File Copy Task**

1. **Select the Task**:

   - Click on the **"Azure File Copy"** task to configure it.

2. **Set Task Details**:

   - **Display Name**: Enter `File Copy Task for MkDocs`.

3. **Configure Source Settings**:

   - **Source**:

     - Click the **"..."** (ellipsis) button next to the **"Source"** field.
     - Navigate through the artifacts to select the folder containing your static website files (e.g., a folder named `site` generated by MkDocs).
     - Select the folder and confirm.

4. **Configure Destination Settings**:

   - **Azure Subscription**: Select the service connection `scConnStaticWebsiteApp` created earlier.
   - **Destination Type**: **"Azure Blob"**.
   - **RM Storage Account**: Enter the name of your storage account.
   - **Container Name**: Enter `$web` (the special container for static websites).
   - **Blob Prefix**: Leave blank unless you want to deploy to a subdirectory.

5. **Additional Options** (Optional):

   - **Overwrite**: Ensure the **"Overwrite"** option is checked if you want to replace existing files.
   - **Recursive**: Check **"Recursive"** to include all subfolders and files.

6. **Save the Pipeline**:

   - Click **"Save"** at the top right corner.
   - Provide a commit message if prompted.

### **Step 5: Create and Deploy a Release**

1. **Create a Release**:

   - Click **"Create release"**.
   - Ensure the correct artifact version is selected.
   - Click **"Create"**.

2. **Monitor the Deployment**:

   - Click on the release number to view its progress.
   - Check the logs for the **"Azure File Copy"** task to ensure files are copied successfully.

---

## **Part 6: Verify the Deployment**

### **Step 1: Access the Static Website**

- Open a web browser and navigate to the **Primary endpoint** URL you copied earlier.
- Your static website should be displayed.

### **Step 2: Troubleshoot if Necessary**

- **Deployment Failures**:

  - Check the logs in the release pipeline for error messages.
  - Ensure the Service Principal has the necessary permissions.

- **Website Not Displaying Correctly**:

  - Confirm that the **Index document name** matches the main HTML file.
  - Verify that all files are correctly uploaded to the `$web` container.

---

## **Part 7: Create a Build Pipeline in Azure DevOps**

In this part, we'll set up a build pipeline in Azure DevOps to generate the HTML files for your static website using MkDocs with the Material theme.

### **Prerequisites**

- **Python**: Ensure that your project requires Python (version 3.12 or your desired version).
- **MkDocs and MkDocs-Material**: Your `requirements.txt` should include `mkdocs` and `mkdocs-material` to build the site.

### **Step 1: Prepare Your Repository**

1. **Add a `requirements.txt` File**:
   - In your repository, create a `requirements.txt` file with the following content:
     ```plaintext
     mkdocs
     mkdocs-material
     ```
   - Add any other Python dependencies your project requires.

2. **Add an `mkdocs.yml` Configuration File**:
   - Ensure you have an `mkdocs.yml` file at the root of your repository configured for your site.

### **Step 2: Create the Azure Pipelines YAML File**

Add the following YAML code to your repository as `azure-pipelines.yml`:

```yaml
# azure-pipelines.yml

trigger:
  - main  # Or your desired branch name

pool:
  vmImage: 'ubuntu-latest'

jobs:
- job: BuildAndPublishMkDocs
  displayName: 'Build and Publish MkDocs Site'
  steps:

    - checkout: self
      displayName: 'Checkout Code'

    - task: UsePythonVersion@0
      inputs:
        versionSpec: '3.12'  # Use the desired Python version
      displayName: 'Set up Python Environment'

    - script: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
      displayName: 'Install Dependencies'

    - script: |
        mkdocs build
      displayName: 'Build MkDocs Site'

    - task: CopyFiles@2
      inputs:
        contents: 'site/**'
        targetFolder: '$(Build.ArtifactStagingDirectory)/site'
      displayName: 'Copy MkDocs Site to Staging Directory'

    - task: PublishBuildArtifacts@1
      inputs:
        PathtoPublish: '$(Build.ArtifactStagingDirectory)/site'
        ArtifactName: 'MkDocsSite'
        publishLocation: 'Container'
      displayName: 'Publish MkDocs Site as Artifact'

```

# **Appendix**

## **Complete Step-by-Step Commands and Actions**

### **1. Create an Azure Storage Account**

- **Azure Portal** ➔ **Create a resource** ➔ **Storage account** ➔ **Create**
- **Basics**:
  - **Storage Account Name**: `[your_unique_name]`
  - **Region**: `[your_preferred_region]`
  - **Performance**: `Standard`
  - **Redundancy**: `LRS`
- **Review + Create** ➔ **Create**

### **2. Enable Static Website Hosting**

- **Storage Account** ➔ **Static website** ➔ **Enabled**
- **Index document name**: `index.html`
- **Error document path**: `404.html`
- **Save**

### **3. Create a Service Principal (App Registration)**

- **Azure Active Directory** ➔ **App registrations** ➔ **New registration**
- **Name**: `StaticWebsiteApp`
- **Register**
- **Certificates & secrets** ➔ **New client secret**
  - **Description**: `StaticWebsiteSecret`
  - **Expires**: `730 days`
- **Add**
- **Copy** the **Client Secret Value**
- **Note down**:
  - **Application (client) ID**
  - **Directory (tenant) ID**

### **4. Assign Roles to the Service Principal**

- **Storage Account** ➔ **Access control (IAM)** ➔ **Add role assignment**
  - **Role**: `Storage Blob Data Contributor`
  - **Assign access to**: `User, group, or service principal`
  - **Select members**: `StaticWebsiteApp`
  - **Review + assign**
- **Repeat** for role `Contributor` (optional)
- **Subscription** ➔ **Access control (IAM)** ➔ **Add role assignment**
  - **Role**: `Reader`
  - **Assign access to**: `User, group, or service principal`
  - **Select members**: `StaticWebsiteApp`
  - **Review + assign`

### **5. Create a Service Connection in Azure DevOps**

- **Azure DevOps** ➔ **Project settings** ➔ **Service connections**
- **New service connection** ➔ **Azure Resource Manager** ➔ **Next**
- **Service principal (manual)** ➔ **Next**
- **Enter details**:
  - **Subscription ID**: `[your_subscription_id]`
  - **Subscription Name**: `scStaticWebsiteApp`
  - **Service Principal ID**: `[Application (client) ID]`
  - **Service Principal Key**: `[Client Secret Value]`
  - **Tenant ID**: `[Directory (tenant) ID]`
  - **Service Connection Name**: `scConnStaticWebsiteApp`
- **Verify and save**

### **6. Create a Release Pipeline in Azure DevOps**

- **Azure DevOps** ➔ **Pipelines** ➔ **Releases** ➔ **New pipeline**
- **Empty job**
- **Add an artifact**:
  - **Source type**: `[your_build_pipeline]`
  - **Source alias**: `_mkdocs`
- **Stage 1** ➔ **1 job, 0 tasks**
- **Agent job** ➔ **+** ➔ **Azure File Copy** ➔ **Add**
- **Configure Azure File Copy Task**:
  - **Display Name**: `File Copy Task for MkDocs`
  - **Source**: `[select your site folder]`
  - **Azure Subscription**: `scConnStaticWebsiteApp`
  - **Destination Type**: `Azure Blob`
  - **RM Storage Account**: `[your_storage_account_name]`
  - **Container Name**: `$web`
- **Save**

### **7. Deploy the Release**

- **Create release** ➔ **Create**
- **Monitor deployment logs**

---

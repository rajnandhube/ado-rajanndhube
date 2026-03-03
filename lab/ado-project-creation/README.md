# 🚀 Azure DevOps Project Provisioning Automation

This documentation provides the operational guide for the Automated Governance Pipeline used to create and configure new Azure DevOps Projects.

---

## 🏗 Step-by-Step Execution Guide

### Step 01: Security & Authentication Setup
Before running the pipeline, you must establish a secure connection to the Azure DevOps API.
1.  **Generate PAT:** Create a Personal Access Token (PAT) for a user with **Project Collection Administrator** rights.
    * **Required Scopes:** `Project and Team (Read, Write, & Manage)`, `Graph (Read & Manage)`, and `Member Entitlement Management (Read)`.
2.  **Create Variable Group:** * Navigate to **Pipelines > Library**.
    * Create a group named `project-vg`.
    * Add a variable `AZURE_DEVOPS_PAT` and paste your token. 
    * **Action:** Click the **Lock icon** to ensure the token is encrypted.
3.  **Authorize Pipeline:** Under the variable group settings, go to **Pipeline Permissions** and ensure "Allow access to all pipelines" is checked.



### Step 02: Process Template Configuration
The script must target the specific process methodology used by your organization.
1.  Open the `azure-pipelines.yml` file.
2.  Locate the `processTemplate` section within the PowerShell script.
3.  Comment/Uncomment the `templateTypeId` based on your requirement:
    * **Basic:** `b8a3a935-7e91-48b8-a94c-606d37c3e9f2`
    * **Agile:** `adcc42ab-9882-4ad5-a3ef-3919920739e3`
    * **Scrum:** `b724908-ef14-45cf-84f8-768b5384da45`

### Step 03: Environment & Approval Gate
To prevent unauthorized project creation, a manual approval gate is required.
1.  Navigate to **Pipelines > Environments**.
2.  Create an environment named `Project-Creation-Gate`.
3.  Click **Approvals and Checks** and add the Project Administrators or DevOps Leads as mandatory approvers.



### Step 04: Pipeline Creation in ADO
1.  Navigate to **Pipelines > New Pipeline**.
2.  Select the source code location where your YAML is stored.
3.  Select **Existing Azure Pipelines YAML file** and point to `azure-pipelines.yml`.
4.  **Save** (do not run yet).

---

## 🚀 Running the Pipeline (Example)

When you are ready to provision a project, click **Run Pipeline** and provide the following parameters:

| Parameter | Example Value |
| :--- | :--- |
| **Project Name** | `Finance-App-2026` |
| **Project Description** | `Core banking module for the 2026 fiscal year.` |
| **Contributors** | `john.doe@org.com, jane.smith@org.com` |

---

## 🛠 Workflow Logic Summary

1.  **Validation Stage:** The pipeline performs a Regex check to ensure the project name is valid and pings the API to verify the name isn't already taken.
2.  **Approval Stage:** The pipeline halts and notifies the Admin. The Admin reviews the requested name and contributors before clicking **Approve**.
3.  **Provisioning Stage:**
    * **Creation:** Uses `curl` to POST to the REST API.
    * **Stabilization:** The script sleeps for 15 seconds to allow background group synchronization.
    * **User Management:** Uses the **Graph API** to resolve the provided emails to Identity Descriptors and adds them to the `[Project]\Contributors` security group.

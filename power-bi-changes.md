### Microsoft Power BI and BAPS Dashboard: Changes and Future State

## Current State

### Microsoft Power BI

#### Account Creation & Access
- A separate Power BI account is created for each department.
- Credentials for each account are shared with the designated point of contact within the respective department.
- Multiple users from the same department utilize the shared credentials.

#### Workspace & Report Creation
- Users log in using the shared credentials to create an Account Workspace in Power BI.
- Reports are generated within the workspace using various data sources, such as Excel.
- Reports remain confined to the departmental account, with no cross-departmental visibility.

### BAPS Dashboard

#### Account Setup
- Each department’s Power BI account credentials are stored in the BAPS Dashboard.
- This configuration allows the dashboard to access and manage reports for the respective department.

#### Synchronization & Report Management
- **Sync Workspace**: Retrieves workspaces from the associated departmental Power BI account.
- **Sync Report**: Imports reports from the synced workspace into the BAPS Dashboard.
- **Report Folder Assignment**: Displays available reports per account.
- **Report Access Assignment**: Grants or restricts report access based on account selection.

## Future State

### Microsoft Power BI

#### Master Account Structure
- A **Master Apps Power BI Account** is established to centrally manage all departmental workspaces.
- Individual departmental workspaces will be created under this master account, such as:
  - **Satsang Network Workspace**
  - **iNAHQ Workspace**
  - **Yuvak Workspace**

#### User-Specific Access via BAPS ID
- Instead of shared credentials, departmental workspaces will be assigned to individual BAPS ID accounts.
- Example: **Satsang Network Workspace** will be shared with:
  - **Samirbhai (sp8528001@na.baps.id)**
  - **Aksharbhai (ap1113020@na.baps.id)**

#### Best Practices for Report Management
- Users must log in using their **BAPS ID accounts** to access Power BI.
- New reports and datasets must be published exclusively to the designated departmental workspace.
- Reports should be created using various data sources, including Excel and Semantic Models.

### BAPS Dashboard

#### Enhanced Synchronization Process
- **Sync Account Workspace**: Links departments to their respective account workspaces.
- **Sync Report**: Imports reports from the departmental workspace into the BAPS Dashboard.
- **Report Folder Assignment**: Now displays reports through Account Workspaces rather than accounts based on departments.
- **Report Access Assignment**: Grants or restricts report access through account workspace selection.

#### Guidelines for Users
- **Shared credentials will no longer be used**.
- **All reports must be published within their designated departmental workspaces**.

## Step-by-Step Implementation Guide

### Microsoft Power BI Configuration (Future State Implementation)

#### 1. Download Existing Reports, Dashboards, and Data
1. Log in to the **Microsoft Power BI Admin Portal** using current departmental account credentials.
2. Download all necessary artifacts (Reports, Dashboards, Data) from existing workspaces.
3. Log out from the **Microsoft Power BI Admin Portal**.
4. Log in to the **Microsoft Power BI Admin Portal** using your **BAPS ID Account**.
5. Verify if you have access to the shared departmental workspace; if not, contact the support team.
6. Upload/Publish the downloaded artifacts (Reports, Dashboards, Data) to the assigned departmental workspace.
7. Verify that all reports and dashboards are functioning correctly.

### BAPS Dashboard Configuration

#### 1. Synchronizing Reports
1. Log in to the **BAPS Dashboard**.
2. Navigate to the **Report** section under **Admin**.
3. Click on the **Sync Report** button.
4. Select the appropriate reports to sync them into the BAPS Dashboard.

#### 2. Assigning Reports to Folder
1. Navigate to **Report Folder Assignment**.
2. Select the appropriate **Department → Workspace → Report**.
3. Assign the report to the designated folder.

#### 3. Configuring Report Access
1. Navigate to **Report Access Assignment**.
2. Select the appropriate **Department → Workspace → Report**.
3. Assign access based on **roles/positions**.

## Key Improvements and Benefits

### Centralized Account Management
- **A single Master Apps Power BI Account simplifies administration**.
- **Eliminates department-based Power BI accounts**.

### User-Specific Authentication
- **No more shared credentials**—users log in with their **BAPS ID accounts**.
- **Enhances security and accountability**.

### Improved Report Organization
- **Each department’s reports are securely stored in its dedicated Departmental Workspace**.
- **Prevents intermixing of reports across different departments**.


# AWS Health to Jira Automation Workflow

This n8n workflow automates the process of monitoring, filtering, and organizing **AWS Health notifications** directly into **Jira Software Cloud**. 

By leveraging Artificial Intelligence (LiteLLM with Anthropic Claude), the workflow automatically filters out noise (routine, non-actionable emails) and creates structured parent tasks and sub-tasks in Jira. This ensures that your Platform Operations or Engineering teams never miss critical AWS deadlines while preventing issue duplication.

---

## Part 1: Beginner's Introduction Guide

### What This Workflow Does
When Amazon Web Services (AWS) sends a health notification or account alert (e.g., end-of-support notices, service degradations, or required configurations), it usually arrives via email. If you manage multiple AWS accounts, managing these alerts manually gets overwhelming. 

This workflow acts as an automated triage engineer:
1. **Listens** for new unread emails from `health@aws.com`.
2. **Cleans and Extracts** the raw information using a lightweight script.
3. **Analyzes** the content using an AI Model to determine if immediate manual action is required.
4. **Checks Jira** to see if a ticket for this specific AWS global event already exists.
5. **Creates and Assigns** tasks automatically, linking account-specific sub-tasks to a single parent issue.

### How It Works (Step-by-Step)

<img width="2516" height="642" alt="image" src="https://github.com/user-attachments/assets/ccd3ab39-263e-4fce-b52d-aedfcdf2592a" />

* **Step 1: Microsoft Outlook Trigger** – Polls your inbox every minute looking for unread emails sent by `health@aws.com`.
* **Step 2: Extract details from Email** – A JavaScript code block strips out messy HTML text and performs an initial regex sweep to look for Account IDs and potential deadlines.
* **Step 3: Prompt for AI & AI (via LLM)** – Passes the raw email context to an AI model. The AI decides if the email is **ACTIONABLE** (requires human intervention, tight deadline) or **INFORMATIONAL** (routine maintenance handled automatically by AWS).
* **Step 4: If3 (Filter Node)** – If the AI determines the email is informational or an error occurred, the workflow cleanly terminates right here to prevent noise.
* **Step 5: Look for Parent Task if it Exists Already** – Searches Jira via JQL (Jira Query Language) to find out if this global AWS event has already spawned a parent issue in your active project.
* **Step 6: If (Decision Node) & Create Parent Task** – If no matching parent task is found, it creates a general parent task outlining the global AWS issue. If it *does* exist, it grabs the key of the existing issue.
* **Step 7: Check if subtasks already exists** – Checks if a sub-task already exists for that *specific* AWS Account ID under the parent issue.
* **Step 8: If2 & Create subtask** – If that account doesn't have a sub-task yet, it creates one, formatting the summary as `AWS Account: [ID] – [Deadline]` and links it directly to the parent issue.

---

## Part 2: Setup and Deployment Guide

Follow these steps to customize, configure credentials, and run this workflow in your own n8n environment.

### Prerequisites
* An **n8n** instance running (Self-hosted or Cloud).
* A **Microsoft Outlook** business/personal account receiving AWS Health notifications.
* A **Jira Software Cloud** account with administrator or standard agent permissions.
* An API Key for **LiteLLM** or a compatible OpenAI/Anthropic endpoint.

### Step 1: Import the Workflow
1. Copy the raw JSON file provided in your configuration.
2. Open your n8n workspace dashboard.
3. Click on the top-right menu icon (three dots) and select **Import from File** (or press `Ctrl+V` / `Cmd+V` directly on the canvas).

### Step 2: Configure Node Credentials

You need to link your own accounts to three separate nodes within this workspace:

#### 1. Microsoft Outlook Trigger
* Double-click the **Microsoft Outlook Trigger** node.
* Under **Credential for Microsoft Outlook OAuth2 API**, select **Create New Credential**.
* Follow the OAuth2 onboarding prompt to grant n8n read access to your mailbox.
* *Optional:* Change the `custom` filter text if your AWS emails route to a specific shared mailbox or alternate alias.

#### 2. AI (via LLM) HTTP Request
* Double-click the **AI (via LLM)** node.
* Locate the **Headers** section.
* Replace the value `Bearer sk-xxxxxx` with your actual API key token (e.g., `Bearer your-real-api-key`).
* If your LiteLLM instance uses a different endpoint URL, update the **URL** field to your custom routing provider.

#### 3. Jira Software Cloud Nodes
There are three Jira nodes (**Look for Parent Task...**, **Create an parent task**, and **Check if subtasks...**).
* Open any of these Jira nodes.
* Under **Credential for Jira Software Cloud API**, click **Create New Credential**.
* Input your Jira **Domain** URL, **Email**, and **API Token** (Generated from your Atlassian Security Profile).
* **CRITICAL CUSTOMIZATION:** Update the project configuration fields inside the Jira nodes:
    * In **Look for Parent Task...**: Change `project = XXX` in the JQL query string to your unique Jira Project Key (e.g., `project = DEVOPS`).
    * In **Create an parent task** & **Create subtask**: Update the **Project** dropdown value to your specific workspace project list item.

### Step 3: Customizing the AI Behavior
If you wish to change what the AI deems "Actionable" vs "Informational", double-click the **Prompt for AI** node. Inside the JavaScript string context, you can append or change guidelines inside the `system` role block, such as adjusting the acceptable maintenance planning window definitions.

### Step 4: Test and Activate
1. Click **Listen for test event** on the Outlook trigger node or execute the workflow manually with mock pinned email data.
2. Verify that parent tasks and sub-tasks are populating and linking correctly inside your designated Jira board.
3. Toggle the workflow status slider in the top right from **Inactive** to **Active**.

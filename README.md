# Intelligent Email Inquiry Routing System for HPF

**Case Study – Automation Engineer (Hasso Plattner Foundation)**

<img width="1854" height="1048" alt="Screenshot from 2025-10-20 10-57-50" src="https://github.com/user-attachments/assets/a81b3316-edfa-4868-9d69-5c7ee35c4183" />


---

## 🧠 Objective

The goal of this project is to design, build, and deploy an **automated email triage system** using **n8n**, powered by an **AI model** for classification and data extraction. The workflow processes inbound emails, extracts key details, classifies intent, stores results, and triggers appropriate automated actions.

---

## 🏗️ Architecture Overview

### **System Components**

* **n8n Web App (Containerized)**: Deployed on **Azure App Service** using the **Container publishing model**.

* **Email Integration**: Gmail trigger node for inbound message polling.

* **AI Layer**: LangChain Agent + GPT-4o-mini via OpenRouter API for classification and data extraction.

* **Data Logging**: Google Sheets integration for structured data persistence.

* **Routing Logic**: Conditional logic in n8n to send automated replies and store results.

This setup ensures the solution is **modular, cloud-native, and easily extensible** with minimal infrastructure management.

---

## ☁️ Why Azure Web App for Containers?

### **Overview**

The n8n instance was deployed as a **Web App on Azure App Service** using the **Container (Docker) publishing model**. This approach packages the entire automation environment — including dependencies, n8n configurations, and credentials — into a single container image that Azure runs and manages as a web application.

## 🚀 Setup & Execution

### **Prerequisites**

Before you begin, ensure you have the following:

* Azure Subscription
* Gmail and Google Sheets API credentials (OAuth2)
* OpenRouter API key
* n8n workflow JSON file (`HPF CaseStudy.json`)

---

### **Deployment Steps**

#### **1. Create an Azure Web App**

1. Log in to your [Azure Portal](https://portal.azure.com/).

2. Create a **Resource Group** and select your preferred **Region**.

3. Navigate to **Azure App Services** → click **Create** → choose **Web App**.

4. Fill in the configuration details:

   * **Resource Group:** Select the one you just created.
   * **Publish:** Container
   * **Region:** Same as your resource group.
   * **Operating System:** Linux
   * **App Service Plan:** Create a new one (e.g., `n8nRg`).
   * **Pricing Plan:** Basic B1

5. Proceed to the **Container** tab:

   * **Image Source:** Other Container
   * **Access Type:** Public
   * **Registry Server URL:** `https://docker.n8n.io`
   * **Image & Tag:** `n8nio/n8n`

6. Click **Review + Create** and deploy the app.

---

#### **2. Set Environment Variables**

After deployment:

1. Stop the App Service instance.

2. Navigate to **Settings → Configuration → Application Settings**.

3. Add the following environment variables:

   ```
   DOCKER_REGISTRY_SERVER_URL = https://docker.n8n.io
   DOCKER_REGISTRY_SERVER_USERNAME = <your-username>
   DOCKER_REGISTRY_SERVER_PASSWORD = <your-password>
   GENERIC_TIMEZONE = <your-timezone>

   N8N_BASIC_AUTH_ACTIVE = true
   N8N_BASIC_AUTH_USER = <username>
   N8N_BASIC_AUTH_PASSWORD = <password>

   N8N_HOSTING = <app-name>.azurewebsites.net
   N8N_PORT = 5678
   N8N_PROTOCOL = https
   WEBHOOK_URL = https://<app-name>.azurewebsites.net
   WEBSITES_ENABLE_APP_SERVICE_STORAGE = true
   ```

4. Save and restart the app.

---

#### **3. Access n8n**

* Once the app restarts, click on the **Default Domain** (e.g., `https://<app-name>.azurewebsites.net`).
* n8n will open in your browser.
* Log in using the credentials you set in the environment variables.

---

#### **4. Import the Workflow**

1. In the n8n dashboard, click **Workflows → Import from File**.
2. Upload `HPF CaseStudy.json`.
3. Save the workflow.

---

#### **5. Connect Credentials**

Configure the following credentials inside n8n:

* **Google Credentials:** Connect Gmail and Google Sheets OAuth2 access.
* **OpenRouter API Key:** Add your OpenRouter API key under credentials management.

---

#### **6. Activate and Test**

* Activate the workflow by toggling it **ON**.
* Send a test email to the connected Gmail account.
* Verify that:

  * Emails are automatically classified.
  * Replies are sent for new applications.
  * Data is logged in Google Sheets.

---

Your **n8n workflow** is now successfully deployed and running on **Azure Web App for Containers**.


---

## 🌐 Why This Approach Works Well for n8n

| Advantage                     | Explanation                                                                                                                                 |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| **Ease of Deployment**        | Azure App Service natively supports Docker containers, allowing you to deploy n8n without managing virtual machines or Kubernetes clusters. |
| **Managed Hosting**           | Handles container orchestration, scaling, SSL, and OS patching automatically — ideal for low-maintenance projects.                          |
| **Persistent Configuration**  | Environment variables and mounted storage provide a clean separation between code, configuration, and state — simplifying updates.          |
| **Continuous Deployment**     | Integrates directly with GitHub Actions, Azure DevOps, or Container Registries for CI/CD pipelines.                                         |
| **Scalability & Reliability** | The Web App automatically scales vertically or horizontally based on email volume, ensuring consistent uptime.                              |
| **Security & Compliance**     | Built-in authentication, HTTPS enforcement, and integration with Azure Key Vault make it production-ready.                                  |
| **Optimized for n8n**         | Since n8n is container-native, App Service can easily host it as a secure, always-on automation platform without custom orchestration.      |

### **Why It’s a Good Fit for n8n Projects**

* **Container-Native Environment** – n8n is officially distributed via Docker; Azure’s App Service for Containers runs it without modification.
* **Simple Scaling** – Perfect for workflows that may grow gradually — scale from one container to multiple instances seamlessly.
* **Minimal Ops Overhead** – No need for infrastructure engineers or Kubernetes knowledge to maintain it.
* **Built-in Monitoring** – Integrates with **Azure Monitor** and **Application Insights** for health and performance visibility.
* **Ideal for Proof-of-Concepts & SME Automation** – Combines low setup cost with enterprise-level stability and security.

---

## ⚙️ Workflow Implementation

### **Trigger: Gmail Trigger**

* Polls every minute for new **unread emails** in the Inbox.
* Authenticated via **Gmail OAuth2**.
* Filters for “INBOX” and “UNREAD” to avoid duplicate executions.

### **AI-Powered Triage ( OpenRouter GPT-4o-mini)**

* Passes email **subject** and **body** to an AI model.
* Model classifies email intent:

  * `New Application`
  * `Status Update`
  * `General Question`
  * `Irrelevant`
* Extracts:

  * `Organization Name`
  * `Contact Person`
  * `Project Title`
* The AI output is parsed and structured into JSON via a **Structured Output Parser**.

### **Routing and Action**

1. **If Node (Conditional Routing)**

   * If `Intent == "New Application"` → Send acknowledgment email + log to Google Sheets.
   * Otherwise → Log only.
2. **Reply Email Node (Gmail API)**

   * Sends templated auto-response:

     ```
     Dear <Contact Person>,

     Thank you for submitting your application. We are pleased to confirm its receipt.
     You can track your application status at:
     https://www.your-foundation-portal.org/login

     Sincerely,
     The Grants Administration Team
     ```
3. **Google Sheets Logging Node**

   * Appends data to a Google Sheet:

     * Contact Person
     * Organization Name
     * Project Title
     * Intent
     * Timestamp

---



## 💡 Assumptions – Email Classification Logic

The following assumptions define how incoming emails are categorized within the workflow:

* **New Application**
  Triggered when a sender submits a **new grant or funding application**. These emails typically include project details, organization information, or application attachments.

* **Status Update**
  Used when the sender is **inquiring about the progress or outcome** of an already submitted application. The intent is to check status or next steps.

* **General Question**
  Applies to emails containing **general queries** related to the foundation, grant process, eligibility, or submission guidelines. These are not tied to any specific application.

* **Irrelevant**
  Captures any email that **does not fit** the above categories — for example, spam, promotional content, or unrelated correspondence.

Each category guides the subsequent workflow actions, ensuring that only relevant emails trigger automated responses or logging processes.




## 🧭 Improvements & Future Enhancements

| Area              | Potential Improvement                                                        |
| ----------------- | ---------------------------------------------------------------------------- |
| **AI Accuracy**   | Integrate Azure OpenAI GPT-4-turbo for better classification.                |
| **Data Storage**  | Replace Google Sheets with Azure Cosmos DB or Table Storage.                 |
| **Notifications** | Add Teams or Slack alerts for critical email categories.                     |
| **Security**      | Integrate Azure Key Vault for secret management.                             |
| **Scalability**   | Migrate to Azure Container Apps or AKS for distributed workflow execution.   |
| **Monitoring**    | Link App Service to Application Insights for real-time telemetry and alerts. |

---

## 📦 Deliverables

* `HPF CaseStudy.json` – Exported n8n workflow
* `README.md` – This documentation file
* Any supporting environment or configuration scripts (if applicable)

---

## 🧾 References

* [n8n Documentation](https://docs.n8n.io/)
* [Azure Web App for Containers](https://learn.microsoft.com/en-us/azure/app-service/containers/)
* [OpenRouter API](https://openrouter.ai/)
* [Google API Docs (Gmail & Sheets)](https://developers.google.com/apis-explorer)

---

**Author:** Junaid Asif Sheikh 
**Date:** October 2025
**Version:** 1.1

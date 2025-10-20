# Intelligent Email Inquiry Routing System for HPF

**Case Study – Automation Engineer (Hasso Plattner Foundation)**



---

## 🧠 Objective

The goal of this project is to design, build, and deploy an **automated email triage system** using **n8n**, powered by an **AI model** for classification and data extraction. The workflow processes inbound emails, extracts key details, classifies intent, stores results, and triggers appropriate automated actions.

---

## 🏗️ Architecture Overview

### **System Components**

* **n8n Web App (Containerized)**: Deployed on **Azure App Service** using the **Container publishing model**.

* **Email Integration**: Gmail trigger node for inbound message polling.

* **AI Layer**:  GPT-4o-mini via OpenRouter API for classification and data extraction.

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



Your **n8n workflow** is now successfully deployed and running on **Azure Web App for Containers**.


---
## ⚙️ Why This Approach Works Well for n8n

This deployment method — using **Azure Web App for Containers** with the official **n8n Docker image** — aligns perfectly with n8n’s recommended architecture and design principles.

* **Official Docker Support**
  The n8n documentation officially recommends running n8n as a **Docker container**, ensuring full compatibility, easy updates, and consistent performance across environments.

* **Native Container Hosting in Azure**
  Azure App Service for Containers is built to **natively run Docker images**, eliminating the need for complex orchestration or infrastructure setup.

* **Direct Access to the Official n8n Image**
  The **official n8n Docker image** can be pulled directly from the trusted Docker registry (`https://docker.n8n.io`), ensuring the deployment always uses a secure and up-to-date version.

* **Simplified & Faster Deployment**
  Since Azure App Service handles networking, scaling, SSL, and maintenance automatically, deploying n8n as a container becomes a **fast, reliable, and low-maintenance process**.

Together, these factors make the Azure App Service container model an ideal hosting environment for **n8n-based automation workflows**, especially for rapid prototyping and production-ready setups.

---


## ⚙️ Workflow Implementation

<img width="1314" height="593" alt="image" src="https://github.com/user-attachments/assets/3a4d6d94-4cf5-4224-ab84-e62cabbeec6e" />



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





## 🚀 Improvements and Future Enhancements

The current workflow effectively automates email triage and response, but several enhancements can further improve accuracy, scalability, and intelligence.

* **Add Email Labeling**
  Implement automatic labeling of emails (e.g., “New Application”, “Status Update”) in Gmail to simplify tracking and organization.

* **Human-in-the-Loop for Complex Cases**
  Introduce a manual review step for ambiguous or high-impact emails to ensure accuracy and contextual understanding.

* **Expand Email Categories**
  Create additional classification types to handle edge cases or specific types of inquiries (e.g., partnership requests, funding clarifications).

* **Refine Workflow Structure**
  Optimize the flow logic to improve readability and maintainability, with clear branching for each category.

* **Integrate RAG for General Questions**
  For general inquiries, use a **Retrieval-Augmented Generation (RAG)** approach connected to a document knowledge base to provide accurate, context-rich responses.

* **Improve Status Update Handling**
  Maintain a database of application statuses to automatically retrieve and respond with the correct application progress information.

* **Handle Missing Information Gracefully**
  Add a dedicated sub-flow for cases where users omit their **name** or **organization details**, prompting the AI or workflow to handle it intelligently.

* **Dual-Intent Detection**
  Create a conditional logic path for emails that contain **multiple intents** (e.g., a new submission that also includes a status inquiry).

* **Integrate Azure OpenAI GPT-4-turbo**
  Replace the current OpenRouter model with **Azure OpenAI GPT-4-turbo**, enabling deeper integration with the Azure ecosystem, stronger compliance, and enterprise-grade performance.

These enhancements will make the workflow more intelligent, user-friendly, and adaptable for large-scale deployment in a real-world foundation environment.


---

## 📦 Deliverables

* `HPF CaseStudy.json` – Exported n8n workflow
* `README.md` – This documentation file
  

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

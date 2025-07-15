
# 🧩 Issue Resolution: Logic App Failing to Access Azure Blob Storage via Connector

While working on a Logic App solution, we encountered a subtle but impactful issue: the workflow could write to Azure Blob Storage but failed when trying to read or manipulate blob data via a Logic App action. Here’s a breakdown of the problem, how we debugged it, and what ultimately resolved it.

---

## 🏗️ Architecture Context

- **Logic App**: Standard SKU deployed in **App Service Environment (ASE)** with **VNet integration**
- **Azure Blob Storage**:
  - Public access: **Disabled**
  - Private endpoint: **Created in same VNet as Logic App**
- **RBAC**: All required permissions were correctly assigned

---

## 🧪 Observed Issue

- The Logic App **successfully stored** application-related files in the Blob Storage container.
- However, **"Create blob (V2)"** action **failed** to access blob data in another step of the workflow.

---

## 🔍 Debugging and Root Cause

To identify the issue, we used **Azure Application Insights** and examined the **Dependencies** table.

### Findings:

- When storing app files, the Logic App accessed:
  ```
  <storage_account_name>.blob.core.windows.net
  ```
  ✔️ This resolved correctly through the **private endpoint**.

- But when using the **“Create blob (V2)”** action, it attempted access via the **Managed Connector**, which internally uses the **Microsoft Multi-Tenant Gateway**.

  ❌ This gateway **resides outside our VNet**, and since **public access is disabled**, the request was blocked.

---

## ✅ Resolution

After further investigation, we discovered that **Built-in Connectors** for Azure Blob Storage **respect the VNet configuration of the App Service Environment**.

### What we did:
- Replaced **“Create blob (V2)”** (Managed connector)
- With **“Update blob in storage container”** (Built-in connector)

This switch ensured that all Blob Storage operations happened over the **private network path**, resolving the issue.

---

## 📘 Reference Links

- [Built-in Azure Blob Connector (Reference)](https://learn.microsoft.com/en-us/azure/logic-apps/connectors/built-in/reference/azureblob/#actions)
- [Azure Blob Managed Connector](https://learn.microsoft.com/en-us/connectors/azureblobconnector/)
- [Compare Built-in vs Managed Connectors](https://learn.microsoft.com/en-us/azure/connectors/compare-built-in-azure-connectors)

---

## 🧠 Key Takeaways

- **Built-in connectors** respect ASE and VNet configuration.
- **Managed connectors** route through Microsoft’s multi-tenant infrastructure and do **not** support private endpoints.
- When using **private endpoints and network-restricted environments**, always double-check your **connector type**.
- Tools like **Application Insights** and the **Dependencies table** are invaluable for tracing backend service calls.

---

Have you run into similar networking issues with Logic Apps or other PaaS services? Share your experience or thoughts in the comments below!

<!-- 

Repo:
https://github.com/degu0055/FunctionAppswithOutputBindings 

Submit here:
https://brightspace.algonquincollege.com/d2l/lms/dropbox/user/folder_submit_files.d2l?ou=791554&db=743547

-->


# Storage Queue Binding

This guide walks you through connecting an Azure Function to Azure Storage Queue using the Python v2 programming model in Visual Studio Code.

---

## ✅ Prerequisites

Before starting, make sure you have:

- [x] An [Azure subscription](https://azure.microsoft.com/free/)
- [x] Python (v3.8+ recommended)
- [x] Visual Studio Code with:
  - Python extension
  - Azure Functions extension
  - Azure Storage extension
  - Azurite extension (optional for local dev)
- [x] [Azure Functions Core Tools](https://learn.microsoft.com/azure/azure-functions/functions-run-local)
- [x] [Azure Storage Explorer](https://learn.microsoft.com/azure/storage/common/storage-explorer)

---

## ⚙️ Setup Environment

### 1. Create or Open an Azure Function Project

Use your existing Azure Function project or create a new one via VS Code.

### 2. Download Remote App Settings

To sync Azure Storage locally:
- Open Command Palette (`F1`)
- Select: `Azure Functions: Download Remote Settings...`
- Choose your Function App
- Overwrite `local.settings.json` when prompted
- Copy the value for `AzureWebJobsStorage` (used below)

---

## 📦 Register Binding Extensions

Ensure your `host.json` includes:

```json
{
  "version": "2.0",
  "extensionBundle": {
    "id": "Microsoft.Azure.Functions.ExtensionBundle",
    "version": "[3.*, 4.0.0)"
  }
}
```

---

## ➕ Add Output Binding to Azure Queue Storage

### 1. Modify `function_app.py`

Add the queue output binding:

```python
import azure.functions as func
import logging

app = func.FunctionApp(http_auth_level=func.AuthLevel.ANONYMOUS)

@app.route(route="HttpExample")
@app.queue_output(arg_name="msg", queue_name="outqueue", connection="AzureWebJobsStorage")
def HttpExample(req: func.HttpRequest, msg: func.Out[func.QueueMessage]) -> func.HttpResponse:
    logging.info('Processing HTTP request.')

    name = req.params.get('name')
    if not name:
        try:
            req_body = req.get_json()
            name = req_body.get('name')
        except ValueError:
            pass

    if name:
        msg.set(name)
        return func.HttpResponse(f"Hello, {name}. This HTTP triggered function executed successfully.")
    else:
        return func.HttpResponse(
            "This HTTP triggered function executed successfully. Pass a name in the query string or in the request body.",
            status_code=200
        )
```

---

## 🧪 Run Locally and Test

1. Press `F5` in VS Code to start the local function app.
2. Right-click `HttpExample` in the **Azure Functions** panel → `Execute Function Now...`
3. Enter input:
```json
{ "name": "Azure" }
```
4. Check the response output.

---

## 🔍 Verify Output in Azure Storage Explorer

1. Open **Azure Storage Explorer**
2. Navigate to your connected storage account
3. Go to **Queues** → `outqueue`
4. Confirm a message with the name (e.g., `Azure`) is present.

---

## 🚀 Deploy to Azure

1. Open Command Palette (`F1`)
2. Select `Azure Functions: Deploy to Function App`
3. Choose your Azure Function App
4. Confirm and deploy

You can now trigger your function from Azure and verify that it sends a message to the queue.

---

# Azure SQL Binding

## 📹 Video Demonstration

[Watch the video](https://www.youtube.com/watch?v=VIDEO_ID_HERE)


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

## Prerequisites

- Azure subscription  
- [Visual Studio Code](https://code.visualstudio.com/) with:
  - Azure Functions extension
  - Python extension  
- [Azure Functions Core Tools](https://learn.microsoft.com/en-us/azure/azure-functions/functions-run-local)  
- Python 3.9+  
- Complete the Quickstart: [Create a Python function in Azure using VS Code](https://learn.microsoft.com/en-us/azure/azure-functions/create-first-function-vs-code-python)

---

## Step 1: Create an Azure SQL Database

1. Follow the [Quickstart: Create a single database - Azure SQL Database](https://learn.microsoft.com/en-us/azure/azure-sql/database/single-database-create-quickstart)  
2. Use these values:  
   - **Database name**: `mySampleDatabase`  
   - **Server name**: must be globally unique  
   - **Authentication**: SQL Server Authentication  
   - **Admin login**: `azureuser`  
   - **Password**: (your strong password)  
   - **Allow Azure Services Access**: ✅ Yes  

3. Go to the **SQL Database > Connection Strings**, and copy the **ADO.NET** string.

---

## Step 2: Create Table

In the Azure portal:

1. Navigate to `mySampleDatabase`  
2. Open **Query Editor**  
3. Paste and run this query:

```sql
CREATE TABLE dbo.ToDo (
    [Id] UNIQUEIDENTIFIER PRIMARY KEY,
    [order] INT NULL,
    [title] NVARCHAR(200) NOT NULL,
    [url] NVARCHAR(200) NOT NULL,
    [completed] BIT NOT NULL
);
```

---

## Step 3: Update Function App Settings

1. In VS Code, open Command Palette:  
   - `Ctrl+Shift+P` → `Azure Functions: Add New Setting...`  
2. Select your Function App  
3. Set:  
   - **Setting Name**: `SqlConnectionString`  
   - **Value**: your updated ADO.NET connection string (edit the password first)  

4. Download remote settings:  
   - `Ctrl+Shift+P` → `Azure Functions: Download Remote Settings...`  
   - Overwrite when prompted

---

## Step 4: Ensure Extension Bundles Are Enabled

Check your `host.json` includes:

```json
{
  "version": "2.0",
  "extensionBundle": {
    "id": "Microsoft.Azure.Functions.ExtensionBundle",
    "version": "[4.*, 5.0.0)"
  }
}
```

---

## Step 5: Add SQL Output Binding (Python)

In your `function_app.py`, modify the function:

```python
import azure.functions as func
import logging
from azure.functions.decorators.core import DataType
import uuid

app = func.FunctionApp()

@app.function_name(name="HttpTrigger1")
@app.route(route="hello", auth_level=func.AuthLevel.ANONYMOUS)
@app.generic_output_binding(
    arg_name="toDoItems",
    type="sql",
    CommandText="dbo.ToDo",
    ConnectionStringSetting="SqlConnectionString",
    data_type=DataType.STRING
)
def test_function(req: func.HttpRequest, toDoItems: func.Out[func.SqlRow]) -> func.HttpResponse:
    logging.info('Processing HTTP request.')

    name = req.get_json().get('name')
    if name:
        toDoItems.set(func.SqlRow({
            "Id": str(uuid.uuid4()),
            "title": name,
            "completed": False,
            "url": ""
        }))
        return func.HttpResponse(f"Hello {name}!")
    else:
        return func.HttpResponse(
            "Please pass a name in the request body",
            status_code=400
        )
```

---

## Step 6: Run Locally

1. Press `F5` to run the project.  
2. Trigger function:  
   - `Azure: Functions` → `HttpTrigger1` → Right-click → **Execute Function Now**  
   - Enter request body: `{ "name": "Azure" }`  

3. Check Azure SQL:  
   - Go to `Query Editor`  
   - Run:

```sql
SELECT TOP 1000 * FROM dbo.ToDo;
```

---

## Step 7: Redeploy to Azure

1. Press `F1` → `Azure Functions: Deploy to Function App`  
2. Select existing app  
3. Confirm overwrite

---

## ✅ Done

Now your Azure Function app is connected to Azure SQL Database using an output binding and writes rows on HTTP trigger!

## 📹 Video Demonstration

[Watch the video](https://drive.google.com/file/d/1NiMWLDf71s8qrfjhDOWumpiSj6lp6LME/view?usp=sharing)

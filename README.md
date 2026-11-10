## Watch ME Build This Lab!
Hands-on Microsoft Azure lab demonstrating the deployment and configuration of a static website using Azure Blob Storage.

# Lab 01: Hosting Your First Static Website in Azure

![Azure](https://img.shields.io/badge/Cloud-Azure-0078d4) ![Level](https://img.shields.io/badge/Difficulty-Beginner-brightgreen) ![Time](https://img.shields.io/badge/Time-30%20min-blue) ![Service](https://img.shields.io/badge/Service-Blob%20Storage-informational)


## Overview

I deployed my first public-facing Azure resource: a static website hosted on **Azure Blob Storage**. Instead of building and patching a web server, Azure runs the infrastructure and I only manage the content. This is a hands-on introduction to **PaaS** and **serverless hosting**.

**Skills demonstrated:** resource groups, storage accounts, static website hosting, naming conventions, validation, troubleshooting, and cost-conscious cleanup.

## Architecture

![Architecture diagram](images/architecture.png)

**Flow:**

1. A user's browser sends an HTTPS request to the storage account's static website endpoint.
2. Azure Storage serves the matching file from the `$web` container.
3. The browser receives and renders `index.html`.

```text
User (Internet) -> Public endpoint URL -> Storage Account -> $web container -> index.html
```

## Prerequisites

- [ ] Active Azure subscription (Free Tier is fine)
- [ ] A text editor (Notepad, TextEdit, or VS Code)
- [ ] Week 1 video modules completed

## Lab variables

| Item | Value |
|---|---|
| Resource group | `rg-lab01-timothye` |
| Region | East US |
| Storage account | `stlab01timothye` |

> Storage account names must be **globally unique**, 3-24 characters, lowercase letters and numbers only.

## Step-by-step instructions

### Phase 1: Create the resource group

1. Sign in to the [Azure Portal](https://portal.azure.com).
2. Search for **Resource groups** and click **+ Create**.
3. On the **Basics** tab, set:
   - Subscription: your subscription
   - Resource group: `rg-lab01-timothye`
   - Region: **(US) East US**
4. Click **Review + create**, then **Create**.

### Phase 2: Create the storage account

1. Search for **Storage accounts** and click **+ Create**.
2. On the **Basics** tab, set:
   - Resource group: `rg-lab01-timothye`
   - Storage account name: `stlab01timothye` (for example `stlab01timothye`)
   - Region: **(US) East US**
   - Performance: **Standard**
   - Redundancy: **Locally-redundant storage (LRS)**, the cheapest option for labs
3. Click **Review + create**, wait for validation, then **Create**.
4. When deployment finishes (about 30 seconds), click **Go to resource**.

### Phase 3: Enable static website hosting

1. In the storage account's left menu, under **Data management**, click **Static website**.
2. Set **Static website** to **Enabled**.
3. Index document name: `index.html`
4. Error document path: `404.html` (optional, good practice)
5. Click **Save**.
6. Copy the **Primary endpoint** URL (for example `https://stlab01timothye.z13.web.core.windows.net/`). This is your site's address.

### Phase 4: Create the website file

Save the following as `index.html` on your desktop:

```html
<!DOCTYPE html>
<html>
<head>
    <title>My First Cloud Site</title>
    <style>
        body { font-family: sans-serif; text-align: center; margin-top: 50px; background-color: #f0f0f0; }
        h1 { color: #0078d4; }
    </style>
</head>
<body>
    <h1>Hello from the Cloud!</h1>
    <p>This site is hosted on Azure Blob Storage.</p>
    <p>Deployed by: <Your Name></p>
</body>
</html>
```

### Phase 5: Upload content

1. In the storage account, open **Containers** (under **Data storage**).
2. Open the **`$web`** container. Azure created it automatically when you enabled static hosting.
3. Click **Upload**, select `index.html`, and click **Upload**.

### Phase 6: Validation

1. Open a new browser tab and paste the Primary endpoint URL.
2. Press Enter.
3. **Expected result:** the page shows "Hello from the Cloud!".

![Screenshot of the deployed site](images/site-screenshot.png)

*Add your own screenshot at `images/site-screenshot.png`.*

**Negative test:** append a path that does not exist, such as `/nope`. With `404.html` uploaded, you should see your custom error page. Without it, Azure returns its default 404 response.

## Security notes

- The static website endpoint is **intentionally public**. Only put content in `$web` that you are happy to publish. Never upload secrets, keys, or private data.
- Traffic is served over HTTPS by default on the endpoint.
- Production hardening would add a custom domain with Azure Front Door or Azure CDN, a WAF, and diagnostic logging to Log Analytics.
- Grant yourself only the roles you need (for example Storage Blob Data Contributor) rather than broad ownership.

## Troubleshooting

| Issue | Fix |
|---|---|
| **404 - The requested content does not exist** | The file name must be exactly `index.html`. Names are case-sensitive, so `Index.html` or `index.txt` will not work. Also confirm the file is in the `$web` container, not another one. |
| **Storage account name is already taken** | Names are unique across all of Azure. Add numbers to the end, for example `stlab01timothye`. |

## Optional: the same lab with Azure CLI

```bash
RG=rg-lab01-timothye
SA=stlab01timothye

az group create --name $RG --location eastus
az storage account create --name $SA --resource-group $RG \
  --location eastus --sku Standard_LRS --kind StorageV2
az storage blob service-properties update --account-name $SA \
  --static-website --index-document index.html --404-document 404.html
az storage blob upload --account-name $SA --container-name '$web' \
  --file index.html --name index.html --content-type text/html --auth-mode login
az storage account show --name $SA --resource-group $RG \
  --query "primaryEndpoints.web" -o tsv
```

## Clean up resources

Delete everything to avoid charges and keep the subscription tidy:

1. Go to **Resource groups** and open `rg-lab01-timothye`.
2. Click **Delete resource group**.
3. Type the resource group name to confirm.
4. Click **Delete**.

Or with the CLI:

```bash
az group delete --name rg-lab01-timothye --yes --no-wait
```

## Key takeaways

- Static hosting on Blob Storage needs no servers to manage or patch.
- Naming conventions and resource groups make cleanup a single action.
- Public by design still requires deciding what is safe to publish.

## Next steps

- Add a custom domain and HTTPS with Azure Front Door or CDN.
- Deploy the site automatically with GitHub Actions.
- Define this lab as code with Bicep or Terraform.

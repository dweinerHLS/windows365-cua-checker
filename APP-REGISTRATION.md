# App Registration Guide – CUA Prerequisite Checker

This guide walks through registering an Azure AD (Entra ID) application so the CUA Prerequisite Checker site can authenticate users and query Microsoft Graph.

---

## Step 1 – Create the App Registration

1. Sign in to the [Microsoft Entra admin center](https://entra.microsoft.com/) as a **Global Administrator** or **Application Administrator**.
2. Go to **Identity → Applications → App registrations**.
3. Click **+ New registration**.
4. Configure:

   | Field | Value |
   |---|---|
   | **Name** | `CUA Prereq Checker` (or any name you like) |
   | **Supported account types** | **Accounts in this organizational directory only** (single tenant) |
   | **Redirect URI** | Select **Single-page application (SPA)** and enter your GitHub Pages URL |

   > **Redirect URI examples:**
   > - `https://<username>.github.io/<repo-name>/`
   > - `http://localhost:8080` (for local testing — add this as a second URI)

5. Click **Register**.

---

## Step 2 – Note the Client ID

On the **Overview** page of your new app registration, copy the:

- **Application (client) ID** — you will need this to configure the tool

---

## Step 3 – Configure the Redirect URI (verify SPA type)

1. In your app registration, go to **Authentication**.
2. Under **Single-page application**, confirm your redirect URI is listed.
3. For local testing, also add `http://localhost:8080`.
4. Ensure **both** of these are **unchecked**:
   - ❌ Access tokens (used for implicit flow)
   - ❌ ID tokens (used for implicit flow)
   
   > The tool uses the **Authorization Code flow with PKCE** (not implicit), so these should remain unchecked.

5. Click **Save**.

---

## Step 4 – Add API Permissions

1. Go to **API permissions → + Add a permission → Microsoft Graph → Delegated permissions**.
2. Add the following permissions:

   | Permission | Description |
   |---|---|
   | `User.Read` | Sign in and read user profile |
   | `Application.Read.All` | Read all application/service principal registrations |
   | `DeviceManagementConfiguration.Read.All` | Read Microsoft Intune device configuration |
   | `Directory.Read.All` | Read directory data (groups, licenses) |
   | `RoleManagement.Read.Directory` | Read directory role assignments |

3. Click **Add permissions**.

---

## Step 5 – Grant Admin Consent

Several of the permissions above require **admin consent** because they read tenant-wide data.

1. On the **API permissions** page, click **Grant admin consent for \<your tenant\>**.
2. Click **Yes** to confirm.
3. All permissions should now show a **green ✅ Granted** status.

> ℹ️ **Only a Global Administrator can grant tenant-wide admin consent.**

---

## Step 6 – Configure the Tool

When opening the CUA Prereq Checker site:

1. Click **⚙️ Configure App** in the top-right header.
2. Paste your **Application (client) ID** into the Client ID field.
3. Verify the **Redirect URI** matches exactly what you registered.
4. Click **Save & Close**.
5. Click **🔐 Sign in with Microsoft**.

The configuration is saved in your browser's `localStorage`, so you only need to do this once per browser.

---

## (Optional) Hardcode the Client ID

If you are deploying this for your organization and want to avoid asking users to configure the Client ID, you can hardcode it in `index.html`:

Find this line:

```js
clientId: localStorage.getItem('cua_clientId') || 'YOUR_CLIENT_ID_HERE',
```

Replace with your Client ID:

```js
clientId: localStorage.getItem('cua_clientId') || 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx',
```

---

## Troubleshooting

| Error | Solution |
|---|---|
| `AADSTS50011: The redirect URI does not match` | Ensure the redirect URI in your app registration exactly matches the URL you're accessing (including trailing `/`) |
| `AADSTS65001: The user or administrator has not consented` | A Global Admin needs to grant admin consent on the API permissions page |
| `403 Forbidden` on Graph calls | The signed-in user may not have sufficient admin role. Results will show as `⚠️ WARN` with a note to check manually |
| `AADSTS700016: Application not found in directory` | The Client ID may be from a different tenant — check you're signing in with an account from the same tenant where the app was registered, or change **Supported account types** to **Multi-tenant** |
| Sign-in popup doesn't open | Check your browser isn't blocking popups for the site |

---

## Multi-Tenant Deployment

If you want this tool to work for **any Microsoft tenant** (not just your own), change the **Supported account types** in your app registration to:

**Accounts in any organizational directory (Any Microsoft Entra ID tenant – Multitenant)**

And update the authority in `index.html`:

```js
authority: 'https://login.microsoftonline.com/common',
```

(This is already the default in the code — no change needed.)

> ⚠️ With multi-tenant mode, users from any tenant can sign in. Each user's own tenant permissions determine what they can read.

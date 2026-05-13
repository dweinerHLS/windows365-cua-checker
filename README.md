# Windows 365 for Agents – CUA Prerequisite Checker

A browser-based tool that connects to your Microsoft tenant via the **Microsoft Graph API** and automatically validates all technical prerequisites required to deploy **Cloud PC pools** for **Computer Use Agent (CUA)** workloads in Microsoft Copilot Studio.

> 🔒 **Privacy:** All checks run entirely in your browser. No data is sent to any external server beyond the Microsoft Graph API endpoints being queried.

---

## Live Demo

🔗 **[https://\<your-github-username\>.github.io/\<repo-name\>/](https://github.com)**

---

## What It Checks

### ✅ Automated (via Microsoft Graph API)

| Category | Check |
|---|---|
| **Service Principals** | Windows 365 SP (`0af06dc6-...`) |
| **Service Principals** | Azure Virtual Desktop SP (`9cdead84-...`) |
| **Service Principals** | AVD Client SP (`a85cf173-...`) |
| **Service Principals** | AVD ARM Provider SP (`50e95039-...`) |
| **Service Principals** | Microsoft Remote Desktop SP (`a4a365df-...`) |
| **RDP Auth** | `isRemoteDesktopProtocolEnabled = true` on MSRD SP |
| **RDP Auth** | `targetDeviceGroups` configured for consent suppression |
| **Intune / Entra** | Windows MDM enrollment restrictions |
| **Intune / Entra** | Microsoft Entra ID P1 license present |
| **Intune / Entra** | Current user admin role (Global/Intune/W365/App Admin) |
| **Intune / Entra** | Dynamic group with `CPCPool_` membership rule |

### 📋 Manual Checklist (click to confirm)

- Azure subscription linked via pay-as-you-go billing plan
- Copilot Studio Add-on enabled in tenant
- Cloud PC feature enabled in Power Platform admin center
- Cross-geo support enabled (if applicable)
- Intune enrollment restriction allows Windows MDM corporate
- Enrollment Status Page configured (lab/demo tenants)
- Computer Use tool added to Copilot Studio agent

---

## Setup

### Step 1 – Register an Azure AD App

Before users can sign in, you need to register an Azure AD (Entra) application:

👉 **[Full App Registration Guide →](./APP-REGISTRATION.md)**

**Quick summary:**
1. Go to [Microsoft Entra App Registrations](https://entra.microsoft.com/#view/Microsoft_AAD_RegisteredApps/CreateApplicationBlade)
2. Create a new app → **Single-page application (SPA)**
3. Set the Redirect URI to your GitHub Pages URL (e.g., `https://username.github.io/repo/`)
4. Add delegated API permissions: `Application.Read.All`, `DeviceManagementConfiguration.Read.All`, `Directory.Read.All`, `RoleManagement.Read.Directory`
5. Grant admin consent
6. Copy the **Application (client) ID**

### Step 2 – Deploy to GitHub Pages

1. Fork or clone this repository
2. Push to GitHub
3. Go to your repo **Settings → Pages**
4. Set Source to **Deploy from a branch**, select `main` / `root`
5. GitHub Pages URL will be: `https://<username>.github.io/<repo-name>/`

### Step 3 – Configure the Client ID

When users open the site, they click **⚙️ Configure App** in the header and paste in your **Application (client) ID**. This is saved to `localStorage` so they only need to do it once per browser.

Alternatively, hardcode it in `index.html` by finding and replacing:

```js
clientId: localStorage.getItem('cua_clientId') || 'YOUR_CLIENT_ID_HERE',
```

with your actual Client ID:

```js
clientId: localStorage.getItem('cua_clientId') || 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx',
```

---

## Required Permissions

The app uses **delegated permissions** (the signed-in user's permissions), so the user must have the appropriate admin roles to get meaningful results.

| Permission | Purpose |
|---|---|
| `User.Read` | Get signed-in user's profile |
| `Application.Read.All` | Read service principals |
| `DeviceManagementConfiguration.Read.All` | Read Intune enrollment restrictions |
| `Directory.Read.All` | Read groups, licenses |
| `RoleManagement.Read.Directory` | Check admin role assignments |

> ⚠️ **Admin consent is required** for `Application.Read.All`, `DeviceManagementConfiguration.Read.All`, and `RoleManagement.Read.Directory`. A Global Admin must grant consent once for the whole tenant.

---

## Local Development

To test locally before deploying to GitHub Pages:

```powershell
# Using Python (built-in)
cd cua-prereq-checker
python -m http.server 8080

# Or using Node.js http-server
npx http-server . -p 8080
```

Then open `http://localhost:8080` in your browser.

> Make sure `http://localhost:8080` is added as a redirect URI in your app registration.

---

## Reference Links

| Resource | URL |
|---|---|
| Use Cloud PC pool for computer use runs | https://learn.microsoft.com/en-us/microsoft-copilot-studio/use-cloud-pc-pool |
| Windows 365 Requirements | https://learn.microsoft.com/en-us/windows-365/enterprise/requirements |
| Configure SSO for AVD (RDP auth) | https://learn.microsoft.com/en-us/azure/virtual-desktop/configure-single-sign-on |
| Set up Pay-As-You-Go | https://learn.microsoft.com/en-us/power-platform/admin/pay-as-you-go-set-up |
| Microsoft Graph PowerShell SDK | https://learn.microsoft.com/en-us/powershell/microsoftgraph/overview |
| Power Platform Admin Center | https://admin.powerplatform.microsoft.com/ |
| Microsoft Entra Admin Center | https://entra.microsoft.com/ |

---

## Credits

Inspired by the [Windows 365 Connectivity Diagnostics Tool](https://paulcollinge.github.io/W365ConnectivityTool/) by Paul Collinge.

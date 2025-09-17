# azure-ad-atlassian-sso-scim-guide
A step-by-step guide for integrating Azure Active Directory (Microsoft Entra ID) with Atlassian Cloud products (Jira, Confluence) for SAML SSO and SCIM provisioning.

# Azure AD (Entra ID) & Atlassian Cloud Integration Guide

## A Step-by-Step Guide for SAML SSO & SCIM User Provisioning

This guide provides detailed, step-by-step instructions for integrating **Azure Active Directory (Microsoft Entra ID)** with **Atlassian Cloud** products (Jira Software, Confluence) to configure **SAML Single Sign-On (SSO)** and **SCIM-based user provisioning**.

> **Goal:** To centralize user identity management in Azure AD, enabling seamless and secure login (SSO) and automatic user provisioning/de-provisioning in Atlassian Cloud.

---

## 📋 Prerequisites

| Component | Requirement |
| :--- | :--- |
| **Azure AD Tenant** | Global Administrator or Privileged Role Administrator account. |
| **Atlassian Cloud** | Organization Administrator account for your Atlassian site (e.g., `myteam.atlassian.net`). |
| **Atlassian Access** | This feature must be enabled on your Atlassian account. It is required for SSO and SCIM. |
| **Licenses** | Azure AD Premium P1 or P2 licenses are required for SCIM provisioning. |

---

## 🚀 Phase 1: Configure SAML Single Sign-On (SSO)

### Step 1.1: Add the Atlassian Cloud Application from the Azure AD Gallery

1.  Sign in to the **[Azure Portal](https://portal.azure.com)** as a Global Administrator.
2.  Navigate to **Azure Active Directory** > **Enterprise applications**.
3.  Click **New application** > **Browse gallery**.
4.  Search for **"Atlassian Cloud"** and select it.
5.  Enter a name for the application (e.g., `Atlassian Cloud Production`) and click **Create**.

### Step 1.2: Configure Basic SAML Settings

1.  In your new Enterprise Application, go to the **Single sign-on** section and select **SAML**.
2.  Click the edit (pencil) icon for **Basic SAML Configuration**.
3.  Enter the following identifiers and URLs:
    *   **Identifier (Entity ID):** `https://auth.atlassian.com/saml/<your-atlassian-org-id>`
        > **How to find your Org ID?** In your Atlassian admin console, go to **Security** > **Identity providers**. Your organization ID is displayed at the top.
    *   **Reply URL (Assertion Consumer Service URL):** `https://auth.atlassian.com/login/callback`
    *   **Sign on URL:** `https://auth.atlassian.com/login/callback?connection=saml-<your-atlassian-org-id>`
4.  Click **Save**.

### Step 1.3: Configure User Attributes & Claims

1.  In the **SAML-based Sign-on** section, click the edit icon for **Attributes & Claims**.
2.  Ensure the following claims are present. If not, add them:

    | Name | Value | Namespace |
    | :--- | :--- | :--- |
    | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier` | `user.mail` | - |
    | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress` | `user.mail` | - |
    | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name` | `user.displayname` | - |

### Step 1.4: Configure the Atlassian Cloud Side

1.  In a new tab, sign in to your **[Atlassian Organization Admin](https://admin.atlassian.com/)** portal.
2.  Go to **Security** > **Identity providers** > **Add SAML connection**.
3.  Give the connection a name (e.g., "Azure AD SSO").
4.  Return to the Azure AD tab. **Download the Federation Metadata XML** from the **SAML Signing Certificate** section.
5.  Back in the Atlassian tab, upload the downloaded XML file.
6.  Click **Continue** and then **Finish**.

---

## 🔄 Phase 2: Configure SCIM User Provisioning (Automatic)

### Step 2.1: Generate a SCIM Bearer Token in Atlassian

1.  In your **Atlassian Organization Admin** portal, go to **Security** > **User provisioning**.
2.  Click **Generate API token**. **Copy this token immediately**—you will not be able to see it again.

### Step 2.2: Configure Provisioning in Azure AD

1.  Back in your Azure AD Enterprise Application, go to the **Provisioning** section.
2.  Set **Provisioning Mode** to **Automatic**.
3.  In the **Admin Credentials** section:
    *   **Tenant URL:** `https://api.atlassian.com/scim/directory/<your-atlassian-org-id>`
    *   **Secret Token:** Paste the API token you generated in Atlassian.
    *   Click **Test Connection** to ensure it succeeds.
4.  Click **Save**.

### Step 2.3: Configure Attribute Mappings

1.  In the **Provisioning** section, click **Mappings**.
2.  Review the default mappings. The most critical ones are:
    *   **userName** -> **userName** (Maps to `emails[type eq "work"].value`)
    *   **active** -> **active**
    *   **emails[type eq "work"].value** -> **emails[type eq "work"].value**
    *   **displayName** -> **displayName**
3.  (Optional) To sync groups, edit the **Provision Azure Active Directory Groups** mapping and ensure the necessary group attributes are mapped.

### Step 2.4: Start the Provisioning Service

1.  In the **Provisioning** section, set **Provisioning Status** to **On**.
2.  Set the **Scope** to sync only assigned users and groups or all users.
3.  Click **Save**. The service will start an initial sync, which may take some time.

---

## ✅ Phase 3: Testing and Validation

### Test SSO Login
1.  Go to your application's overview page in Azure AD and click **Assign users and groups**.
2.  Assign a test user to the application.
3.  Use the ** assigned test user** to navigate to your Jira or Confluence site URL.
4.  You should be redirected to the Microsoft login page and, after authentication, seamlessly logged into Atlassian.

### Test Provisioning
1.  Assign a new user or group in Azure AD.
2.  Wait ~20-40 minutes for the provisioning cycle to run, or manually trigger it by going to the **Provisioning** blade and clicking **Provision on demand**.
3.  Verify the user appears in your Atlassian admin directory with the correct attributes.

---

## 🚨 Troubleshooting Common Issues

*   **"User not provisioned"**: Check the **Provisioning Logs** in Azure AD for detailed error messages. Common causes are duplicate emails or missing required attributes in Azure AD.
*   **SSO Login Fails**: Use the browser's developer tools (F12 -> Network tab) to capture the SAML request/response. Verify the NameID and other claims match the expected format in Atlassian.
*   **"Invalid credentials" when testing connection**: Regenerate the API token in Atlassian and update it in Azure AD.

---

## 📚 References

*   [Official Microsoft Docs: Atlassian Cloud Tutorial](https://learn.microsoft.com/en-us/azure/active-directory/saas-apps/atlassian-cloud-tutorial)
*   [Official Atlassian Docs: Azure AD Integration](https://support.atlassian.com/security-and-access-policies/docs/how-to-configure-scim-provisioning-with-azure-ad/)

---

## ⚖️ Disclaimer

This guide is provided for educational and reference purposes. Always test configurations in a non-production environment first. Procedures and UI may change over time.

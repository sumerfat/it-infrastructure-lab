# Lab 1: Enterprise Identity Management & User Onboarding

## 📋 Scenario Overview
A new employee, **Max Mustermann**, is joining the "Finance" department at a regional corporate office. As part of the IT Support / System Administration team, my task is to securely provision his user account, assign him to the correct departmental security groups, and ensure he has access to the required network shares while maintaining the principle of least privilege.

---

## 🛠️ Step-by-Step Administration Workflow

### 1. User Creation & Provisioning
* **Organizational Unit (OU):** The account is created under the `OU=Finance,OU=Staff,DC=company,DC=local` path to keep the directory structured and ensure specific Group Policy Objects (GPOs) apply to the Finance team.
* **Account Details:** 
  * **First Name:** Max
  * **Last Name:** Mustermann
  * **User Logon Name (User Principal Name):** `m.mustermann@company.local`
* **Security Password Policy:** A temporary complex password is generated. The option **"User must change password at next logon"** is explicitly checked to ensure security compliance and user privacy.

### 2. Role-Based Access Control (RBAC) & Group Membership
Instead of assigning permissions directly to Max's account, role-based access control is managed via Security Groups:
* **`G-Finance-Users` (Global Group):** Max is added to this group. This automatically maps the department's network drives (e.g., `F:\Finance-Share`) via Group Policy Preferences upon his next login.
* **`DL-Finance-ReadWrite` (Domain Local Group):** This group controls the actual NTFS folder permissions on the file server. `G-Finance-Users` is nested inside this Domain Local group following Microsoft's AGDLP best practice (Accounts -> Global -> Domain Local -> Permissions).

### 3. Verification & Common Support Troubleshooting
To ensure a seamless first day for the user, the following standard support verifications are documented:
* **Account Lockouts:** If the user enters the wrong password 3 times, the account flags as locked. In Active Directory Users and Computers (ADUC), navigating to the user's properties under the **"Account"** tab allows us to check **"Unlock account"** to restore access.
* **Group Policy Replication:** If the new network drive doesn't appear immediately on the user's machine, running the command `gpupdate /force` in the Windows Command Prompt forces the client to pull the latest directory changes.

---

## 💻 Technical Environment Summary
* **Directory Service:** Active Directory Domain Services (AD DS) / Microsoft Entra ID
* **Management Tools:** Active Directory Users and Computers (ADUC), Administrative Center, Windows Command Prompt.
* **Focus Skills:** 1st/2nd Level Ticket Resolution, Account Lifecycle, Permissions Management.

<div align="center">

# 🔐 Active Directory
## User Account Management & Password Reset

*A hands-on lab demonstrating core help-desk and system administration tasks in a Windows Server Active Directory environment — performed both through the **ADUC GUI** and **PowerShell**.*

<br>

![Platform](https://img.shields.io/badge/Platform-Windows%20Server-0078D6?style=for-the-badge&logo=windows&logoColor=white)
![Active Directory](https://img.shields.io/badge/Active%20Directory-FF6F00?style=for-the-badge&logo=microsoft&logoColor=white)
![PowerShell](https://img.shields.io/badge/PowerShell-5391FE?style=for-the-badge&logo=powershell&logoColor=white)

![Skill](https://img.shields.io/badge/Skill-Help%20Desk%20%7C%20SysAdmin-2EA043?style=flat-square)
![Level](https://img.shields.io/badge/Level-Entry%20to%20Junior-blue?style=flat-square)
![Status](https://img.shields.io/badge/Status-Complete-success?style=flat-square)

</div>

---

## 📑 Table of Contents

- [📋 Overview](#-overview)
- [🎯 Skills Demonstrated](#-skills-demonstrated)
- [🛠️ Environment & Tools](#️-environment--tools)
- [🧠 Background: The Concepts](#-background-the-concepts)
- [🚀 Walkthrough](#-walkthrough)
- [🏢 Real-World Scenarios](#-real-world-scenarios-this-solves)
- [🔒 Security Best Practices](#-security-best-practices-applied)
- [💡 Lessons Learned](#-lessons-learned)
- [🔭 Next Steps](#-next-steps--possible-extensions)

---

## 📋 Overview

Password resets and account lockouts are the **single most common ticket** a help-desk team handles. This project documents the full lifecycle of managing a user account in Active Directory — resetting a password, unlocking an account, forcing a password change at next logon, and verifying the result — using the exact tools a real IT support technician uses every day.

The lab was built in a domain called `thm.local`, with users organized into department-based Organizational Units (OUs): **IT, Management, Marketing, Research and Development, and Sales**.

> [!NOTE]
> Every task here maps directly to a real-world ticket. Doing it cleanly, documenting it, and being able to do it *via script* is the difference between *"I clicked a button"* and *"I understand identity management."*

---

## 🎯 Skills Demonstrated

| | Skill |
|---|---|
| 🗂️ | Navigating Active Directory and its Organizational Unit (OU) structure |
| 🔑 | Resetting user passwords (GUI **and** PowerShell) |
| 🔓 | Unlocking locked-out accounts |
| 🔁 | Forcing a password change at next logon (security best practice) |
| 🟢 | Enabling / disabling accounts (onboarding & offboarding) |
| ✅ | Verifying account state and reading account attributes |
| 🛡️ | Basic identity & access management (IAM) concepts |
| 📝 | Documentation and following a repeatable runbook |

---

## 🛠️ Environment & Tools

<div align="center">

| Component | Detail |
|:----------|:-------|
| **Operating System** | Windows Server (Domain Controller) |
| **Directory Service** | Active Directory Domain Services (AD DS) |
| **Domain** | `thm.local` |
| **Primary GUI Tool** | Active Directory Users and Computers (ADUC) |
| **Scripting** | PowerShell + ActiveDirectory module |
| **Sample Users** | Claire, Mary, Phillip (IT OU) |

</div>

---

## 🧠 Background: The Concepts

<details>
<summary><b>What is Active Directory? (click to expand)</b></summary>

<br>

**Active Directory (AD)** is Microsoft's directory service. Think of it as the central "phone book" and security gatekeeper for a Windows network — it stores every user, computer, and group, and it decides who can log in and what they're allowed to access.

</details>

<details>
<summary><b>What are Organizational Units (OUs)?</b></summary>

<br>

**Organizational Units (OUs)** are folders used to organize AD objects (usually by department). In this lab the OUs mirror a real company: IT, Management, Marketing, Research and Development, and Sales. OUs make it easy to apply policies and delegate admin rights to the right people.

</details>

<details>
<summary><b>Why do password resets land on the help desk?</b></summary>

<br>

Users forget passwords, accounts lock after too many failed attempts, and security policy may require periodic changes. An IT support technician needs to resolve these *quickly and securely* without compromising the account.

</details>

---

## 🚀 Walkthrough

### 🖱️ Part 1 — Reset a Password Using the GUI (ADUC)

1. Open **Active Directory Users and Computers** (`dsa.msc`).
2. In the left pane, expand the domain `thm.local` → expand the **THM** OU → click the **IT** OU.
3. In the right pane, locate the user (e.g., **Claire**).
4. **Right-click** the user → select **Reset Password**.
5. Enter a strong temporary password.
6. ✅ Check **"User must change password at next logon."**
7. Click **OK**.

> [!TIP]
> Always force the change at next logon — this ensures the *user*, not the admin, sets the final password. **You never want to know a user's password.**

<div align="center">

<img width="2045" height="968" alt="reset passsword GUI" src="https://github.com/user-attachments/assets/4f0a256a-f619-45a7-ba13-4518bc63f971" />


</div>

---

### 🔓 Part 2 — Unlock a Locked-Out Account (GUI)

If a user has entered the wrong password too many times, the account locks.

1. In ADUC, right-click the user → **Properties**.
2. Go to the **Account** tab.
3. Check **"Unlock account."**
4. Click **OK**.

<div align="center">

<img width="2048" height="968" alt="Unlock a locked out account" src="https://github.com/user-attachments/assets/2bfc8bbe-588d-4223-a637-46e31938c5ce" />


</div>

---

### ⚡ Part 3 — The Same Tasks in PowerShell

> [!IMPORTANT]
> This is where you show depth. The GUI solves one ticket — a script solves a hundred.

```powershell
# Load the Active Directory module (run as a Domain Admin / on the DC)
Import-Module ActiveDirectory
```

**Reset a user's password:**
```powershell
Set-ADAccountPassword -Identity "Claire" -Reset `
    -NewPassword (ConvertTo-SecureString -AsPlainText "TempP@ssw0rd!2026" -Force)
```

**Force the user to change it at next logon (best practice):**
```powershell
Set-ADUser -Identity "Claire" -ChangePasswordAtLogon $true
```

**Unlock a locked-out account:**
```powershell
Unlock-ADAccount -Identity "Claire"
```

**Find every locked-out account in the domain (proactive support):**
```powershell
Search-ADAccount -LockedOut | Select-Object Name, SamAccountName, LockedOut
```

**Disable / enable an account (offboarding & onboarding):**
```powershell
Disable-ADAccount -Identity "Phillip"   # e.g., employee leaves
Enable-ADAccount  -Identity "Phillip"   # e.g., returns / new hire activated
```

<div align="center">

<img width="2048" height="969" alt="same task in powershell" src="https://github.com/user-attachments/assets/4389a76a-f94e-4201-8ba7-457fc8565263" />


</div>

---

### ✅ Part 4 — Verify Your Work

> [!WARNING]
> Always confirm the change. **Never assume it worked.**

```powershell
Get-ADUser -Identity "Claire" -Properties LockedOut, PasswordExpired, PasswordLastSet |
    Select-Object Name, Enabled, LockedOut, PasswordExpired, PasswordLastSet
```

List every user in a specific OU to confirm you're in the right place:

```powershell
Get-ADUser -Filter * -SearchBase "OU=IT,OU=THM,DC=thm,DC=local" |
    Select-Object Name, SamAccountName, Enabled
```

<div align="center">

<img width="2048" height="968" alt="work verification " src="https://github.com/user-attachments/assets/9508c6c5-7d6c-4089-abc4-56ba994f1445" />

</div>

---

## 🏢 Real-World Scenarios This Solves

<div align="center">

| Scenario | Action |
|:---------|:-------|
| 😰 *"I'm locked out and have a meeting in 5 minutes!"* | `Unlock-ADAccount` |
| 🤔 *"I forgot my password."* | Reset + force change at next logon |
| 👋 New employee starting Monday | Create/enable account, set initial password |
| 📦 Employee leaving the company | `Disable-ADAccount` *(disable, don't delete)* |
| 🔍 Security audit: who's locked out? | `Search-ADAccount -LockedOut` |

</div>

---

## 🔒 Security Best Practices Applied

> [!CAUTION]
> The habits below are what separate a careful technician from a risky one.

- 🚫 **Never store or memorize a user's password** — set a temporary one and force a change at next logon.
- 🗃️ **Disable, don't delete** departing users' accounts — this preserves data and supports audits.
- 🔐 **Use strong temporary passwords** that meet the domain's complexity policy.
- ✅ **Verify after every change** rather than assuming success.
- 🎯 **Least privilege** — only admins who need reset rights should be delegated them on an OU.

---

## 💡 Lessons Learned

- The GUI is great for one-off tickets; **PowerShell is essential** the moment you touch more than one account.
- OU structure isn't just tidy — it's how you scope policies and delegate admin rights safely.
- Small habits (verify, force-change-at-logon, disable-don't-delete) build real trust as an admin.

---

## 🔭 Next Steps / Possible Extensions

```diff
+ This lab can grow into a much bigger portfolio piece:
```

- [ ] Bulk-create users from a CSV with `Import-Csv` + `New-ADUser`
- [ ] Write a self-service "reset & unlock" PowerShell function with parameters
- [ ] Create and link a **Group Policy Object (GPO)** (password policy / login banner)
- [ ] Delegate password-reset rights to a help-desk group on a single OU
- [ ] Add a PowerShell report that emails all locked-out accounts daily

---

<div align="center">

### 👤 Author

**Kirkland Metellus** — Aspiring System Administrator | IT Support

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](#)
[![GitHub](https://img.shields.io/badge/GitHub-Follow-181717?style=for-the-badge&logo=github&logoColor=white)](#)

<br>

*Built as part of my hands-on journey from IT support into system administration.*
*⭐ Feedback and connections welcome!*

</div>

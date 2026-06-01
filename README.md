# 🔐 Active Directory: User Account Management & Password Reset

> A hands-on lab demonstrating core help-desk and system administration tasks in a Windows Server Active Directory environment — performed both through the **Active Directory Users and Computers (ADUC)** GUI and through **PowerShell**.

![Platform](https://img.shields.io/badge/Platform-Windows%20Server-blue)
![Tool](https://img.shields.io/badge/Tool-Active%20Directory-orange)
![Scripting](https://img.shields.io/badge/Scripting-PowerShell-5391FE)
![Skill](https://img.shields.io/badge/Skill-Help%20Desk%20%7C%20SysAdmin-success)

---

## 📋 Overview

Password resets and account lockouts are the **single most common ticket** a help-desk team handles. This project documents the full lifecycle of managing a user account in Active Directory — resetting a password, unlocking an account, forcing a password change at next logon, and verifying the result — using the exact tools a real IT support technician uses every day.

The lab was built in a domain called `thm.local`, with users organized into department-based Organizational Units (OUs): **IT, Management, Marketing, Research and Development, and Sales**.

**Why it matters:** Every task here maps directly to a real-world ticket. Doing it cleanly, documenting it, and being able to do it *via script* is the difference between "I clicked a button" and "I understand identity management."

---

## 🎯 Skills Demonstrated

- Navigating Active Directory and its Organizational Unit (OU) structure
- Resetting user passwords (GUI **and** PowerShell)
- Unlocking locked-out accounts
- Forcing a password change at next logon (security best practice)
- Enabling / disabling accounts (onboarding & offboarding)
- Verifying account state and reading account attributes
- Basic identity & access management (IAM) concepts
- Documentation and following a repeatable runbook

---

## 🛠️ Environment & Tools

| Component | Detail |
|-----------|--------|
| **Operating System** | Windows Server (Domain Controller) |
| **Directory Service** | Active Directory Domain Services (AD DS) |
| **Domain** | `thm.local` |
| **Primary GUI Tool** | Active Directory Users and Computers (ADUC) |
| **Scripting** | PowerShell + ActiveDirectory module |
| **Sample Users** | Claire, Mary, Phillip (IT OU) |

---

## 🧠 Background: The Concepts

**Active Directory (AD)** is Microsoft's directory service. Think of it as the central "phone book" and security gatekeeper for a Windows network — it stores every user, computer, and group, and it decides who can log in and what they're allowed to access.

**Organizational Units (OUs)** are folders used to organize AD objects (usually by department). In this lab the OUs mirror a real company: IT, Management, Marketing, Research and Development, and Sales. OUs make it easy to apply policies and delegate admin rights to the right people.

**Why password resets land on the help desk:** Users forget passwords, accounts lock after too many failed attempts, and security policy may require periodic changes. An IT support technician needs to resolve these *quickly and securely* without compromising the account.

---

## 🚀 Walkthrough

### Part 1 — Reset a Password Using the GUI (ADUC)

1. Open **Active Directory Users and Computers** (`dsa.msc`).
2. In the left pane, expand the domain `thm.local` → expand the **THM** OU → click the **IT** OU.
3. In the right pane, locate the user (e.g., **Claire**).
4. **Right-click** the user → select **Reset Password**.
5. Enter a strong temporary password.
6. ✅ Check **"User must change password at next logon"** — this ensures the user, not the admin, sets the final password (you never want to know a user's password).
7. Click **OK**.

<img width="2045" height="968" alt="reset passsword GUI" src="https://github.com/user-attachments/assets/4302dae0-37db-4cd7-aea3-1d37805f02df" />


---

### Part 2 — Unlock a Locked-Out Account (GUI)

If a user has entered the wrong password too many times, the account locks.

1. In ADUC, right-click the user → **Properties**.
2. Go to the **Account** tab.
3. Check **"Unlock account. This account is currently locked out on this Active Directory Domain Controller."**
4. Click **OK**.

<img width="2048" height="968" alt="Unlock a locked out account" src="https://github.com/user-attachments/assets/ab31eaef-6105-49ac-b79c-c96783fe0563" />


---

### Part 3 — The Same Tasks in PowerShell

This is where you show depth. The PowerShell `ActiveDirectory` module lets you do everything the GUI does — but faster, repeatably, and at scale.

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

**Find every locked-out account in the domain (great for proactive support):**
```powershell
Search-ADAccount -LockedOut | Select-Object Name, SamAccountName, LockedOut
```

**Disable / enable an account (offboarding & onboarding):**
```powershell
Disable-ADAccount -Identity "Phillip"   # e.g., employee leaves
Enable-ADAccount  -Identity "Phillip"   # e.g., returns / new hire activated
```

<img width="2048" height="969" alt="same task in powershell" src="https://github.com/user-attachments/assets/dcbd17ce-073d-4c79-b1ad-c4514429c4ed" />


---

### Part 4 — Verify Your Work

Always confirm the change. Never assume.

```powershell
Get-ADUser -Identity "Claire" -Properties LockedOut, PasswordExpired, PasswordLastSet |
    Select-Object Name, Enabled, LockedOut, PasswordExpired, PasswordLastSet
```

You can also list every user in a specific OU to confirm you're working in the right place:

```powershell
Get-ADUser -Filter * -SearchBase "OU=IT,OU=THM,DC=thm,DC=local" |
    Select-Object Name, SamAccountName, Enabled
```

<img width="2048" height="968" alt="work verification " src="https://github.com/user-attachments/assets/42f3e33c-e3f2-46a2-a143-a6d748012805" />


---

## 🏢 Real-World Scenarios This Solves

| Scenario | Action |
|----------|--------|
| "I'm locked out and have a meeting in 5 minutes!" | `Unlock-ADAccount` |
| "I forgot my password." | Reset + force change at next logon |
| New employee starting Monday | Create/enable account, set initial password |
| Employee leaving the company | `Disable-ADAccount` (disable, don't delete — preserves data & audit trail) |
| Security audit: who's locked out right now? | `Search-ADAccount -LockedOut` |

---

## 🔒 Security Best Practices Applied

- **Never permanently store or memorize a user's password** — always set a temporary one and force a change at next logon.
- **Disable, don't immediately delete** departing users' accounts — this preserves data and supports investigations/audits.
- **Use strong temporary passwords** that meet the domain's complexity policy.
- **Verify after every change** rather than assuming success.
- **Use the principle of least privilege** — only admins who need to reset passwords should be delegated that right on an OU.

---

## 💡 Lessons Learned

- The GUI is great for one-off tickets; **PowerShell is essential** the moment you need to do something to more than one account.
- OU structure isn't just tidy — it's how you scope policies and delegate admin rights safely.
- Small habits (verify, force-change-at-logon, disable-don't-delete) are what separate a careful technician from a risky one.

---

## 🔭 Next Steps / Possible Extensions

This lab can grow into a much bigger portfolio piece:

- [ ] Bulk-create users from a CSV with `Import-Csv` + `New-ADUser`
- [ ] Write a self-service "reset & unlock" PowerShell function with parameters
- [ ] Create and link a **Group Policy Object (GPO)** (e.g., password policy or login banner)
- [ ] Delegate password-reset rights to a help-desk group on a single OU
- [ ] Add a PowerShell report that emails all locked-out accounts daily

---

## 👤 Author

**[Your Name]** — Aspiring System Administrator | IT Support
📍 [Your City] · 🔗 [LinkedIn] · 💻 [GitHub]

> *Built as part of my hands-on journey from IT support into system administration. Feedback and connections welcome!*

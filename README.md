# Active Directory Installation and Setup Lab

## Prerequisites

- Azure Subscription
- A VM running Windows Server to act as the Domain Controller
- A virtual network and a subnet
- A VM to act as the client computer

---

## Initial Setup

This section details the step-by-step process of setting up a "corporate" network in the Azure cloud that incorporates multiple PCs and uses Active Directory to manage permissions and shares securely.

1. **Create a Resource Group and Virtual Network** on Azure.
2. **Create a Domain Controller VM named `DC-1`**:
   - Ensure the VM is on the same Virtual Network and region.
   - Select **Windows Server 2022** for the OS Image.
3. **Create the Client VM**:
   - Add to the same Resource Group and Virtual Network.
   - Use **Windows 10 Pro** OS image.
4. **Assign a static private IP to DC-1**:
   - Go to `DC-1 > Network > Network Settings > IP Configuration`.
   - Change allocation to Static and save.
5. **Disable Windows Firewall on DC-1**:
   - RDP into DC-1.
   - Run `wf.msc` and turn firewall off in properties.
6. **Set Client VM DNS to DC-1's private IP**:
   - Go to `Client VM > Network Settings > DNS servers`.
   - Set to Custom and enter DC-1’s IP.
   - Restart the Client VM.
7. **Ping DC-1 from Client VM** to confirm connectivity:
   - Use `ipconfig /all` to verify DNS.
   - Ping DC-1's private IP.

---

## Installing Active Directory

1. RDP into DC-1.
2. Open **Server Manager** > Add roles and features.
3. Select **Active Directory Domain Services** and click **Add Features**.
4. Accept restart if prompted and install.
5. Promote the server to a domain controller:
   - Create a **new forest**, name it, and set a **DSRM password**.
   - Complete installation and let the server reboot.

---

## Creating Organizational Units (OUs)

1. Open **Administrative Tools** > **Active Directory Users and Computers**.
2. Under your domain:
   - Create OUs: `_EMPLOYEES` and `_ADMINS`.
3. In `_ADMINS`, create a user:
   - Name: Jane Doe
   - Username: `jane_admin`

---

## Adding Client VM to the Domain

1. RDP into the Client VM.
2. Go to **System Settings** > **Rename this PC (Advanced)**.
3. Click **Change**, select **Domain**, enter domain name.
4. Use credentials for a domain user (`jane_admin`) to join.
5. After reboot, verify domain join on DC-1 in **Active Directory Users and Computers**.
6. Create a new OU `clients` and move Client VM into it.

---

## Enable RDP Access for Domain Users

1. On Client VM, go to **System Settings** > **Remote Desktop**.
2. Click **Select users that can remotely access this PC**.
3. Add: `Domain Users`.
4. You can now log on as any domain user.

---

## Create Additional Users via PowerShell

1. On DC-1, open **PowerShell ISE as Administrator**.
2. Run the script from:
   [Generate-Names-Create-Users.ps1](https://github.com/joshmadakor1/AD_PS/blob/master/Generate-Names-Create-Users.ps1)
3. Observe new user accounts being created.
4. Use one to log into the Client VM.

---

## Simulate Account Lockouts

1. Run `gpmc.msc` to open **Group Policy Management**.
2. Navigate to: `Forest > Domains > mydomain.com`.
3. Create and link a new GPO. Edit it.
4. Set:
   - `Computer Configuration > Policies > Windows Settings > Security Settings > Account Policies > Account Lockout Policy`
   - Threshold: **3 attempts**
   - Duration and reset time: **10 minutes**
5. Run `gpupdate /force`.
6. Attempt bad logins to trigger lockout.
7. On DC-1, unlock the account:
   - Go to user > Properties > Account > Unlock account.

---

✅ **Lab Complete**: You now have a fully functional Active Directory environment with security policies and user access controls configured.

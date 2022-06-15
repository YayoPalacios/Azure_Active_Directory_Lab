# Lab - Build an Active Directory environment with Microsoft Azure

> This project was inspired by The Cyber Mentor’s [video](https://www.youtube.com/watch?v=xftEuVQ7kY0&t=555s) where Heath configures a local AD environment which I transposed to MS Azure ruling out any performance issues.
> 

<br>

## What is Active Directory?

Active Directory (AD) is a database and set of services that connect users with the network resources they need to get their work done.

The database (or directory) contains critical information about your environment, including what users and computers there are and who is allowed to do what. For example, the database might list 100 user accounts with details like each person’s job title, phone number and password. It will also record their permissions.

These services control much of the activity that goes on an IT environment.

<br>

## **How does AD work?**

The main Active Directory service is Active Directory Domain Services (AD DS), which is part of the Windows Server operating system. The servers that run AD DS are called domain controllers (DC’s). Organizations normally have multiple DC's, and each one has a copy of the directory for the entire domain. Changes made to the directory on one domain controller — such as password update or the deletion of a user account — are replicated to the other DC’s so they all stay up to date. Desktops, laptops and other devices running Windows (rather than Windows Server) can be part of an Active Directory environment but they do not run AD DS. AD DS relies on several established protocols and standards, including LDAP (Lightweight Directory Access Protocol), Kerberos and DNS (Domain Name System).

It’s important to understand that Active Directory is only for on-premises Microsoft environments. AD and Azure AD are separate but can work together to some degree if your organization has both on-premises and cloud IT environments (a hybrid deployment).

<br>

<p align="center">
<img src="https://imgur.com/xAOv7Rw.png" height="50%" width="50%" alt="AD"/>
</p>

[Source](https://www.quest.com/solutions/active-directory/what-is-active-directory.aspx#:~:text=Active%20Directory%20(AD)%20is%20a,who%27s%20allowed%20to%20do%20what.)

<br>

## Important:

We’re intentionally setting up a vulnerable ecosystem, some of these practices are simply unrealistic in a working real world solution.
The whole intention is getting ourselves a practice ground to get our hands dirty and gain some knowledge with AD’s numerous vulnerabilities and defensive methodologies.

<br>

<p align="center">
<img src="https://imgur.com/XYhQ43l.png" height="80%" width="80%" alt="PPT"/>
</p>

<br>

## Project summary:

<p align="center">
<img src="https://imgur.com/xHhg3Va.png" height="85%" width="85%" alt="AD"/>
</p>

<br>

- First, we’ll create an Azure (free) subscription that will give you 200 USD worth of credits.

<br>

- We’ll generate a resource group to easily contain and manage our project components.

<br>

<p align="center">
<img src="https://imgur.com/SCCyt3K.png" height="80%" width="80%" alt="RG"/>
</p>

<br>

- Then, we’re going to create our own virtual network which we’ll use to link all our resources.

<br>

<p align="center">
<img src="https://imgur.com/hJTWpgJ.png" height="80%" width="80%" alt="Vnet"/>
</p>

<br>

<p align="center">
<img src="https://imgur.com/1GISMQh.png" height="80%" width="80%" alt="RG"/>
</p>

<br>

- Next, we’ll spin up our first VM that we’re going to configure as our Domain Controller. For this lab, we’re using a **Windows Server 2019** image from MS Azure.

<br>

<p align="center">
<img src="https://imgur.com/yUd2Rgx.png" height="80%" width="80%" alt="VMDC"/>
</p>

<br>

- We’ll connect to our VM using its IP address via [Remote Desktop](https://support.microsoft.com/en-us/windows/how-to-use-remote-desktop-5fe128d5-8fb1-7a23-3b8a-41e636865e8c).

<br>

<p align="center">
<img src="https://imgur.com/ySgd9bC.png" height="80%" width="80%" alt="VMDC"/>
</p>

<br>

- I’m installing the [BgInfo](https://docs.microsoft.com/en-us/sysinternals/downloads/bginfo) utility on our machines due to illustrative purposes. This will display relevant information about our Windows systems.

<br>

<p align="center">
<img src="https://imgur.com/qyRY773.png" height="50%" width="50%" alt="BGinfo"/>
</p>

<br>

- At this point, our VM should look as follows.

<br>

<p align="center">
<img src="https://imgur.com/VtiWA8W.png" height="80%" width="80%" alt="RG"/>
</p>

<br>

- Now, we’ll go ahead and configure our services to promote this machine to a Domain Controller.

<br>

- Launch **Server Manager** and click on the **Add roles and features** option. Under Installation Type, we’ll pick **Role-based or feature-based installation**.

<br>

- Then, on the **Server Roles** tab, check the **Active Directory Domain services** checkbox. Click on Next and go through the remaining installation dialogs until it’s completed.

<br>

<p align="center">
<img src="https://imgur.com/tAx8BSo.png" height="80%" width="80%" alt="Services"/>
</p>

<br>

<p align="center">
<img src="https://imgur.com/5DAjWzg.png" height="80%" width="80%" alt="DC"/>
</p>

<br>

<p align="center">
<img src="https://imgur.com/YxeIAKr.png" height="80%" width="80%" alt="DC"/>
</p>

<br>

<p align="center">
<img src="https://imgur.com/UJpASZo.png" height="80%" width="80%" alt="Services2"/>
</p>

[Here’s an in-depth guide on setting up a Domain controller](https://www.manageengine.com/products/active-directory-audit/kb/how-to/how-to-setup-a-domain-controller.html)

<br>

- Keep in mind you’ll need to restart the VM a couple of times throughout the setup. It should look something like this once we’re done with the operation.

<br>

<p align="center">
<img src="https://imgur.com/4GN8PvW.png" height="60%" width="60%" alt="BGinfo"/>
</p>

<br>

- The next step is to configure **Active Directory Certificate Services**. This process is very similar to the steps listed above (Server Manager > Add roles and features > Check Active Directory Certificate Services).

<br>

- Now, we’re creating a few AD user accounts applying the following PS script. We can always approach this manually however, I thought this would be a nice addition to put our scripting skills to practice in a very simplistic way.

<br>

- Prior to executing it, make sure to run the following cmdlet:

<br>

```powershell
Set-ExecutionPolicy Unrestricted
```

#### [Azure_Active_Directory_Lab/create_users.ps1](https://github.com/YayoPalacios/Azure_Active_Directory_Lab/blob/main/create_users.ps1)

<br>

```powershell
$PASSWORD_FOR_USERS   = "Password1"
$USER_FIRST_LAST_LIST = Get-Content .\names.txt
# ------------------------------------------------------ #

$password = ConvertTo-SecureString $PASSWORD_FOR_USERS -AsPlainText -Force
New-ADOrganizationalUnit -Name _USERS -ProtectedFromAccidentalDeletion $false

foreach ($n in $USER_FIRST_LAST_LIST) {
    $first = $n.Split(" ")[0].ToLower()
    $last = $n.Split(" ")[1].ToLower()
    $username = "$($first.Substring(0,1))$($last)".ToLower()
    Write-Host "Creating domain user account: $($username)" -BackgroundColor Black -ForegroundColor Cyan
    
    New-AdUser -AccountPassword $password `
               -GivenName $first `
               -Surname $last `
               -DisplayName $username `
               -Name $username `
               -EmployeeID $username `
               -PasswordNeverExpires $true `
               -Path "ou=_USERS,$(([ADSI]`"").distinguishedName)" `
               -Enabled $true
}
```

<br>

<p align="center">
<img src="https://imgur.com/hbfWxkf.png" height="80%" width="80%" alt="PS"/>
</p>

<br>

<p align="center">
<img src="https://imgur.com/tU3oHs7.png" height="80%" width="80%" alt="AD"/>
</p>

<br>

- We’re setting up the Service Principal Names for our **SQLService** account using the **setspn** utility.

<br>

## What is setspn in Active Directory?

Reads, modifies, and deletes the Service Principal Names (SPN) directory property for an Active Directory service account. You use SPNs to locate a target principal name for running a service. You can use **setspn** to view the current SPNs, reset the account's default SPNs, and add or delete supplemental SPNs.

[Setspn | Microsoft Docs](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/cc731241(v=ws.11))

<br>

<p align="center">
<img src="https://imgur.com/B1MsbUh.png" height="80%" width="80%" alt="Setspn"/>
</p>

<br>

<p align="center">
<img src="https://imgur.com/MSPeXJd.png" height="80%" width="80%" alt="Setspn"/>
</p>

<br>

- Now, we’re spinning up a couple of user VM’s for our environment, see specs below.

<br>

<p align="center">
<img src="https://imgur.com/zDREyu3.png" height="80%" width="80%" alt="VM1"/>
</p>

<br>

<p align="center">
<img src="https://imgur.com/nOARnU1.png" height="80%" width="80%" alt="VM's"/>
</p>

<br>

- We’re joining both user machines to our Domain, in my case I named it HOUSEOFSTARK.local.

<br>

<p align="center">
<img src="https://imgur.com/o4Maa8U.png" height="80%" width="80%" alt="Join"/>
</p>

<br>

<p align="center">
<img src="https://imgur.com/NN5Qw44.png" height="80%" width="80%" alt="Join2"/>
</p>

<br>

<p align="center">
<img src="https://imgur.com/yEusWem.png" height="85%" width="85%" alt="VM's"/>
</p>

<br>

- We’re now adding our two main user accounts to our local **Administrators** group via **Computer Management**. Don’t forget we need to log in as a domain administrator to set these permissions on both of our VM’s.

<br>

<p align="center">
<img src="https://imgur.com/Qt7STjn.png" height="80%" width="80%" alt="compmgmt"/>
</p>

<br>

- Our AD ecosystem should be all set now.

<br>

<p align="center">
<img src="https://imgur.com/hDZOM6O.png" height="85%" width="85%" alt="AD_Env"/>
</p>

<br>

- All we need to do know is go through a quick Linux install (Ubuntu Server 20.04 LTS). The initial plan was to spin up a Kali machine however, I noticed Offensive Security retired the image from the Azure Marketplace.

<br>

- We’re using [PuTTY](https://putty.org/) to connect to our Ubuntu machine using SSH.

<br>

- Make sure to install both **netdiscover** and **nmap** utilities.

<br>

- We’ll run [netdiscover](https://helpmanual.io/man8/netdiscover/) to find our hosts.

<br>

```bash
sudo netdiscover -r 10.0.1.4/24
```

<br>

<p align="center">
<img src="https://imgur.com/g3bbVhv.png" height="85%" width="85%" alt="netdiscover"/>
</p>

<br>

- Here’s an [nmap](https://linux.die.net/man/1/nmap) scan of our domain.

<br>

<p align="center">
<img src="https://imgur.com/0iYNXLi.png" height="85%" width="85%" alt="RG"/>
</p>

<br>

Hope you enjoyed!

<br>

## Further learning:

To learn more about these concepts, you can check out the following articles:

- [https://adsecurity.org/](https://adsecurity.org/)
- [https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/active-directory-domain-services](https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/active-directory-domain-services)
- [https://www.kali.org/tools/netdiscover/](https://www.kali.org/tools/netdiscover/)
- [https://www.kali.org/tools/nmap/](https://www.kali.org/tools/nmap/)

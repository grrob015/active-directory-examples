# active-directory-examples
Showing the functionality of active directory with file sharing, user creation and organization, and more.

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)

## Active Directory Domain Services Functionalities

In my [previous demonstration](https://github.com/grrob015/active-directory-setup) where I set up Active Directory Domain Services in Azure VMs, we saw glimpses of the power of Active Directory in some of the settings that we configured in the installation (especially if you've seen my [osTicket demonstration](https://github.com/grrob015/osticket-settings)). Now that we have a fake company with an admin and ten employees, let's simulate some examples of things that might happen on the job!

## Group Policy and Account Lockouts

First, Jane needs to enable account lockouts on the company domain. Otherwise, hackers could guess passwords indefinitely without consequence and stealing Big Plastic Company's secrets would only become a matter of time. (⚠️ This is not a cybersecurity course! Account lockouts are just a great segway into **group policy** and **password resets**!) To do this, log into your domain controller as Jane Doe, our domain admin. Right click the Windows icon in the bottom left corner of your screen (or press Windows + R) to open run. Type `gpmc.sc` into Run and press Enter to bring up the Group Policy Management Console:


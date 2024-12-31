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

## Configuring Group Policy and Account Lockouts

First, Jane needs to enable account lockouts on the company domain. Otherwise, hackers could guess passwords indefinitely without consequence and stealing Big Plastic Company's secrets would only become a matter of time. (⚠️ This is not a cybersecurity course! Account lockouts are just a great segway into **group policy** and **password resets**!) To do this, log into your domain controller as Jane Doe, our domain admin. Right click the Windows icon in the bottom left corner of your screen (or press Windows + R) to open run. Type `gpmc.sc` into Run and press Enter to bring up the Group Policy Management Console. Expand the dropdown menus until you can see your company's domain:

![1  group policy management center](https://github.com/user-attachments/assets/06ef9d5c-39a2-474e-827f-58b6e0a4cc7d)

Right click the "Default Domain Policy" and select "Edit". In the following dropdown menus, open these folders:

1. **Computer Configuration**
2. **Policies**
3. **Windows Settings**
4. **Security Settings**
5. **Account Policies**
6. Then, click on **Account Lockout Policy**

![2  matryoshka folders](https://github.com/user-attachments/assets/009829fb-8239-46d7-8870-0a84cc1eaf63)

You will see four settings that can be configured for account lockouts on this domain:

- **Account lockout duration**: how long the account will be locked out for once the threshold of failed attempts is met.
- **Account lockout threshhold**: how many failed attempts are allowed before the account is locked out.
- **Allow Administrator account lockout**: yes/no option that says whether or not admins like Jane can get locked out (we will be leaving this one disabled for safety).
- **Reset account lockout counter after**: how long you must wait if there are failed attempts (but the threshold isn't met) for the threshold counter to reset.

Double click each policy item to edit them to your liking, and click the "Explain" tab at the top for more information and integer formatting information. Here are my settings, just for an example:

![3  example policies](https://github.com/user-attachments/assets/b401a40e-8838-48cc-b40d-3a8d76baf64e)

Now that we have a new group policy, we can either wait for it to update automatically (up to ~90 minutes) or force an update so we can test it faster. Let's log into our client as Jane and open an administrator Command Prompt to force an update faster.

![4  gpudateforce](https://github.com/user-attachments/assets/c9502d28-679c-418a-8034-45dca96ee77d)

## Account Lockouts and Pawword Resets in Action

Now it's time to see it in action! Log out from the client and choose an unlucky user that [we created](https://github.com/grrob015/active-directory-setup?tab=readme-ov-file#creating-normal-employees) to lockout. I'll say that Ken (username `ken.xibu`) forgot his password on his first day of work and didn't realize that trying to guess it would cause issues! Fail to log into you user's account enough times to trigger your account lockout policy on the client virtual machine. 

![5  good error](https://github.com/user-attachments/assets/abcb5d7d-49d7-4b8a-8ffe-ae10cebaae22)
<!-- my desktop background is https://wall.alphacoders.com/big.php?i=1337140 if you're curious -->

Distraught, Ken comes over to Jane's desk and asks her to unlock his account for him. How does Jane do it? First, log into the domain controller as Jane and open the "Active Directory Users and Computers" application. From the Start menu, you can select it from the "Windows Administrative Tools" dropdown menu or search for "users", both methods work.

Next, double click on Ken's account or right-click it and select "Properties". In the "Account" Menu, you will see a box you can tick to unlock an account that is currently locked out. 

![6  unlocking ken](https://github.com/user-attachments/assets/768242e1-b378-43dc-83e5-b3dd9161ae80)

Now, try logging on as Ken again on your client virtual machine. This time, he should be allowed to, since a domain admin unlocked his account!

![7  ken logging in](https://github.com/user-attachments/assets/8ddaeb67-6e21-4e76-a395-7012174f6c21)

<!-- Starting to think I made bad choices hiring these people... -->
Choose another one of your accounts and have them forget their password as well. Lad (username `lad.vone`) comes into work on his first day a little after Ken and notices that Ken got locked out for guessing too many times, so Lad walks over to Jane's desk right away and says he can't remember his password. How does Jane reset a user's password?

Start by logging into your domain controller as Jane if you don't already have it open. Find the account in "Active Directory Users and Computers" you need to password reset and right click it, selecting "Reset Password...". A menu will pop up that configures password settings and also allows you to unlock an account if necessary. Lad can now log in with his new password and Jane can even force him to change it if that option is selected (more similarities with [osTicket options](https://github.com/grrob015/osticket-examples?tab=readme-ov-file#john-forgets-his-password)). 

![8  lad password reset](https://github.com/user-attachments/assets/d6168140-a28e-41a2-a4b6-1e3637a301d1)

💡 If a company has hundreds or even thousands of users, finding the right one could be a challenge. Right click the folder you want to search and select "Find..." to search users by name or description.

## Enabling and Disabling Accounts

Jane gets a phone call from me (the CEO) that says that Jase Jex (username `jase.jex`) has quit and that her account needs to be disabled, because we don't want her accessing company data anymore. Jane logs into the domain controller and opens "Active Directory Users and Computers", finds Jase's account, right clicks it, and selects "Disable Account". A little down arrow will appear next to the user icon to signify a disabled account.

![10  jase disabled](https://github.com/user-attachments/assets/633f20c4-0565-4980-9987-9982d915f1de)

If Jase (hypotheitcally) gets an offer from another company to sell Big Plastic Company's data to them, and she tries to log in, she will get this error message:

![11  jase DENIED](https://github.com/user-attachments/assets/6c2c07ad-197c-4e96-b901-a1ed9f7f8bf0)

💡 At any point, a disabled account can be enabled again by right-clicking it and selecting "Enable Account".






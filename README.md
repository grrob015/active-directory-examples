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

## Network File Shares and Permissions

In this section, we'll learn how to set up network file sharing so that users can collaborate on projects with shared files. This mimics what google drive would do, except for the fact that network file sharing is centrally managed and we don't technically even need internet access to have it working. As long as the local network is still up, file sharing will still function. 

To see how network file sharing might work, log into your domain controller as Jane if you don't have that open. Navigate to your `C:\` drive and create four folders:

- **read-access** (All employees will be able to read this but not edit it. Useful for things like meeting notes or company literature.)
- **write-access** (All employees will be able to read and write in this folder. Useful for shared projects with multiple users collaborating.)
- **admin-access** (Only admins can read and write in this folder. Good for sensitive/technical information.)
- **engineering** (Only engineers can read and write in this folder. Good for sensitive/technical data specific to the engineering wing.)

![12  four new folders](https://github.com/user-attachments/assets/fcad3d3c-3c0e-4dc1-af0b-fca6c8986c1a)

Now, we'll need to set the permission levels for the specific groups that we want to have access to this. In order to change permissions:

1. Right click a folder and select "Properties".
2. Go to the "Sharing" tab in folder properties.
3. Click the "Share..." button to bring up the "Network Access Menu".
4. Type the group you want to add and use the dropdown menu to select a level of permissions.

![13  how to enable sharing and permissions](https://github.com/user-attachments/assets/b3fd0f61-263c-4bdd-8001-e10f1e83ed6e)

For the first three folders:

1. For `read-access` - Add **Domain Users** as a group and give them **Read** access.
2. For `write-access` - Add **Domain Users** as a group and give them **Read/Write** access.
3. For `admin-access` - Add **Domain Admins** as a group and give them **Read/Write** access.

We will make an engineering wing and give engineers as an object specific permissions later. 

For now, create a notepad file in each of the three folders with some text on it, then log into your client as different users and see what you can do. For example, if we log into our client as non-admin employee Ken, and click "Network" on the sidebar in File Explorer, we _might_ get an error.
<!-- Fairly sure this was caused by the windows setup popup that says "do you want this computer to be discoverable by other pcs on the network??" and I usually select no. -->

![14  error bad](https://github.com/user-attachments/assets/2a22f046-6980-48c4-9c7e-4497a8e87341)

## Fixing the Network

Ken calls Jane over to his computer and shows Jane the error. Fortunately, with Active Directory services, this is an easy fix. She closes the error and click the header the says "Click to change..." and selects "Turn on network discovery and file sharing"

![15  jane clicks](https://github.com/user-attachments/assets/38fe9d1d-963c-46bb-bb33-79321888fb1d)

She then puts her administrator credentials into the popup.

![16  jane fixes problems](https://github.com/user-attachments/assets/120fb176-0f1a-4c11-8087-cf09e74806db)

And since this is a work network, she clicks the network that turns on file sharing for only private networks.

![17  jane clicks v2](https://github.com/user-attachments/assets/2061b542-15d0-4f17-bdc9-c4926c3e355c)

And that's all there is to it! Setting up Active Directory can be a hassle, but in moments like this when changing settings just requires a set of credentials, its usefulness shines through.

## Accessing the Domain Controller's Files

Now that he has file sharing enabled, Ken can see the folders we created. In order to do so, click the address bar and type `\\domain-controller-name`, where "domain-controller-name" is the name of the virtual machine we promoted to a domain controller in Azure. For me, it would be `\\dc-1`. Now we can see the files and read them!

💡 In the sidebar, right-click the "dc-1" folder and select "Pin to Quick Access" so you don't have to type the name every time!

![18  kens view fixed](https://github.com/user-attachments/assets/0cd70f97-00ea-474c-905a-e8ad128af4c6)

As you can see, Ken can see all of our shared folders, but if he tries to open admin access, he gets this error.  

![18 5 kens new adventure](https://github.com/user-attachments/assets/675f74ea-ee59-4bf1-b790-10e7fbd6751d)

If he tries to mess with the company rules in the "read-access" folder, he gets this error:

![19  kens new rules](https://github.com/user-attachments/assets/a6a96c19-16be-4229-b5f7-25f9282f195c)

In the "write-access" folder, though, he has a lot more freedom. Saving the document doesn't trigger an error (notice the lack of an asterisk in the file name). 💡 In Windows, when a file has been modified but not saved, the file name will have an asterisk appended to the front of it.

![20  kens notes](https://github.com/user-attachments/assets/2ad48e85-6ae3-4bde-ac4e-fa6f66fecf41)

## Creating the Engineering Wing

Now it's time to work with the last folder, "engineering". In order to create a group of engineers, log into your domain controller as Jane if you don't have that open. Open "Active Directory Users and Computers", and within the "_EMPLOYEES" Organizational Unit, create a group called "Engineers".

![21  new engineers](https://github.com/user-attachments/assets/e6cfc791-be9c-4a1e-9b56-77d650e0dc47)

Go back to your "engineering" folder and then give the object "Engineers" read/write permission on the folder.

![22  fast permissions version](https://github.com/user-attachments/assets/02639742-ce84-4bca-9451-a06ef5c6fe27)

Open "Active Directory Users and Computers" once again and double click your "Engineers" security group. Go to the "Members" tab and click "Add", then choose one or two users to add to the security group.

![23  adding users](https://github.com/user-attachments/assets/bd50c0c7-961e-447d-8ef4-63277e8b3c21)

Once this is done, you can log out of your domain controller for good.

## Viewing the Engineering Files

Choose both a non-engineer employee and an engineer employee to log in as. Remote desktop into your client virtual machine and see which one can view and edit the engineering folder. Only users who belong to the security group "Engineers" should be able to see and access it! (By default, the owner of a file or folder has read/write permissions on it, so don't choose Jane!)

Ken's network folder and access:

![24  kens view](https://github.com/user-attachments/assets/512bbe47-7ae5-438c-96fd-e43c259e3f03)

Engineer Lad's folder and access: 

![25  lad's view](https://github.com/user-attachments/assets/7691590e-e696-4f60-a055-1bb1dd64542e)

## Final Remarks

This concludes the Active Directory Domain Services Demonstration. We worked through and showed:

- Account Lockouts
- Group Policy Management
- Password Resets
- Enabling/Disabling Accounts
- Network File Sharing
- Security Groups

There is so much more you can do with Active Directory and so many more layers of detail even to the things that we touched upon, but this concludes the scope of my demonstration. Perhaps this could be a jumping-off point for research of your own. Thank you for reading!




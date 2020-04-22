.. title:: Nutanix Flow 200 UK HOL



.. _idbased:

---------------------------------
Identity Based Microsegmentation
---------------------------------

*The estimated time to complete this lab is 100 minutes.*

++++++++++++
Overview
++++++++++++

The VDI Policy is based on identity-based categorization of the VDI VMs using Active Directory group membership.
Configuring VDI policy includes adding an Active Directory domain that is used for the ID firewall (ID Based Security) and configuring a service account for the domain.

++++++++++++++++++
ID Based Security
++++++++++++++++++

ID firewall is an extension to Flow that allows you to write security policies based on users
and groups in an Active Directory domain in which your VDI VMs are attached. When using
ID firewall, you can import groups from Active Directory into Prism Central as categories (in the category key ADGroup), and then write policies around these categories as you would any for other category.
A new type of policy has been added for this purpose - the VDI Policy. ID Firewall takes care of automatically placing VDI VMs in the appropriate categories on detecting user logons into the VM hosted on Nutanix infrastructure associated with Prism Central, thus allowing user and group based enforcement of Flow policies.

++++++++++++++++++++++
Creating a VDI Policy
++++++++++++++++++++++

ID firewall integrates Nutanix Flow with Microsoft Active Directory (AD), such that groups in AD can be imported into Prism Central as categories.
These imported categories can then be used in the VDI policy as target groups, inbound traffic, and outbound traffic.
Prism Central automatically places VMs inside the imported AD group categories when user logons are detected on VMs that are part of the Active Directory domain and also present on Nutanix managed clusters, thus applying security policies based on user group membership.

.. note::

  Flow ID firewall does not detect user logoffs, the policy applied to a VM is kept applied until next user logon on the same VM.

  - VMs with an AppType category assigned does not get categorized by ID Based Security. This is by design.

  - If not already available, configure an Active Directory domain that is used for ID firewall.

  - Configure a service account with required configuration for the Active Directory domain

  - The registered Prism Element clusters must be running AOS version 5.17 or later

  - The registered Prism Element clusters must be running AHV version 20190916.114 or later

#. In the Security Policies dashboard (see "Security Policies Summary View" in the Prism Central Guide) and click Create Security Policy. Select Secure VDI Groups (VDI Policy) and click Create.

   You can create only one VDI policy for securing applications through ID Firewall. The Secure a VDI Environment page is displayed.

#. On the Define Policy tab, do the following in the indicated fields, and then click Next:

- The Policy Name and Purpose fields are auto-populated.

- Optionally, in the Advanced Configuration section, select the Allow radio button to allow IPv6 traffic. The policy rules apply to IPv4 traffic only and all IPv6 traffic is blocked by default.

.. note::

  If you choose to block IPv6 traffic, the traffic will reamin blocked even in monitoring mode.

- Optionally, click the toggle button against Policy Hit Logs to log traffic flow hits on the policy rules.

You can configure syslog monitoring for the policy hit logs for Flow, see "Configuring Syslog Monitoring" in the Prism Central Guide for details.

.. note::

  Policy hit logs are not generated if both source and destination are in inbound or outbound category.


#. In the Secure AD Groups tab, do the following in the indicated fields and click Next.

- For Inbound Traffic, click + Add Source and enter the category or subnets that the VDI group can receive the traffic from, as the source.

- For each VDI ADGroup, click +Add AD Group to select the AD groups (categorized VDI VMs) that you want to secure. You can click Import all AD Groups to add all imported ADGroup categories to the VDI policy.

- For Outbound Traffic, click + Add Destination and enter the category or subnets that the VDI group can send the traffic to, as the destination.

.. note::

  VDI Policy does not support visualisation

.. figure:: images/1.png

#. Do one of the following:
- Click Apply Now to apply the VDI Policy.

- Click Save and Monitor to save the configuration.

You can switch between the monitoring and applied states on the Security Policies page and clicking the appropriate option in the Actions menu.

+++++++++++++++++++++++++++++++++++++++++++++
Configuring Active Directory Domain Services
+++++++++++++++++++++++++++++++++++++++++++++

Active Directory Domain Services configuration is used to import user groups for identity based security policies.

**Before you begin**

- Microsegmentation must be enabled to be able to use the ID Firewall feature, see Enabling Microsegmentation.

- For supported AHV and AOS versions, see Prism Central Release Notes.\


Active Directory Requirements:
...............................

- Minimum supported domain functional level in Active Directory is Windows Server 2008 R2.

- ID Firewall checks the membership of Security Groups only, Distribution Groups are not supported.

- NTP must be configured in Active Directory.

- DNS must be configured if you want to use host name for domain controllers.

#. Log on to the Prism Central web console.

#. Click the collapse menu ("hamburger") button on the left of the main menu and then select **Prism Central Settings** to display the Settings page.

#. Click **ID Based Security** from the Settings menu (on the left).

The **ID Based Security** page is displayed. This page allows you to **Add New Domain** or use an **Existing AD**.

4. If you select **Use Existing AD** in step 3, do the following in the indicated fields:

- Click the **Manually Add Domain Controller** button, then click **+ Domain Controller**.

- Enter the **IP Address** or **Host name** of the domain controllers that you want to monitor for user logons events. You must add all the domain controllers associated with your Active Directory manually.

Click :fa:`plus-circle` and add each domain controller individually, then click the blue check mark icon to save.

.. note::

  DNS must be configured on the cluster for the host name option to work.
  CAUTION: Do not use the Domain Admin account as the service account considering the security best practices.
  Create a new domain user and grant it required permissions as described in Configure Service Account for ID Firewall.

5. If you select **Add New Domain** in step 3, a set of fields is displayed. Do the following in the indicated fields:

- **Name**: Enter a directory name.

  This is a name you choose to identify this entry; it need not be the name of an actual directory.

- **Domain**: Enter the domain name.

  Enter the domain name in DNS format, for example, **nutanix.com**.

- **Directory URL**: Enter the LDAP address of the directory, including the port number.

- **Service Account Username**: Enter the service account user name in the user_name@domain.com format that you want Prism Central to use to detect logons and query user and group information from Active Directory.

  A service account is a special user account that an application or service uses to interact with the Active Directory. Enter your Active Directory service account credentials in this (username) and the following (password) field.

.. note::

  Ensure that you update the service account credentials here whenever the service account password changes or when a different service account is used.

- **Service Account Password**: Enter the service account password.

- When all the fields are correct, click the **Save** button (lower right).

  - ID Firewall uses the service account for ID based security with additional requirements, see Configure Service Account for ID Firewall on page 6.

Once saved, the **Referenced AD Groups** section is displayed. You can add a new user group by clicking **+ Add User Group** and edit the auto-generated **Category Value**. After the active directory configuration is complete, you can create the VDI Policy, see Creating a VDI Policy.

6. Optionally, click Add Inclusion Criteria under Manage the VM Inclusion Criteria to specify which VMs are assigned to AD Group categories upon user logon based on VM name.

.. note::

  It is recommend that users add inclusion criteria if at all possible to prevent any unintended categorizations.

  The VMs with AppType category assigned cannot be categorized by ID Based Security.


Configure Service Account for ID Firewall
..........................................

Active Directory service account in Prism Central is used for connectivity with the Active Directory domain services. ID Firewall also uses the same service account for ID based security.
  To configure a service account for ID firewall, do the following.

#. Create a new user in the Active Directory.

#. Add the user to the Distributed COM Users group and the Event Log Readers domain groups.

#. Start the dcomcnfg.exe utility and go to Component Services > Computers > My Computer > DCOM Config.

#. Right-click on Windows Management and Instrumentation and select Properties from the menu.

#. Switch to Security tab, select Customize option in the Access Permissions section and then click Edit.

#. Add the user and grant Local Access and Remote Access permissions to the user. Click OK to confirm changes.

#. Run the WMIMGMT.msc command to start Windows Management Instrumentation snap-in.

#. Right-click on WMI control (local) and select Properties from the menu.

#. Switch to Security tab and expand Root tree.

#. Select CIMV2 in the expanded tree and click Security.

#. Go to Advanced > Add > Principal and enter the user name.

#. Change scope by selecting This namespace and subnamespaces in the Applies to drop-down menu.

#. Click the check-box to grant the Enable Account and Remote Enable permissions. Click OK to confirm changes.

#. Restart the winmgmt service.

C:\> net stop winmgmt

C:\> net start winmgmt

Alternatively, reboot the domain controller.


#. Repeat step 3 to step 14 on every domain controller.



Takeaways
+++++++++

What are the key things you should know about **Nutanix Flow**?

- In this exercise you utilized Flow to quarantine a VM using the two modalities of the quarantine policy, which are strict and forensic.
- Quarantine policies are evaluated at a higher priority than application policies. A quarantine traffic can block traffic that would otherwise be allowed by an application policy.
- The forensic modality is key to allow limited access a quarantined VM while the VM is quarantined.

.. |blueprints| image:: images/blueprints.png

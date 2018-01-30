## What is this?
Authentication Policies and Silos allow us to restrict user accounts from accessing remote servers as long as the connection is initiated from specified hosts.  In the following example configuration, we will create a silo containing all Domain controllers and other Tier 0 servers, Tier 0 user accounts, and Tier 0 PAWs.  This will only allow Tier 0 user accounts to login to Tier 0 servers (and DCs) from their Tier 0 PAW.  All other connections will be denied.

## Enable Dynamic Access Control (DAC) on Domain Controllers
Create a GPO and apply it to the Domain Controller's OU called **Security - Allow Dynamic Access Control - DCs** with the following settings:

*Computer Configuration > Policies > Admin Templates > System > KDC*
* KDC support for claims, compound authentication and Kerberos armoring: **Enabled, Always provide claims**

On the scope tab:
* Ensure the Link is Enabled.  
* Ensure **Authenticated Users** is selected under **Security Filtering**.
* Ensure there is no WMI filter applied

On the Details tab:
* Set GPO status to: **User configuration settings disabled**

## Enable DAC on PAWs
Create a GPO and apply it to the DOMAIN.COM\Company\Computers OU called **Security - Allow Dynamic Access Control - PAW** with the following settings:

*Computer Configuration > Policies > Admin Templates > System > Kerberos*
* Kerberos client support for claims, compound authentication and Kerberos armoring: **Enabled**

## Configure the Authentication Policy and Silo
1. Create the policy by opening Active Directory Administrative Center and navigate to **Authentication > Policies > right click and create new policy**.
2. Name it: Allow Tier0 users access to Tier0 servers from Tier0 PAW
3. Description: This Policy allows Tier 0 users to access Tier 0 servers from Tier 0 PAWs only.
4. Assigned Silos: (Will come back to this after you create the silo)
5. User sign on > Specify a TGT lifetime for user accounts: 240
6. Computer > Edit > Add a condition
      1- User > AuthenticationSilo > Equals > Value > Tier0
7. Click OK

## Configure the Silo
1. In Active Directory Administrative Center, navigate to **Authentication > Policy Silos > right click and create a new silo**
2. Name: Tier 0
3. Put a bullet in Enforce silo policies
4. Description: All members of Tier 0 users and computers.
5. permitted Accounts > Add the following Computer and user accounts (cannot add groups)
    1. All Domain Controllers
    2. All PAW Tier 0 Computer accounts
    3. All Tier 0 User accounts
    4. Any other Tier 0 server accounts
6. Authentication Policy > Use a single policy... Select the policy you created above

Update the Policy to include the new Silo
1. Edit the policy
2. Assign the silo you created above under Assigned Silos

## Test your settings

Make sure to gpupdate on your servers and your PAWs.

Authentication | Result
---------------|--------
RDP into a DC with your Tier 0 account from your Tier 0 PAW | PASS
RDP from a DC to your Tier 0 PAW | PASS
RDP into a member Tier 1 server from your Tier 0 PAW with your Tier 0 account | FAIL
RDP into your Tier 0 PAW from a HelpDesk workstation | FAIL
Log into your Tier 0 PAW with a non-Tier 0 user account | FAIL
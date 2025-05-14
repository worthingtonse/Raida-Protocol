# Active Mode

RAIDA Tech’s Token has two modes: 
Private mode that is the most private token in the world with no user accounts, logins or tracked transactions.  
Active mode is the most controllable token in the world which requires application for accounts and everything a user does can be controlled, tracked  and surveilled.
Active Mode is an optional feature that allows Wyoming the option of controlling every aspect of the token and to exercise complete surveillance so that every transaction is tracked and every person is accountable. People can be instantly locked out of the system and their funds confiscated. 

Note: According to the Theory of Perfect Money, Accountability mode will make the token less valuable because people don’t like to be controlled and survailed. The only reason people would use this is if the are forced to. 

Another downside is that the State of Wyoming will become a central authority. It is better if there is a lot of independent authorities such as banks that do this work. So Private mode through banks is advantageous. This is exactly how it is done with real cash. 

Active Mode requires much more administrative effort and costs. 

NOTE: The name "West Domain" is analogous to "Windows Domain" or "Linux Realm", tools used to administer enterprise networks. 

# Components of Government Mode include:
West Domain

User Accounts

User Groups

Domain Resources

User Permissions (Access Control Lists)

User Rights

## Domain

A West Domain is a computer network that uses redundant databases to manage user accounts, user groups and access to 
resources such as Authentication Services and Token Lockers.

The West Domain needs "Domain Administrators" who are able to perform various tasks such as creating user accounts, setting password policies, and setting how many tokens each user has permission to authenticate each day. The control can even go down to what parts of the world users are allowed to access the system from and what time of day they may access. 

## Accounts
Everytime a user wants to join the West Domain, the Domain Administrator will need to create an account for them. 
Accounts have a DNS name such as "David.Falk@WST.WY.gov" and are associated with a dotted decimal number such as 154.56.214.899. Accounts can be recorded in a DNS server to allow for global lookup. 

When the account is created, the user will receive an access token that allows them to use the services of the West Domain. Users will need to import this access token into their software to use the West Token. 

## Groups
Groups are logical groupings of users so that they can be given rights and permissions to access resources on the Domain. 
There are Built-in groups that are permanent in the Domain and Custom Groups that can be created as needed by administrators.  
The Domain Administrators can assign people to be members of custom groups and some built-in groups. 

### Built-in Groups
These groups cannot be deleted. Everyone is automatically a member of the Authenticated Users group and the RAIDA Servers Group cannot be modified.
In Phase I, the built-in groups include:
* Authenticated Users (All accounts are members of this group by default)
* Account Administrators (People who are allowed to create, disable and delete accounts). 
* RAIDA Servers (The actual servers used as nodes)
* Treasurers ( People who are allowed to create, disable and delete tokens) 

NOTE: In Phase I, the Treasures Group is able to create and delete but not disable. Disable is not needed right now because we can disable accounts instead and that 
will do nearly the same thing. We will add this in disable tokens Phase II. 

### Custom Groups
Custom Groups reduce the administrative effort needed to administer the domain. For example, we can have a custom group called "Suspect" that is not allowed to use the "Locker" facilities. If we want to stop Bob from using lockers we just 
add his account to the Suspect group. 

In Phase I we will not have custom groups. These will be created in Phase II. To start with we can enable and disable accounts. This is not very granular but it's good for a start. 

## Resources
Resources are things on the ranch that users can use. The West Ranch has these resources: 

* Token Authentication
* Token Change Service
* System Status Reports
* Token Healing
* Mint
* Lockers
* Quantum Safe Key Exchange
* Users & Groups

# Permissions / ACL (Access Control Lists)
Each resource tracks user permissions in its ACL. Each list has a "allow" part on top and a "deny" part on the bottom. To have permission to use a resource, the user must pass two tests: They must be on the allow part of the list, and they must not be on the deny part of the list. 

These Resources have the Authenticated Users built-in group in the allow part. This allows everyone to use these services unless specifically denied. 
* Token Authentication
* Token Change Service
* Token Healing
* Lockers
* Quantum Safe Key Exchange

These Resources have denied everyone automatically except members added to specific groups. 
* Mint: Treasurers built-in group
* System Status Reports: Account Administrators and Treasurers built in groups
* Membership Services:  Account Administrators built in group

# Rights
The ranch has a list of Rights. 
* Create user accounts
* Disable user accounts
* Delete user accounts
* Create tokens
* Disable / freeze tokens 
* Delete / Burn tokens
* Grant and deny permission to use services. 

For example, the Realm Administrator can say that user Bill@West.Wy.gov is able to trade up to $75K per day except on Sundays while Jill@West.Wy.gov may only trade $1K per day from 9am to 5pm MT. 



## Modifications needed for Phase I
* New Denomination Code x78 Client User Keys
* New Denomination Code x77 Account Admin User Keys
* Users & Groups: Create Account (just like create money service)
* Users & Groups: Delete Account And User’s Tokens ( Just like delete all the money service)
* DNS Server to store user names.
* A Users & Groups Client. This must be able to call “create new users” and register them with a DNS server (optional). Must also be able to delete users. 
* Audit services would return user ID too. 
* Need an encryption type that uses the user keys and changes the AN of the encryption key to the challenge. Challenge response becomes a MD5 hash of the challenge sent. 
* Client program would need a place to keep the user keys.
* Client program would need to pown the key every time it uses an user key. encrypted. 
* RAIDA needs new encryption errors such as “Access Denied”

## Tasks

Choices the commission needs to make: 

BlockChain |  Active Mode | Private Mode 
---|---|---
hmnm | mmd| dfdf

Only allow approved people to use the token | Allow anyone to use the token
* Creat, list, update and delete users | No users
* Know how many tokens each person has | Do not know how many tokens a person has
* Know the giver and receiver of tokens when they transact | Do not know the giver and receiver
* Know the physical location of users when they transact | Do not know the physical location of the user
* Know the time that users transact | 
* Complete advanced analsyis of the economy by anyalizing all the transactions. 
* Be able to cancel a user's accounts and seize all tokens
* Be able to stop certains coins that a user has from being spent.
* Limit the number of tokens a person can trade in a single time period.
* Limit the ability of users to send tokens to specific users
* Create accountability with spending
* Allow more than one user to have accountabiliy with a resource (Stop theft on exchanges)
* Allow uses to change their passwords.
* Create a password policy including password complexity, length, history and expiration date.
* Create a security policy that includes the amount of tokens an individual can trade each day, the times and physical locations.
*  




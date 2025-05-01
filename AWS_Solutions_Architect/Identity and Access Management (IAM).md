# IAM: User and Groups -> 

- IAM = Identity and Access Management and is a GLOBAL SERVICE
- On AWS Account creation, AWS by default creates the root account that has all the permission related to that AWS Account.
- Root Account should not be used or shared as best practices.
- Users are people within your organisation and can be grouped in one or multiple groups.
- Groups can only contain other users and not GROUPS!

![[Pasted image 20250430151942.png|1000]]

---
# IAM: Permissions ->

- Users or Groups can be assigned JSON Documents called policies.
- These policies define the `permissions` of the users.
- In AWS, we apply the `the least privilege principle` which infers, do not give the user, more permissions that he/she needs.

![[Pasted image 20250430170554.png|600]]

---

# IAM Policies Structure :

![[Pasted image 20250501174257.png | 1400]]

---

# IAM - Password Policy: 

- ﻿﻿Strong passwords = higher security for your account
- ﻿﻿In AWS, you can setup a password policy:
	- ﻿﻿Set a minimum password length
	- ﻿﻿Require specific character types:
		- ﻿﻿including uppercase letters
		- ﻿﻿lowercase letters
		- ﻿﻿numbers
		- ﻿﻿non-alphanumeric characters
	- ﻿﻿Allow all IAM users to change their own passwords
	- ﻿﻿Require users to change their password after some time (password expiration)
	- ﻿﻿Prevent password re-use

---
# Multi Factor Authentication - MFA

- ﻿﻿Users have access to your account and can possibly change configurations or delete resources in your AWS account
- ﻿﻿You want to protect your Root Accounts and IAM users
- ﻿﻿MFA = password you know + security device you own

## Main benefit of MFA:

- If a password is stolen or hacked, the account is not compromised
## Types of MFA Device : 

- Virtual MFA Device - Like Google Authenticator or Authy on Phone (Supports Multiple tokens on a single device)
- Universal 2nd Factor (U2F) Security Key - Support for multiple root and IAM users using a single security key
  
  ![[Pasted image 20250501181833.png | 500]]
  
- Hardware Key Fob MFA Device
- Hardware Key Fob MFA Device for AWS GovCloud (US)
  
  ![[Pasted image 20250501182017.png | 500]]

---


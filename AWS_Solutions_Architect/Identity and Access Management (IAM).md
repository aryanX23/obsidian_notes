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


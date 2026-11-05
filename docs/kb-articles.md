# Knowledge base articles

These are the four internal how-to articles I wrote in osTicket's knowledge base (category: Account Management, all marked Internal). They live inside osTicket, so I'm keeping a copy here that anyone can read.

## How to unlock a locked-out account

**Symptom:** The user's sign-in screen says "The referenced account is currently locked out and may not be logged on to."

**Cause:** Too many wrong passwords. The policy locks an account after 5 failed attempts, and it unlocks by itself after 15 minutes.

**Before you start:** Verify who is asking, by calling back the number on file or confirming with their manager. Don't act on the ticket text alone.

**Steps:**
1. On the domain controller, open Active Directory Users and Computers.
2. Find the user, open their Properties, and go to the Account tab.
3. Tick "Unlock account" and click OK.
4. Ask the user to try again.

**Verify:** The user can sign in. If they have forgotten the password, use "How to reset a user's password."

**Escalate:** If one user keeps getting locked out, or many users are locked at once, someone may be guessing passwords. Tell Tier 2 or security.

## How to reset a user's password

**Before you start:** Verify identity by callback (and a security question). A ticket alone is not proof.

**Steps:**
1. In Active Directory Users and Computers, right-click the user and choose Reset Password.
2. Enter a temporary password that is unique to this reset.
3. Keep "User must change password at next logon" ticked.
4. Tick "Unlock the user's account" only if the account is also locked.
5. Click OK.

**Hand-off:** Give the temporary password by phone. Never put it in the ticket, an email, or chat.

**Verify:** The user signs in and is asked to choose a new password.

**Never:** ask a user for their current password.

## How to set up a new hire account

**Before you start:** Confirm the request comes from the hiring manager.

**Steps:**
1. In Active Directory Users and Computers, open Company, right-click the Users OU, and choose New, then User.
2. Use the standard username format: first initial plus last name (for example, cnguyen).
3. Enter a temporary password and keep "User must change password at next logon" ticked.
4. Add the user to their team's group (for example, Sales-Team). Give access through groups, never to individual people.
5. Give the temporary password to the manager by phone.

**Verify:** On a test sign-in, the password change is forced and the team's shared folder opens.

## How to offboard an employee (DISABLE, don't delete)

**Before you start:** Confirm the request comes from the employee's manager and that HR has confirmed the departure date. Do the work on or before the last day.

**Steps:**
1. In Active Directory Users and Computers, open the user's Properties and check the Member Of tab to see their groups.
2. Open each group (for example, Sales-Team), go to the Members tab, select the user, and click Remove.
3. Right-click the user and choose Disable Account. A small down-arrow icon appears on the account.
4. Optional: in the user's General tab, write the ticket number and date in the Description field, so anyone later can see why the account is disabled.

**Do not delete the account.** Disabling blocks sign-in while keeping the user's files and history for the retention period set by company policy. Deleting is permanent.

**Verify:** A sign-in attempt as the user is refused with a message that the account has been disabled.

**Notes:** Equipment return and mailbox handling are separate tasks. If the person is leaving on bad terms, disable the account immediately and tell security.

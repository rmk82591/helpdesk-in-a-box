# Help Desk in a Box

A self-hosted help desk (osTicket) on Ubuntu Server in VirtualBox, built to practice what a Tier 1 support desk does all day: take a request, set a priority and a deadline, fix the problem, document the fix, and close the ticket. I worked five tickets from start to finish against my [Active Directory home lab](https://github.com/rmk82591/AD-home-lab).

## Architecture

```mermaid
flowchart LR
  U["Employee<br/>web form"] -->|opens ticket| T["Ticket1 - Ubuntu 24.04<br/>osTicket, Apache, PHP, MariaDB"]
  A["Tier 1 agent<br/>staff panel"] -->|works ticket| T
  A -.->|fixes by hand| DC["AD-DC01<br/>Active Directory"]
  A -.->|tests| C["Client01<br/>Windows 11"]
  DC --- C
```

## The setup

| Machine | OS | Role | Specs |
|---|---|---|---|
| Ticket1 | Ubuntu Server 24.04.5 LTS | Help desk server | 2 GB RAM, 2 vCPU, 25 GB disk |
| AD-DC01, Client01 | Windows Server 2025, Windows 11 | The company network the tickets are about (from the [AD home lab](https://github.com/rmk82591/AD-home-lab)) | |

- **Stack:** Apache 2.4.58, PHP 8.3.6, MariaDB 10.11.14, osTicket 1.18.4
- **Access:** the web page and SSH are reachable only from my own PC, through VirtualBox port forwarding bound to localhost
- **Hypervisor:** VirtualBox 7.2.8

![Stack versions](docs/images/02-stack-versions.png)
*Apache 2.4.58, PHP 8.3.6, and MariaDB 10.11 installed and running.*

![osTicket requirements check](docs/images/04-osticket-requirements-check.png)
*osTicket's requirements check: required items pass. IMAP is skipped on purpose, since email was out of scope.*

## How it was built

1. Installed Ubuntu Server 24.04 LTS with OpenSSH, updated it, and took a snapshot
2. Installed the web stack: `sudo apt install -y apache2 mariadb-server php libapache2-mod-php php-mysql php-gd php-intl php-apcu php-mbstring php-xml php-curl php-zip unzip`
3. Hardened MariaDB and created an `osticket` database with a user limited to it
4. Downloaded osTicket 1.18.4, placed it in Apache's web folder, and ran the web installer
5. Locked it down afterward: config file read-only, installer folder removed
6. Configured five help topics, two SLA plans, and a Tier 1 Agent role with an agent account
7. Worked five tickets and wrote four knowledge base articles

## Security choices

- **Least-privilege database user:** the `osticket` user can only touch the `osticket` database, and MariaDB listens on localhost only.
- **Hardened after install:** the config file is read-only and the setup folder is removed, so the installer can't be re-run.
- **Roles:** the Tier 1 Agent role can work tickets but can't delete, merge, or transfer them. Day-to-day work happened as that agent, not as the admin.
- **Knowledge base is internal-only,** because reset procedures shouldn't be readable by outsiders.
- **Temporary passwords never go in tickets.** The plan is always "hand it over by phone."
- **Supported versions on purpose:** I stayed on Ubuntu 24.04 LTS, because the newer Ubuntu's PHP 8.5 is outside osTicket 1.18.4's supported range (PHP 8.2 to 8.4).
- **Snapshots at every milestone,** with a short description on each.

![Database user limited to one database](docs/images/03-db-least-privilege.png)
*The osticket user can only touch the osticket database, and MariaDB listens on localhost only. A password hash on this screen is covered.*

## How tickets flow

Help topics set the default priority and deadline:

| Help topic | Priority | SLA plan |
|---|---|---|
| Account Lockout | High | Urgent-4h |
| Password Reset | Normal | Urgent-4h |
| File Share Access | Normal | Standard-24h |
| New Hire Setup | Normal | Standard-24h |
| Offboarding | High | Urgent-4h |

SLA timers run on business hours, 8 to 5 on weekdays, so a ticket opened Wednesday afternoon with a 24-hour SLA is due the following Monday.

![Help topics](docs/images/06-help-topics.png)
*Five custom help topics with priorities.*

![SLA plans](docs/images/07-sla-plans.png)
*Two custom SLA plans: Urgent-4h and Standard-24h.*

![Tier 1 Agent role](docs/images/08-tier1-role.png)
*Tier 1 Agent role: can work tickets, but can't delete, merge, or transfer them.*

![Agent view](docs/images/09-tier1-agent-view.png)
*The same queue as a Tier 1 agent: no Admin Panel link, and no delete button.*

## The five tickets

| # | Ticket | Requester | Topic | What I did | Outcome |
|---|---|---|---|---|---|
| 1 | #176914 | Jordan (employee) | Account Lockout | Noted identity verification (simulated), unlocked the account in AD | Closed in 13 minutes |
| 2 | #664429 | Sam (employee) | File Share Access | Added Sam to the Sales-Team group; tested access from Client01 | Closed |
| 3 | #744674 | Jordan (hiring manager) | New Hire Setup | Created `cnguyen`, added to Sales-Team, forced a password change, tested first sign-in | Closed in 30 minutes |
| 4 | #628633 | Jordan (manager) | Offboarding | Removed Sam from Sales-Team and disabled the account (not deleted) | Closed in 30 minutes |
| 5 | #791969 | Jordan (employee) | Password Reset | Reset the password in AD with a forced change at next sign-in | Closed in 18 minutes |

Ticket 1 reuses the lockout procedure from my Active Directory home lab. Tickets 2 to 5 involved new work in AD, performed for this project.

### Ticket 1: lockout

![The web form](docs/images/10-user-ticket-form.png)
*Ticket 1 as the employee sees it: the web form with the Account Lockout topic.*

![Ticket claimed](docs/images/11-ticket1-assigned.png)
*Ticket #176914 claimed, with High priority, the Urgent-4h SLA, and an internal note about verifying identity.*

![Ticket closed](docs/images/12-ticket1-closed.png)
*Ticket #176914 closed in 13 minutes: claimed, identity check noted, unlocked in AD, user notified.*

### Ticket 2: share access

![Sam has access](docs/images/14-sam-share-allowed.png)
*Sam, now in Sales-Team, opens the Sales share and creates a file. `whoami` confirms the account.*

### Ticket 3: new hire

![New account](docs/images/16-new-hire-cnguyen.png)
*Casey Nguyen's account in Company\Users, a member of Sales-Team and Domain Users.*

![Casey has access](docs/images/17-casey-share-allowed.png)
*Casey, a new member of Sales-Team, opens the Sales share and creates a file. `whoami` confirms the account.*

![Ticket 3 closed](docs/images/18-ticket3-closed.png)
*Ticket #744674 closed in 30 minutes: account created in Company\Users, added to Sales-Team, password change enforced at first sign-in.*

### Ticket 4: offboarding

![Sam disabled](docs/images/19-sam-disabled.png)
*Sam Lee's account disabled in Active Directory (down-arrow icon) and removed from Sales-Team, not deleted.*

![Ticket 4 closed](docs/images/20-ticket4-closed.png)
*Ticket #628633 closed in 30 minutes: Sales access removed and account disabled, not deleted, to preserve data.*

### Ticket 5: password reset

![Ticket 5 closed](docs/images/22-ticket5-closed.png)
*Ticket #791969 closed in 18 minutes: identity verified by callback (simulated), password reset in AD, change forced at next sign-in.*

<details>
<summary>More screenshots</summary>

![Snapshots](docs/images/00-snapshot-list.png)
*VirtualBox snapshots as rebuild checkpoints.*

![SSH session](docs/images/01-ssh-connected.png)
*SSH from my PC into the server, confirming Ubuntu 24.04.5 LTS.*

![Staff login](docs/images/05-staff-dashboard.png)
*First staff login after the install.*

![Unlock in AD](docs/images/12a-unlock-account.png)
*Captured in the AD home lab (Project 2): unlocking Jordan's account in Active Directory Users and Computers.*

![Sam denied](docs/images/13a-sam-share-denied.png)
*Captured in the AD home lab (Project 2): Sam denied on the Sales share before he joined the group.*

![Ticket 2 claimed](docs/images/13-ticket2-assigned.png)
*Ticket #664429 (Sam's share request) claimed, with an internal note and the Standard-24h SLA.*

![Ticket 2 closed](docs/images/15-ticket2-closed.png)
*Ticket #664429 closed: Sam added to the Sales-Team group, access confirmed from Client01, resolution documented.*

![Forced password change](docs/images/16a-forced-password-change.png)
*First sign-in as cnguyen: Windows asks for the temporary password plus a new one before Casey is let in.*

![Sam blocked](docs/images/19a-sam-signin-blocked.png)
*Sam's sign-in attempt on Client01 is refused after the account is disabled.*

![Password reset dialog](docs/images/21-password-reset-dialog.png)
*Resetting Jordan's password in Active Directory Users and Computers, with the forced password change at next sign-in checked.*

</details>

## Knowledge base

I wrote four internal how-to articles from the tickets: unlock a locked-out account, reset a user's password, set up a new hire account, and offboard an employee (disable, don't delete). They live inside osTicket, so the text is also in [`docs/kb-articles.md`](docs/kb-articles.md).

![Knowledge base list](docs/images/23-knowledgebase-article-list.png)
*Knowledge base with four internal how-to articles written from the tickets.*

## Lessons learned

- **Check version support before building.** The first server I built ran a newer Ubuntu than I meant to, with PHP 8.5, which osTicket doesn't support. I caught it from a package error, confirmed it with `lsb_release`, and rebuilt on 24.04 LTS.
- **osTicket rejected my lab email domain.** The agent form said "Valid email is required," because osTicket checks that an address's domain has a mail server in public DNS, and my lab domain only exists in private Active Directory DNS. I turned off Verify Email Addresses (Admin Panel, Emails, Settings), since no mail leaves this lab. A production help desk would keep it on.
- **Command output can contain credential material.** `SHOW GRANTS` printed a password hash, so I covered it before publishing the screenshot.
- **Access goes through groups, not people.** Sam, Casey, and Jordan all get Sales access the same way, which makes onboarding and offboarding two group changes.
- **Disable, don't delete.** Offboarding disables the account so files and history are kept for the retention period.
- **SLA clocks run on business hours,** so due dates don't always land when you'd expect.

## Limitations

- osTicket isn't integrated with Active Directory. I did the AD work by hand in a separate window.
- No email: confirmations aren't delivered, and email-to-ticket isn't configured (IMAP not installed on purpose).
- Identity checks are simulated, and the tickets are scenarios I wrote, not real users.
- The help desk page runs over plain HTTP on localhost only. A real one would use HTTPS.
- The Windows evaluation licenses expire (about 90 days for Windows 11, 180 for Server).

## Next ideas

- Sign in to osTicket with Active Directory accounts (LDAP)
- Email-to-ticket with a lab mail server
- HTTPS in front of the help desk

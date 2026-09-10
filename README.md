
 Hello, I'm AYOOLA
# Active Directory & GPO Deployment Project

**IAM / Systems Administration Portfolio Documentation**

Prepared by: **Moses Shadrach Ayoola**
Date: *22 August 2026*
Version: *v1.0*

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Project Overview & Objectives](#2-project-overview--objectives)
3. [Environment & Infrastructure](#3-environment--infrastructure)
4. [Active Directory Installation & Domain Controller Promotion](#4-active-directory-installation--domain-controller-promotion)
5. [Organisational Unit (OU) Design](#5-organisational-unit-ou-design)
6. [Identity Lifecycle — User & Group Provisioning](#6-identity-lifecycle--user--group-provisioning)
7. [Group Policy Object (GPO) Design & Implementation](#7-group-policy-object-gpo-design--implementation)
8. [GPO Security Filtering & Scope Management](#8-gpo-security-filtering--scope-management)
9. [Identity Lifecycle Management (Joiner / Mover / Leaver)](#9-identity-lifecycle-management-joiner--mover--leaver)
10. [AD Certificate Services — Overview & Relevance](#10-ad-certificate-services--overview--relevance)
11. [IAM Incident & Service Request Management](#11-iam-incident--service-request-management)
12. [Key Competencies Demonstrated](#12-key-competencies-demonstrated)
13. [References & Tools](#13-references--tools)

---

## 1. Executive Summary

This project presents the design and implementation of a simulated enterprise Identity and Access Management (IAM) environment using Microsoft Active Directory Domain Services (AD DS) and Group Policy. The environment was developed for **ApexSecure Technologies**, a fictional organisation, with the objective of demonstrating how a cybersecurity administrator can centrally manage identities, authentication, access control, and security policies.

The implementation includes deployment of a Windows Server domain controller, creation of the domain, structured Organisational Units (OUs), security groups, user accounts, and security-focused Group Policy Objects (GPOs). The project also demonstrates Identity Lifecycle Management through Joiner, Mover, and Leaver processes, together with security filtering, password controls, account lockout policies, and simulated IAM service requests.

The final environment provides a practical demonstration of centralised identity administration, least-privilege access, policy enforcement, and repeatable account lifecycle procedures. All identities and infrastructure used in this project are fictional and are intended solely for an isolated cybersecurity laboratory.

---

## 2. Project Overview & Objectives

The project simulates the IAM environment of ApexSecure Technologies, a fictional technology organisation. The goal is to build a manageable Active Directory structure that supports secure identity administration while demonstrating practical cybersecurity controls. The implementation is performed in an isolated VirtualBox lab.

### 2.1 Scenario

**Scenario:** Enterprise

ApexSecure Technologies requires centralised identity management for staff in its Abuja office. The organisation needs controlled user provisioning, department-based access, strong authentication policies, account lockout controls, and repeatable onboarding and offboarding procedures.

### 2.2 Objectives

- **Deploy a functional Active Directory environment** — install AD DS and promote a server to Domain Controller to support centralised identity management for a simulated Finance department
- **Enforce security baselines through Group Policy** — implement password, account lockout, and access-restriction policies scoped to Finance staff to reduce the risk of unauthorised access
- **Demonstrate identity lifecycle management** — provision users and groups following a consistent naming convention, and verify policy enforcement by testing as an end user on a domain-joined client

### 2.3 Scope

**In scope:** Windows Server AD DS deployment, domain controller promotion, DNS integration, OU design, security groups, user provisioning, Group Policy, security filtering, JML lifecycle processes, account lockout and password controls, and simulated IAM service requests.

**Out of scope:** production deployment, real employee data, cloud identity integration, internet-facing services, and connection to a real corporate network.

---

## 3. Environment & Infrastructure

The lab is hosted in Oracle VirtualBox using an isolated virtual network. The environment contains a Windows Server domain controller and a Windows client used to test domain authentication and Group Policy application.

### 3.1 Lab Topology Diagram

![Oracle VirtualBox Manager](screenshots/03-1-lab-topology-virtualbox-manager.png)

### 3.2 Virtual Machines & Roles

- **Domain Controller:** APEX-DC01 — Windows Server 2022 — `192.168.10.10`
- **Client:** APEX-CLIENT01 — Windows 11 Pro — `192.168.10.20`
- **DNS:** Hosted on APEX-DC01
- **Domain:** apexsecure.tech
- **NetBIOS name:** APEXSECURE
- **Virtualisation platform:** Oracle VirtualBox

| VM Name | Role | OS | IP Address |
|---|---|---|---|
| APEX-DC01 | Domain Controller / DNS | Windows Server 2022 | 192.168.10.10 |
| APEX-CLIENT01 | Domain-joined Client | Windows 8 | 192.168.10.20 |

### 3.3 Software & Tool Versions

Windows Server 2022; Windows 8; Oracle VirtualBox; Active Directory Users and Computers (ADUC); Group Policy Management Console (GPMC); PowerShell; Windows Event Viewer; `dcdiag`; `gpresult`.

- Windows Server version: 10.0.20348 (Build 20348)
- Windows client version: 6.2.9200 (Build 9200)
- Hypervisor: VirtualBox

![Windows Server 2022 domain controller running in Oracle VirtualBox](screenshots/03-3-windows-server-version.png)

![Windows 8 client running in Oracle VirtualBox](screenshots/03-3-windows-client-version.png)

---

## 4. Active Directory Installation & Domain Controller Promotion

AD DS is installed on APEX-DC01 and the server is promoted to the first domain controller for the new forest. DNS is installed as part of the domain controller deployment. After promotion, ADUC, DNS Manager, GPMC, and command-line verification tools are used to confirm that the domain is functioning correctly.

### 4.1 Installing the AD DS Role

*Server Manager — Add Roles and Features*

### 4.2 Promoting to Domain Controller

![Domain controller promotion in progress](screenshots/04-2-promoting-to-dc-1.png)

![Domain controller promotion complete](screenshots/04-2-promoting-to-dc-2.png)

### 4.3 Post-Promotion Verification

Verification includes checking domain controller health with `dcdiag`, confirming the domain in ADUC, verifying DNS records, and confirming that the client can resolve and communicate with the domain controller.

![Post-promotion domain controller verification](screenshots/04-3-post-promotion-verification.png)

---

## 5. Organisational Unit (OU) Design

The OU design uses three primary organisational containers for the Abuja office: **Users**, **Groups**, and **Computers**. This separation supports clear administrative boundaries, targeted Group Policy application, and easier identity and access management. Departmental user accounts are organised under the Users OU, security groups are maintained under the Groups OU, and domain-joined workstation accounts are maintained under Computers.

### 5.1 OU Structure Diagram / Rationale


The three primary OUs under the Abuja organisational structure are:

1. **Users OU** — contains standard employee accounts organised by department such as IT, Finance, HR, and Operations.
2. **Groups OU** — contains security groups used to implement role-based access and simplify permission management.
3. **Computers OU** — contains domain-joined workstation and computer accounts so workstation-focused GPOs can be applied separately from user policies.

This three-OU design keeps identity objects logically separated and makes GPO targeting, administration, and future delegation easier.

### 5.2 OU Layout in ADUC

![OU layout in Active Directory Users and Computers](screenshots/05-2-ou-layout-in-aduc.png)

---

## 6. Identity Lifecycle — User & Group Provisioning

Users are provisioned using a consistent naming standard and placed in the OU that matches their business function. Security groups are used to represent access roles, rather than assigning permissions individually wherever possible. This supports role-based access control and simplifies future access reviews.

### 6.1 Naming Convention & UPN Standard

- **Naming convention:** first name.last name
- **UPN suffix:** apexsecure.tech
- **Computer naming convention:** `APEX-<ROLE>-<NUMBER>`
- **Example user:** `john.doe@apexsecure.local`

### 6.2 Group Structure per OU

Recommended security groups include: `GG-Abuja-IT`, `GG-Abuja-Finance`, `GG-Abuja-HR`, `GG-Abuja-Operations`, `GG-IT-Admins`, and `GG-Helpdesk`. Global security groups represent business roles and are used to control access to resources.

| Group Name | Type/Scope | OU | Purpose |
|---|---|---|---|
| GG-Abuja-IT | Security / Global | Abuja/Users/IT | IT role access |
| GG-Abuja-Finance | Security / Global | Abuja/Users/Finance | Finance role access |
| GG-Abuja-HR | Security / Global | Abuja/Users/HR | HR role access |

![Security group structure per OU](screenshots/06-2-group-structure-per-ou.png)

### 6.3 Full User Provisioning Register

A total of 12 fictional lab users is provisioned, with three users assigned to each department. All users follow the firstname.lastname naming convention and use the `@apexsecure.tech` UPN suffix.

- **IT:** john.doe, michael.obi, grace.ade
- **Finance:** david.okafor, sarah.ali, emeka.nwosu
- **HR:** mary.adams, peter.bello, ruth.ayo
- **Operations:** daniel.eke, james.obi, esther.ugo

| Username | Full Name | Department | OU |
|---|---|---|---|
| john.doe | John Doe | IT | Abuja/Users/IT |
| michael.obi | Michael Obi | IT | Abuja/Users/IT |
| grace.ade | Grace Ade | IT | Abuja/Users/IT |
| david.okafor | David Okafor | Finance | Abuja/Users/Finance |
| sarah.ali | Sarah Ali | Finance | Abuja/Users/Finance |
| emeka.nwosu | Emeka Nwosu | Finance | Abuja/Users/Finance |
| mary.adams | Mary Adams | HR | Abuja/Users/HR |
| peter.bello | Peter Bello | HR | Abuja/Users/HR |
| ruth.ayo | Ruth Ayo | HR | Abuja/Users/HR |
| daniel.eke | Daniel Eke | Operations | Abuja/Users/Operations |
| james.obi | James Obi | Operations | Abuja/Users/Operations |
| esther.ugo | Esther Ugo | Operations | Abuja/Users/Operations |

![Full user provisioning register in ADUC](screenshots/06-3-user-provisioning-register.png)

---

## 7. Group Policy Object (GPO) Design & Implementation

GPOs are used to enforce consistent security controls across domain users and computers. The project separates the default domain password policy from workstation hardening and department-specific restrictions so that each policy has a clear purpose and scope.

### 7.1 Default Domain Password Policy GPO

- Minimum password length: **12 characters**
- Maximum password age: **60 days**
- Minimum password age: **1 day**
- Password history: **10 previous passwords**
- Password complexity: **Enabled**
- Account lockout threshold: **5 invalid attempts**
- Account lockout duration: **15 minutes**
- Reset lockout counter after: **15 minutes**

These settings strengthen authentication and reduce the likelihood of successful password guessing while keeping the policy practical for a simulated enterprise environment.

![Default domain password policy GPO settings](screenshots/07-1-default-domain-password-policy-1.png)

![Default domain password policy GPO — account lockout settings](screenshots/07-1-default-domain-password-policy-2.png)

### 7.2 Restricted Access GPO — Control Panel & CMD Lockdown

A restricted-user GPO is linked to the appropriate user OU. It prevents standard users from accessing selected administrative interfaces such as Control Panel and Command Prompt. The purpose is to reduce unnecessary local administrative capability and demonstrate policy-based access restriction.

![Restricted access GPO configuration](screenshots/07-2-restricted-access-gpo-1.png)

![Restricted access GPO linked to user OU](screenshots/07-2-restricted-access-gpo-2.png)

![Restricted access GPO verification](screenshots/07-2-restricted-access-gpo-3.png)

### 7.3 Additional GPOs — IAM Security Hardening

Proposed hardening GPOs:

1. **Workstation Security Baseline** — enables screen lock after 10 minutes of inactivity.
2. **Removable Media Control** — restricts unauthorised use of removable storage where appropriate.
3. **Windows Defender Baseline** — enables Microsoft Defender and security notifications.
4. **Audit Policy** — enables auditing for logon events, account management, and policy changes.
5. **User Restrictions** — limits selected system configuration tools for standard users.

| GPO Name | Setting(s) | Linked OU | Purpose |
|---|---|---|---|
| Workstation Security Baseline | 10-minute screen lock; security baseline | Abuja/Computers | Reduce unattended-session risk |
| Windows Defender Baseline | Defender enabled; security notifications | Abuja/Computers | Endpoint hardening |
| Audit Policy Baseline | Logon, account management and policy-change auditing | Domain/Computers | Security monitoring |

![Additional IAM security hardening GPOs](screenshots/07-3-additional-gpos-hardening.png)

---

## 8. GPO Security Filtering & Scope Management

GPO scope is controlled primarily through OU linking and security filtering. For example, department-specific policies can be linked to the relevant user OU, while administrative policies are limited to approved security groups. `gpresult /r` is used on the client to verify which policies were actually applied.

![GPO security filtering configuration](screenshots/08-gpo-security-filtering-1.png)

![gpresult /r output on the domain-joined client](screenshots/08-gpo-security-filtering-2.png)

---

## 9. Identity Lifecycle Management (Joiner / Mover / Leaver)

The JML process provides a repeatable approach to identity lifecycle management. Each identity event is documented, approved, executed, and verified to reduce orphaned accounts and excessive access.

### 9.1 Joiner — New User Onboarding

Create the user account, populate required attributes, place the account in the correct OU, assign the appropriate security groups, apply the standard password policy, and verify successful domain authentication.

![Joiner — new user onboarding](screenshots/09-1-joiner-new-user-onboarding.png)

### 9.2 Mover — Department/OU Transfer

When a user changes department, move the account to the new OU, remove obsolete role groups, add the new approved role groups, and verify the resulting GPO and access scope.

![Mover — department/OU transfer](screenshots/09-2-mover-department-transfer.png)

### 9.3 Leaver — Offboarding & Deprovisioning

Disable the account, remove unnecessary group memberships, document the action, and retain the account in an appropriate disabled-user location according to the organisation's retention process. This prevents continued access after employment ends.

![Leaver — account disabled](screenshots/09-3-leaver-offboarding-1.png)

![Leaver — offboarding verification](screenshots/09-3-leaver-offboarding-2.png)

---

## 10. AD Certificate Services — Overview & Relevance

AD Certificate Services (AD CS) is considered as a future enhancement to the lab. It can support certificate-based authentication, secure services, and device or user certificates. For this version of the project, AD CS is documented as planned rather than required for the core AD DS deployment.

---

## 11. IAM Incident & Service Request Management

The lab simulates common IAM service requests and incidents, including account lockout and password reset. Each event is recorded with the request, action taken, verification result, and closure status.

![IAM service request — account lockout](screenshots/11-iam-incident-service-request-1.png)

![IAM service request — password reset](screenshots/11-iam-incident-service-request-2.png)

| Ticket # | Issue | Resolution | Date |
|---|---|---|---|
| IAM 001 | User account lock | Account unlocked through ADUC and login verified | 21/08/2026 |
| IAM 002 | User request for password change | Password change through ADUC and login verified | 21/08/2026 |

---

## 12. Key Competencies Demonstrated

- Active Directory Domain Services deployment and domain controller administration
- OU architecture and identity organisation
- User and security group provisioning
- Group Policy design, linking, and security filtering
- Password and account lockout policy administration
- Joiner, Mover, Leaver identity lifecycle management
- Least-privilege and role-based access concepts
- PowerShell-based administration
- IAM incident and service-request handling
- Technical documentation and evidence collection

---

## 13. References & Tools

- Microsoft Learn — Active Directory Domain Services documentation
- Microsoft Learn — Group Policy documentation
- Microsoft Learn — Windows Server security and auditing documentation
- Oracle VirtualBox
- PowerShell, ADUC, GPMC, Event Viewer, `dcdiag`, `gpresult`

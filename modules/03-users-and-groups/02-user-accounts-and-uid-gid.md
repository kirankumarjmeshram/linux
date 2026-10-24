# 🆔 Linux User Accounts and UID/GID

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:

- Understand Linux user accounts and their components.
- Differentiate between root, regular, and system accounts.
- Explain UID (User ID) and GID (Group ID).
- Understand how Linux assigns and uses UIDs and GIDs.
- Identify the purpose of `/etc/passwd`, `/etc/shadow`, `/etc/group`, and `/etc/gshadow`.
- Understand how numeric IDs relate to file ownership and permissions.
- Use Linux commands to inspect user accounts and their identifiers.

---

## 📖 Introduction

Linux is a multi-user operating system that supports multiple users, applications, and services running on the same system.

To manage access securely, Linux maintains **user accounts** and assigns each account a numeric identifier called a **UID (User ID)**. Groups have their own numeric identifiers called **GIDs (Group IDs)**.

Usernames and group names make these accounts easier for administrators to identify, while Linux uses numeric IDs internally to manage ownership, permissions, and access to system resources.

Linux stores local account and group information in files such as `/etc/passwd`, `/etc/shadow`, and `/etc/group`.

Understanding user accounts, UIDs, and GIDs is essential for Linux administration and DevOps tasks such as managing server access, configuring application permissions, troubleshooting file ownership issues, and running applications securely.

---

## 👤 Understanding Linux User Accounts

### 1. What Is a User Account?

A **Linux user account** represents a person, application, or system service that interacts with the operating system.

Linux uses user accounts to identify processes, manage file ownership, and control access to system resources.

Each user account has a unique **UID (User ID)** that Linux uses to identify the account internally.

For example:

- `kiran` — a regular user who logs in and works on the system.
- `root` — the superuser with extensive administrative privileges.
- `www-data` — a service account commonly used by web servers on Debian-based systems.

To display the current username:

    whoami

To display the user's UID and group information:

    id

**Key point:** A user account provides an identity under which processes run and permissions are evaluated.

---

### 2. Components of a User Account

A Linux user account contains several pieces of information used to identify the user and configure their environment.

For local accounts, much of this information is represented in `/etc/passwd`.

Example entry:

    kiran❌1000:1000:Kiran Meshram:/home/kiran:/bin/bash

The fields are separated by colons (`:`).

| Field          | Example           | Meaning                                                                                |
| -------------- | ----------------- | -------------------------------------------------------------------------------------- |
| Username       | `kiran`         | Name used to identify the account                                                      |
| Password field | `x`             | Indicates that password information is stored separately, typically in `/etc/shadow` |
| UID            | `1000`          | Numeric user identifier                                                                |
| GID            | `1000`          | Numeric ID of the user's primary group                                                 |
| GECOS          | `Kiran Meshram` | Optional descriptive information                                                       |
| Home directory | `/home/kiran`   | Default home directory                                                                 |
| Login shell    | `/bin/bash`     | Shell launched for a normal interactive login, if configured                           |

**Important:** The `/etc/passwd` file is generally readable by all users, so password hashes are not normally stored there. Protected password information is typically maintained in `/etc/shadow`.

You can inspect your account entry using:

    getent passwd "$USER"

To display the local account file:

    cat /etc/passwd

Use `getent` when you want to query the system's configured account databases, which may include centralized sources as well as local files.

---

### 3. Types of User Accounts

Linux systems commonly use three broad categories of accounts.

#### 1. Root Account

The `root` account is the superuser account with UID `0`.

It has extensive privileges to administer the system, including managing users, changing permissions, installing software, and modifying system configuration.

Check the root account:

    id root

Root access should be used carefully because incorrect commands can affect the entire system.

#### 2. Regular User Accounts

Regular accounts are typically used by people who log in and perform everyday tasks.

They generally have limited privileges and access to their own files. Administrative operations may require `sudo` permission.

Example:

    id kiran

Regular user accounts commonly have home directories under `/home`, although the location can vary.

#### 3. System or Service Accounts

System accounts are used to run services and applications rather than to provide normal interactive access to a person.

Examples may include:

- `www-data` — commonly used by web server processes on Debian-based systems.
- `sshd` — may be used for privilege separation by the OpenSSH service, depending on the configuration.
- Other application-specific accounts created during software installation.

These accounts often have restricted permissions and may use a non-interactive shell.

**Note:** Account names, UID ranges, and account purposes vary across distributions and configurations.

---

### 4. Local and Centralized User Accounts

Linux can obtain user account information from local files or from centralized identity services.

#### Local User Accounts

Local accounts are managed directly on an individual Linux machine.

Their information is commonly stored in files such as:

- `/etc/passwd` — account details.
- `/etc/shadow` — protected password information and related settings.
- `/etc/group` — group information.
- `/etc/gshadow` — protected group administration information.

Common commands for managing local accounts include:

    sudo useradd kiran
    sudo passwd kiran
    sudo userdel kiran

These commands change system configuration, so understand their effects before running them. Do not create another account with a username that already exists.

#### Centralized User Accounts

In organizations, user identities may be managed through a central identity service rather than configured separately on every server.

Common technologies include:

- **LDAP** — a directory protocol often used to access centralized identity information.
- **Active Directory** — Microsoft's directory service, commonly used in enterprise environments.
- **SSSD** — a Linux service that can connect systems to identity providers such as LDAP and Active Directory.
- **NIS** — an older network information service still found in some legacy environments.

Centralized identity management helps administrators manage access across multiple servers and apply consistent account policies.

For example, a company can manage developer identities centrally and configure its Linux servers to recognize those users.

The `getent` command can query configured identity sources:

    getent passwd kiran
    getent group developers

Whether a particular account is found depends on the local configuration and available identity services.

#### Local vs Centralized Accounts

| Feature            | Local accounts                           | Centralized accounts                                       |
| ------------------ | ---------------------------------------- | ---------------------------------------------------------- |
| Account management | On an individual machine                 | Through a central identity service                         |
| Typical use        | Personal systems and standalone servers  | Enterprise and multi-server environments                   |
| Administration     | Often managed separately on each machine | Can be managed centrally                                   |
| Availability       | Depends on the local system              | Depends on identity-service configuration and availability |
| Example            | Account in `/etc/passwd`               | Account provided through LDAP or Active Directory          |

**DevOps relevance:** Understanding both approaches helps when configuring SSH access, managing server permissions, onboarding team members, and troubleshooting identity-related issues across multiple Linux servers.

---

## 🆔 Understanding UID (User ID)

### 1. What Is a UID?

A **UID (User ID)** is a numeric identifier assigned to a Linux user account. Linux uses UIDs to identify users internally and determine ownership of files, directories, and processes.

Usernames are human-readable names, while UIDs provide the numeric identity used by the operating system.

For example, run:

    id kiran

Example output:

    uid=1000(kiran) gid=1000(kiran) groups=1000(kiran),27(sudo)

Here:

- `uid=1000` — the user's UID.
- `kiran` — the username associated with that UID.
- `gid=1000` — the user's primary group ID.

To check the current user's UID:

    id -u

To check another user's UID:

    id -u kiran

To display account information, including the UID:

    getent passwd kiran

**Key point:** Each user account has a UID. Usernames help people identify accounts, while Linux uses numeric IDs to manage identity and permissions.

### 2. UID 0: The Root User

Linux determines privileged identity primarily through UID `0`, not merely through the username `root`.

For example, a process running with effective UID `0` generally has superuser privileges, subject to applicable security mechanisms and restrictions.

This is why administrators must be careful when running commands as root.

You can execute an administrative command using `sudo`:

    sudo apt update

This runs the command with elevated privileges when permitted by the system's `sudo` configuration.

**Best practice:** Use a regular account for everyday tasks and elevate privileges only when necessary. Avoid running applications as root unless required.

### 3. System Users vs Regular Users

Linux uses different accounts for people, applications, and system services.

#### System Users

System users are commonly created to run operating-system components, background processes, and applications.

Examples include:

- `www-data` — commonly used by web servers on Debian-based distributions.
- `sshd` — may be used for privilege separation by OpenSSH.
- Application-specific service accounts created during software installation.

These accounts often have restricted permissions and may use a non-interactive shell.

#### Regular Users

Regular users are typically created for people who log in and perform everyday tasks.

They commonly have:

- A home directory.
- A login shell.
- Permissions to manage their own files.
- Administrative access only when explicitly granted.

#### Comparison

| Feature         | System user                       | Regular user                       |
| --------------- | --------------------------------- | ---------------------------------- |
| Primary purpose | Run services and system processes | Support interactive work by people |
| Login access    | Often restricted                  | Usually enabled                    |
| Home directory  | May be absent or service-specific | Commonly under `/home`           |
| Shell           | May be non-interactive            | Commonly an interactive shell      |
| Privileges      | Limited to service requirements   | Based on assigned permissions      |

These are common patterns, not strict rules. A system account can have a login shell, and a regular account can be configured without one.

To inspect an account:

    getent passwd www-data
    getent passwd kiran

The `www-data` account may not exist on every Linux distribution.

**Security principle:** Give each service only the permissions it needs. Running every application as root increases the impact of a compromise.

### 4. UID Ranges and Distribution Differences

Linux uses numeric User IDs (UIDs) to identify user accounts. UID ranges are commonly used to distinguish the root account, system accounts, and regular user accounts.

#### Common UID Ranges

| UID        | Typical purpose                            |
| ---------- | ------------------------------------------ |
| `0`      | Root account                               |
| `1–999` | Often used for system and service accounts |
| `1000+`  | Often used for regular user accounts       |

These ranges are common conventions, not universal rules. The exact ranges depend on the Linux distribution and system configuration.

#### Why Do UID Ranges Differ?

Different Linux distributions may configure different UID allocation policies for system and regular accounts.

For example:

- **Ubuntu/Debian:** Regular user accounts commonly start at UID `1000`.
- **RHEL-based distributions:** Regular user accounts commonly start at UID `1000` on modern installations.
- **Customized systems:** Administrators can configure different ranges or assign specific UIDs manually.

System accounts may also have UIDs outside the ranges shown above. Therefore, do not identify an account's purpose using only its UID.

#### How to Check UID Range Settings

On many Linux distributions, the `/etc/login.defs` file contains default UID allocation settings.

Display the relevant settings:

    grep -E '^(UID_MIN|UID_MAX|SYS_UID_MIN|SYS_UID_MAX)' /etc/login.defs

Example output:

    UID_MIN                  1000
    UID_MAX                 60000
    SYS_UID_MIN               101
    SYS_UID_MAX               999

The values above are illustrative; your system may show different values.

| Setting         | Meaning                                                                 |
| --------------- | ----------------------------------------------------------------------- |
| `UID_MIN`     | Minimum UID for regular accounts created by tools that use this setting |
| `UID_MAX`     | Maximum UID for regular accounts created by tools that use this setting |
| `SYS_UID_MIN` | Minimum UID for system accounts                                         |
| `SYS_UID_MAX` | Maximum UID for system accounts                                         |

These settings are defaults used by certain account-management tools. They do not necessarily define the UIDs of every existing account or every centralized identity source.

#### How to Inspect Existing UIDs

Display local usernames and their UIDs:

    awk -F: '{print $1, $3}' /etc/passwd

Example output:

    root 0
    daemon 1
    www-data 33
    kiran 1000

The account names and IDs shown here are examples and may differ on your machine.

To inspect a specific account:

    id kiran

Or query the system's configured account databases:

    getent passwd kiran

**Important:** UID ranges help explain account allocation, but the account's actual purpose and configuration determine whether it is a system account, a regular user, or a service account.

### 5. How Linux Uses UID

Linux uses a **UID (User ID)** to identify the user associated with a process and to help determine whether that process can access files and other system resources.

Although users work with usernames, Linux uses numeric IDs internally for ownership and permission checks.

#### 1. File and Directory Ownership

Linux files and directories have a numeric owner UID and group GID.

To display numeric ownership:

    ls -ln app.txt

Example output:

    -rw-r----- 1 1000 1002 120 Oct 24 10:00 app.txt

Here:

- `1000` — UID of the file owner.
- `1002` — GID of the file's group.
- `rw-r-----` — permission bits controlling access for the owner, group, and others.

Linux compares the process's identity with the file's ownership and permissions when evaluating access.

#### 2. Process Ownership

Every Linux process runs with user identity information. Important identifiers include the real UID and effective UID.

Display process IDs and their user identities:

    ps -eo pid,uid,euid,comm

Example output:

    PID   UID  EUID  COMMAND
    1200  1000 1000  bash
    1300  0    0     system-service

The values are illustrative.

- **PID:** Identifies the process.
- **UID:** Shows the real user ID.
- **EUID:** Shows the effective user ID, which is important for many permission checks.
- **COMMAND:** Shows the command name.

A process can have different real and effective UIDs, depending on how it was started and configured.

#### 3. Access Control

Linux uses process credentials, file ownership, and permission rules to determine whether an operation is allowed.

For example, suppose `app.txt` belongs to UID `1000` and has these permissions:

    -rw------- 1 1000 1000 120 Oct 24 10:00 app.txt

Only the file owner has read and write permission through these standard permission bits. A process running under another UID ordinarily cannot read the file unless another applicable access mechanism grants permission.

Access can also be affected by supplementary groups, ACLs, capabilities, and other security controls.

#### 4. Root Privileges

The root account has UID `0`. Processes running with effective UID `0` generally have extensive administrative privileges, subject to applicable security restrictions.

For example:

    sudo cat /etc/shadow

This command requests elevated privileges to read the protected password-information file, assuming the user is authorized to use `sudo`.

**Security best practice:** Run applications with the least privileges they need rather than running everything as root.

#### 5. UID in Docker and DevOps

UIDs are especially important when applications run inside Docker containers and access files mounted from the host.

For example:

- A container application runs as UID `1001`.
- A host directory is owned by UID `1000`.
- The directory does not grant write permission to the container's user or its groups.

The application may receive a `Permission denied` error.

Useful commands for troubleshooting include:

    id
    ls -ln /path/to/directory
    ps -eo pid,uid,euid,comm

Check the container's configured user and the host directory's ownership and permissions before changing them.

**Key takeaway:** Linux uses UIDs to identify users associated with processes and to evaluate access to files and other resources. Understanding UID behavior helps troubleshoot permissions and configure services securely.

---

## 👥 Understanding GID (Group ID)

### 1. What Is a GID?

A **GID (Group ID)** is a numeric identifier assigned to a Linux group. Linux uses GIDs to identify groups internally and manage group-based access to files, directories, and other resources.

Groups allow administrators to grant permissions to multiple users without configuring each user individually.

For example, a group named `developers` might have GID `1002` and contain users such as `kiran`, `amit`, and `priya`.

To display group information for a user:

    id kiran

Example output:

    uid=1000(kiran) gid=1000(kiran) groups=1000(kiran),27(sudo),1002(developers)

Here:

- `uid=1000(kiran)` — the user's UID.
- `gid=1000(kiran)` — the user's primary GID.
- `1002(developers)` — a supplementary group.

To look up a group's GID:

    getent group developers

Example output:

    developers❌1002:kiran,amit,priya

The GID is `1002` in this example. Actual IDs and group memberships vary by system.

---

### 2. Primary Group

Every Linux user has a **primary group**. It is the default group associated with the user and is commonly used as the group owner of files the user creates.

For example, a user might have:

- Username: `kiran`
- UID: `1000`
- Primary group: `kiran`
- Primary GID: `1000`

Display the user's primary GID:

    id -g kiran

Display the primary group name:

    id -gn kiran

#### How the Primary Group Works

Suppose `kiran` creates a file named `app.txt`. In a typical configuration, the file's group ownership will be the user's primary group.

    touch app.txt
    ls -l app.txt

Example output:

    -rw-r--r-- 1 kiran kiran 0 Oct 24 10:00 app.txt

Here:

- `kiran` is the file owner.
- `kiran` is the group owner.

The actual group ownership can differ because of directory setgid permissions, ACLs, or other system configuration.

**Key point:** The primary group is the user's default group, but it does not have to share the username.

---

### 3. Supplementary Groups

**Supplementary groups** provide users with additional group memberships and associated permissions.

A user can belong to multiple supplementary groups while retaining the same primary group.

For example, `kiran` might have:

| Group          | Purpose                                                          |
| -------------- | ---------------------------------------------------------------- |
| `kiran`      | Primary group                                                    |
| `developers` | Shared project access                                            |
| `docker`     | Access to the Docker daemon on systems configured for this group |
| `sudo`       | Permission to use `sudo`, subject to system configuration      |

These are illustrative examples. Group names and privileges depend on the distribution and system configuration.

To display all groups for the current user:

    groups

To display groups for a specific user:

    groups kiran

To display numeric group IDs:

    id -G kiran

To display group names and IDs:

    id kiran

#### Adding a User to a Supplementary Group

For example, add `kiran` to the `developers` group:

    sudo usermod -aG developers kiran

The `-aG` options append the group to the user's supplementary group list.

**Important:** Omitting `-a` when using `-G` can replace the user's existing supplementary group memberships. Also, the user may need to start a new login session for the updated membership to take effect.

Membership in certain groups, such as `docker`, may grant powerful privileges. Only assign such memberships when appropriate.

---

### 4. How Linux Uses GID

Linux uses GIDs to identify group ownership and evaluate group-based access permissions.

#### 1. Group Ownership of Files

Every file and directory has a group owner represented internally by a GID.

Display file ownership:

    ls -ln app.txt

Example output:

    -rw-r----- 1 1000 1002 120 Oct 24 10:00 app.txt

Here:

- `1000` is the owner's UID.
- `1002` is the group's GID.
- `rw-r-----` defines the standard owner, group, and other permissions.

To display the names instead of numeric IDs:

    ls -l app.txt

#### 2. Group-Based Permissions

Linux permissions are divided into three categories:

| Category | Applies to                                               |
| -------- | -------------------------------------------------------- |
| Owner    | The file's owning user                                   |
| Group    | Processes that qualify through the file's owning group   |
| Others   | Other users not covered by the owner or group categories |

For example:

    sudo chown root:developers /opt/myapp
    sudo chmod 770 /opt/myapp

Here:

- `root` becomes the directory owner.
- `developers` becomes the group owner.
- `770` grants read, write, and execute permissions to the owner and group, with no permissions for others.

For directories, execute permission allows traversal. Users still need appropriate permissions on the relevant files and subdirectories.

#### 3. Shared Access for Teams

Suppose three developers need access to the same project directory.

Instead of granting permissions individually, an administrator can assign the directory to the `developers` group and configure suitable group permissions.

Users who qualify through that group can then access the shared directory according to its permissions.

#### 4. GID in Docker and DevOps

GIDs also matter when applications access shared directories or host-mounted volumes in Docker.

For example:

- A directory on the host belongs to GID `1002`.
- A container process runs with a different primary GID.
- The process is not a member of the owning group and has no other applicable write permission.

The application may encounter a `Permission denied` error.

Useful troubleshooting commands include:

    id
    ls -ln /path/to/directory
    getent group developers

Check the process's UID/GID, the directory's ownership, and its permissions before changing them.

**Key takeaway:** GIDs allow Linux to organize users into groups and manage shared access to files, directories, and resources. Understanding GIDs is essential for Linux administration, server security, and troubleshooting DevOps permission issues.

---

## 📄 User and Group Information

Linux stores local user and group account information in several configuration files. These files help the operating system identify users, manage group memberships, and configure password-related information.

The four important files are:

- `/etc/passwd` — user account information.
- `/etc/shadow` — protected password information and password-aging settings.
- `/etc/group` — group information and supplementary group membership lists.
- `/etc/gshadow` — protected group administration information.

---

### 1. `/etc/passwd`

The `/etc/passwd` file contains information about local user accounts, including usernames, UIDs, primary GIDs, home directories, and login shells.

Despite its name, it normally does not contain password hashes. The `x` field typically indicates that password information is stored separately in `/etc/shadow`.

View the file:

    cat /etc/passwd

To inspect a particular account:

    getent passwd "$USER"

#### Structure of `/etc/passwd`

Example entry:

    kiran❌1000:1000:Kiran Meshram:/home/kiran:/bin/bash

Each field is separated by a colon (`:`).

| Field                | Example           | Meaning                                                     |
| -------------------- | ----------------- | ----------------------------------------------------------- |
| Username             | `kiran`         | User account name                                           |
| Password placeholder | `x`             | Password information is typically stored in `/etc/shadow` |
| UID                  | `1000`          | Numeric user ID                                             |
| GID                  | `1000`          | Primary group ID                                            |
| GECOS                | `Kiran Meshram` | Optional descriptive information                            |
| Home directory       | `/home/kiran`   | User's home directory                                       |
| Login shell          | `/bin/bash`     | Shell configured for login                                  |

#### Why Is `/etc/passwd` Important?

- Maps usernames to numeric UIDs.
- Identifies each user's primary GID.
- Stores home directory and shell information.
- Helps administrators inspect local account configuration.

**Security note:** `/etc/passwd` is generally readable by all users because programs need to look up account information. Avoid placing secrets in this file.

---

### 2. `/etc/shadow`

The `/etc/shadow` file stores protected password information and password-aging settings for local accounts.

It is normally readable only by root and appropriately privileged processes.

Inspect it only when necessary:

    sudo less /etc/shadow

For routine account inspection, use commands such as `passwd -S` or `chage -l` instead of exposing password hashes.

#### Structure of `/etc/shadow`

Example entry:

    kiran:$y$j9T$example-hash:20000:0:99999:7:::

This is an illustrative example, not a real password hash.

The fields are separated by colons.

| Field           | Example                 | Meaning                                                      |
| --------------- | ----------------------- | ------------------------------------------------------------ |
| Username        | `kiran`               | Account name                                                 |
| Password field  | `$y$j9T$example-hash` | Password hash or a special value                             |
| Last changed    | `20000`               | Days since the Unix epoch when the password was last changed |
| Minimum age     | `0`                   | Minimum days before the password can be changed again        |
| Maximum age     | `99999`               | Maximum password age in days                                 |
| Warning period  | `7`                   | Days before expiry when a warning may be issued              |
| Inactive period | Empty                   | Inactivity period after password expiry                      |
| Account expiry  | Empty                   | Account expiry date, if configured                           |
| Reserved field  | Empty                   | Reserved for future use                                      |

The actual field values and password-aging policies depend on the account configuration.

#### Special Password Field Values

- An empty password field can have security implications and should not be assumed safe.
- A value such as `!` or `*` generally indicates that password-based Unix authentication is locked or unavailable for that account.
- A locked password does not necessarily disable all forms of access, such as SSH key authentication.

#### Useful Commands

Check password status:

    sudo passwd -S kiran

View password-aging information:

    sudo chage -l kiran

Change a user's password interactively:

    sudo passwd kiran

**Security note:** Never share `/etc/shadow` contents publicly. Password hashes are sensitive and may be targeted by offline password-cracking attempts.

---

### 3. `/etc/group`

The `/etc/group` file stores local group information, including group names, GIDs, and supplementary group members.

View the file:

    cat /etc/group

Look up a specific group:

    getent group developers

#### Structure of `/etc/group`

Example entry:

    developers❌1002:kiran,amit,priya

Each field is separated by a colon.

| Field                | Example              | Meaning                                                    |
| -------------------- | -------------------- | ---------------------------------------------------------- |
| Group name           | `developers`       | Group name                                                 |
| Password placeholder | `x`                | Group password information is generally managed separately |
| GID                  | `1002`             | Numeric group ID                                           |
| Member list          | `kiran,amit,priya` | Users listed as supplementary members of the group         |

#### Why Is `/etc/group` Important?

- Maps group names to numeric GIDs.
- Lists supplementary group members.
- Helps administrators manage shared file and directory access.
- Supports inspection of local group configuration.

**Important:** A user's primary group is identified by the GID field in `/etc/passwd`. The user does not have to appear in that group's member list in `/etc/group`.

To inspect a user's groups:

    id kiran
    groups kiran

---

### 4. `/etc/gshadow`

The `/etc/gshadow` file stores protected group administration information for local groups.

It may contain group password hashes, group administrators, and group member lists.

Inspect it only when needed:

    sudo less /etc/gshadow

#### Structure of `/etc/gshadow`

Example entry:

    developers:!::kiran,amit,priya

Each field is separated by a colon.

| Field                | Example              | Meaning                                                               |
| -------------------- | -------------------- | --------------------------------------------------------------------- |
| Group name           | `developers`       | Group name                                                            |
| Password field       | `!`                | Indicates that group-password authentication is locked or unavailable |
| Group administrators | Empty                | Users permitted to administer the group through supported mechanisms  |
| Member list          | `kiran,amit,priya` | Group member list maintained in this file                             |

The example is illustrative. The actual contents vary by system.

#### Why Is `/etc/gshadow` Important?

- Stores protected group administration data.
- Supports management of group administrators and members.
- Separates sensitive group information from the generally readable `/etc/group` file.

Most administrators manage group membership through commands such as:

    sudo groupadd developers
    sudo usermod -aG developers kiran
    sudo gpasswd -a kiran developers

These commands modify account or group configuration. Use them carefully, particularly on production systems.

**Security note:** Avoid displaying or sharing `/etc/gshadow` contents unnecessarily.

---

### 5. Understanding User and Group Entries

The four files work together to provide local account and group information.

#### Example User Entry

In `/etc/passwd`:

    kiran❌1000:1000:Kiran Meshram:/home/kiran:/bin/bash

This tells us:

- The username is `kiran`.
- The UID is `1000`.
- The primary GID is `1000`.
- The home directory is `/home/kiran`.
- The configured login shell is `/bin/bash`.

#### Example Group Entry

In `/etc/group`:

    developers❌1002:kiran,amit,priya

This tells us:

- The group name is `developers`.
- The GID is `1002`.
- The listed supplementary members are `kiran`, `amit`, and `priya`.

If `kiran` belongs to this group, the group can provide additional access wherever permissions are granted to GID `1002`.

#### How the Files Relate

| File             | Main purpose                                               |
| ---------------- | ---------------------------------------------------------- |
| `/etc/passwd`  | User identity, UID, primary GID, home directory, and shell |
| `/etc/shadow`  | Protected password information and password-aging settings |
| `/etc/group`   | Group identity, GID, and supplementary member list         |
| `/etc/gshadow` | Protected group administration information                 |

#### Useful Inspection Commands

Display the current user's account entry:

    getent passwd "$USER"

Display the current user's group information:

    id

Look up a group:

    getent group sudo

Display local usernames and UIDs:

    awk -F: '{print $1, $3}' /etc/passwd

Display local group names and GIDs:

    awk -F: '{print $1, $3}' /etc/group

The `getent` command is often preferable when querying configured identity sources because accounts and groups may come from centralized services in addition to local files.

#### Hands-on Practice

1. Find your username, UID, primary GID, home directory, and shell.
2. Inspect your account with `getent passwd "$USER"`.
3. Display your group memberships using `id` and `groups`.
4. Look up a group using `getent group sudo`.
5. Compare the information returned by `getent` with the local entries in `/etc/passwd` and `/etc/group`.
6. Use `sudo passwd -S "$USER"` to inspect your password status without displaying password hashes.

**Key takeaway:** These files form an important part of traditional Linux local account management. Understanding their fields helps troubleshoot account configuration, group membership, authentication, and permission issues in Linux and DevOps environments.

---

## 🔐 User Identity and File Ownership

---

## 1. How UID and GID Relate to File Ownership

Linux uses **UID (User ID)** and **GID (Group ID)** to identify the user and group that own a file or directory.

Every file and directory has:

- **Owner UID:** Identifies the owning user.
- **Group GID:** Identifies the owning group.
- **Permissions:** Define access for the owner, group, and others.

### Example

Run:

    ls -l app.txt

Example output:

    -rw-r----- 1 kiran developers 120 Oct 24 10:00 app.txt

Here:

- `kiran` is the owner.
- `developers` is the group owner.
- `rw-` gives the owner read and write permissions.
- `r--` gives the group read permission.
- `---` gives others no permissions through the standard permission bits.

To display numeric ownership:

    ls -ln app.txt

Example output:

    -rw-r----- 1 1000 1002 120 Oct 24 10:00 app.txt

Here, UID `1000` identifies the owner and GID `1002` identifies the group.

### Changing Ownership

Use `chown` to change the owner or group of a file.

Change the owner:

    sudo chown amit app.txt

Change the group:

    sudo chown :developers app.txt

Change both owner and group:

    sudo chown amit:developers app.txt

Verify the changes:

    ls -l app.txt

**Important:** Changing ownership may require administrative privileges. Always verify the target path before running commands on production systems.

---

## 2. Numeric IDs vs Usernames and Group Names

Linux displays usernames and group names to make ownership easier for humans to understand. Internally, filesystem ownership is represented by numeric IDs.

For example, these commands display the same ownership in different formats:

    ls -l app.txt
    ls -ln app.txt

Example output from `ls -l`:

    -rw-r--r-- 1 kiran developers 120 Oct 24 10:00 app.txt

Example output from `ls -ln`:

    -rw-r--r-- 1 1000 1002 120 Oct 24 10:00 app.txt

The numeric IDs are used to determine ownership. Linux resolves those IDs to names using its configured account databases.

### What Happens When a Username Changes?

If the username `kiran` changes to `kiran-dev` but the UID remains `1000`, files owned by UID `1000` remain associated with that same numeric identity.

The displayed username changes because Linux resolves UID `1000` to the new account name.

### Why This Matters in DevOps

Numeric ownership is especially important when:

- Moving files between Linux servers.
- Restoring backups.
- Sharing directories between host systems and Docker containers.
- Troubleshooting permission errors.
- Managing application service accounts.

**Key point:** The numeric UID and GID determine filesystem ownership; usernames and group names are readable representations of those IDs.

---

## 3. What Happens When a User or Group Is Deleted?

Deleting a user or group does not automatically remove every file associated with its numeric ID.

### Deleting a User

For example:

    sudo userdel kiran

This deletes the local user account, but ordinarily does not remove the user's home directory or all files owned by that user.

To delete the account and its home directory:

    sudo userdel -r kiran

Use `-r` carefully. It can remove the user's home directory and mail spool, and the action may be difficult to reverse.

Files owned by the deleted user's UID may remain elsewhere on the filesystem.

### What Happens to the Files?

Suppose a file is owned by UID `1000`.

Before deleting the user:

    -rw-r--r-- 1 kiran developers 120 Oct 24 10:00 app.txt

After deleting the user, the file may still be owned by UID `1000`. If no account resolves that UID, Linux tools may display the number instead of a username.

If another account is later assigned UID `1000`, the file may appear to belong to that account.

This can create unexpected access or ownership issues.

### Deleting a Group

Delete a group using:

    sudo groupdel developers

The group cannot ordinarily be deleted if it is the primary group of an existing user.

Deleting a group does not automatically change the GID stored on files owned by that group. Those files may display the numeric GID when no group name resolves it.

### Best Practices Before Deleting Accounts

1. Identify files owned by the user's UID or the group's GID.
2. Check whether applications or services depend on the account.
3. Back up important data.
4. Decide whether files should be retained, transferred, or removed.
5. Verify ownership and permissions after the change.

For example, search a filesystem for files owned by a particular UID:

    sudo find / -xdev -uid 1000 -ls 2>/dev/null

Search for files owned by a particular GID:

    sudo find / -xdev -gid 1002 -ls 2>/dev/null

The `-xdev` option limits the search to the filesystem containing `/`. Additional mounted filesystems may need separate checks.

**Important:** Never delete an account or group on a production server without first checking its dependencies and file ownership.

---

## 4. Common UID/GID Permission Issues

UID and GID mismatches are common causes of file access problems on Linux servers and in containerized applications.

### Issue 1: Permission Denied

A user attempts to write to a file but receives:

    Permission denied

Possible causes:

- The file belongs to another UID.
- The user is not covered by the file's group permissions.
- The relevant permission bits do not allow writing.
- ACLs or other security controls restrict access.

Troubleshooting commands:

    id
    ls -ln app.txt
    stat app.txt

Check the process identity, numeric ownership, and permissions before making changes.

### Issue 2: Docker Volume Permission Problems

Suppose:

- A host directory is owned by UID `1000` and GID `1000`.
- A container application runs as UID `1001` and GID `1001`.
- The directory does not grant write access to that process through its owner, group, or other permissions.

The application may fail to write files to the mounted directory.

Inspect the host directory:

    ls -ln /path/to/directory

Inspect the container's user identity:

    docker exec <container_name> id

Possible solutions include:

- Running the application with an appropriate UID/GID.
- Configuring suitable group ownership and permissions.
- Using ACLs when appropriate.
- Adjusting the container's volume and user configuration.

Avoid automatically applying `chmod 777`. It grants broad access and can create security risks.

### Issue 3: Files Show Numeric Owners

A file listing displays:

    -rw-r--r-- 1 1005 1005 120 Oct 24 10:00 app.txt

This can happen when the system cannot resolve UID `1005` or GID `1005` to account names.

Check whether the IDs exist:

    getent passwd 1005
    getent group 1005

If no matching account or group is found, investigate whether the account was deleted, the file was copied from another system, or the identity service is unavailable.

Do not assume that the file is corrupted. Its numeric ownership may be valid even if the corresponding account name is unavailable.

### Issue 4: User Added to a Group but Access Still Fails

A user is added to a supplementary group:

    sudo usermod -aG developers kiran

However, the user's existing terminal session may not reflect the updated group membership.

Check membership:

    id kiran
    getent group developers

Start a new login session and verify again. Existing processes generally retain the group credentials they started with.

Also check the target file's group ownership and permissions:

    ls -l /opt/myapp

### Issue 5: Incorrect Ownership After Copying Files

Files copied from another server may retain numeric UIDs and GIDs that do not match the intended accounts on the destination.

Inspect ownership:

    ls -ln /path/to/files

If ownership is incorrect, determine the intended owner and group before changing them.

For example:

    sudo chown -R kiran:developers /opt/myapp

**Warning:** Recursive ownership changes affect every file and directory under the specified path. Verify the path and the intended ownership before executing this command.

### Troubleshooting Checklist

When investigating a UID/GID-related problem:

1. Run `id` to inspect the current user's identity and group memberships.
2. Run `ls -ln` to inspect numeric file ownership.
3. Run `stat` to inspect file metadata.
4. Use `getent passwd` and `getent group` to check account and group resolution.
5. Inspect the permissions and any relevant ACLs.
6. Check whether a container or service runs under a different UID/GID.
7. Change ownership or permissions only after identifying the cause.

Understanding UID and GID behavior helps administrators resolve

---

## ⚙️ Commands to Inspect User Accounts and IDs

### 1. `id`

Displays the current user's UID, primary GID, and group memberships.

    id
    id kiran

### 2. `whoami`

Displays the username associated with the current effective user ID.

    whoami

### 3. `getent`

Retrieves user and group information from configured system databases.

    getent passwd kiran
    getent group developers

### 4. `ls -ln`

Lists files with detailed information, displaying numeric UID and GID instead of usernames and group names.

    ls -ln
    ls -ln app.txt

### 5. `stat`

Displays detailed file information, including ownership, permissions, size, and timestamps.

    stat app.txt
    stat -c '%u %g' app.txt

---

## 🌐 Why User Accounts and UID/GID Matter in DevOps

User accounts, UIDs, and GIDs are essential for managing access, securing servers, and running applications reliably.

- **Server Security:** Restrict SSH access and administrative privileges to authorized users.
- **File Permissions:** Control who can read, write, or execute application files.
- **Application Services:** Run applications using dedicated service accounts instead of root.
- **Docker Containers:** Troubleshoot permission issues involving container users and host-mounted volumes.
- **CI/CD Pipelines:** Configure appropriate permissions for build and deployment processes.
- **Team Collaboration:** Use groups to provide shared access to project directories and deployment files.
- **Troubleshooting:** Identify ownership mismatches that cause `Permission denied` errors.

**Real-world example:** A Dockerized application cannot write logs to a host directory because the container's UID does not have write permission. Checking `id`, `ls -ln`, and `stat` helps identify the ownership mismatch and determine the appropriate fix.

**Key takeaway:** Understanding Linux users, groups, UIDs, and GIDs helps DevOps engineers manage server access, secure applications, and resolve permission issues efficiently.

---

## 💻 Hands-on Practice

### 1. Inspect Your User Account

Use the following commands to inspect your current user account.

    whoami
    id
    getent passwd "$USER"

Identify your username, UID, primary GID, home directory, and login shell.

### 2. Find Your UID and GID

Run these commands to display your numeric identifiers.

    id -u
    id -g
    id -G
    groups

Identify your UID, primary GID, and supplementary group memberships.

### 3. Examine User and Group Database Entries

Inspect your user account and group information.

    getent passwd "$USER"
    getent group "$(id -gn)"
    cat /etc/passwd
    cat /etc/group

Identify the fields containing usernames, UIDs, GIDs, home directories, and group memberships.

### 4. Inspect Numeric File Ownership

Create a sample file and inspect its ownership.

    touch practice.txt
    ls -l practice.txt
    ls -ln practice.txt
    stat practice.txt
    stat -c '%u %g' practice.txt

Compare the username and group name with their numeric UID and GID.

### 🧪 Mini Lab

**Scenario:** You are troubleshooting a Linux server where an application cannot access a file.

1. Find your current UID and GID.
2. Create a file named `app.log`.
3. Inspect its ownership using `ls -l` and `ls -ln`.
4. Use `stat` to display its numeric UID and GID.
5. Find your account entry using `getent passwd "$USER"`.
6. Explain how incorrect UID/GID ownership could cause a `Permission denied` error.

**Expected outcome:** You should be able to inspect user accounts, identify UID/GID values, and understand how Linux uses ownership and permissions to control file access.

---

## 🎯 Interview Questions

1. What is a Linux user account?
2. What is the difference between UID and GID?
3. What is the UID of the root user?
4. What is the difference between system users and regular users?
5. What is the difference between primary and supplementary groups?
6. What information is stored in `/etc/passwd` and `/etc/shadow`?
7. What is the purpose of `/etc/group` and `/etc/gshadow`?
8. What is the difference between local and centralized user accounts?
9. How does Linux use UID and GID to manage file ownership?
10. What happens to file ownership when a user or group is deleted?
11. What is the difference between `id` and `whoami`?
12. What is the purpose of the `getent` command?
13. What is the difference between `ls -l` and `ls -ln`?
14. How can you check a file's numeric UID and GID?
15. Why can UID/GID mismatches cause permission issues in Docker containers?

---

## 📝 Summary

- Linux user accounts represent people, applications, and system services.
- Every user has a UID, and every group has a GID.
- UID `0` identifies the root user.
- Users have a primary group and can belong to supplementary groups.
- `/etc/passwd`, `/etc/shadow`, `/etc/group`, and `/etc/gshadow` store local account and group information.
- Linux uses numeric UIDs and GIDs to manage file ownership and access permissions.
- Deleting a user or group does not automatically remove all files associated with its numeric ID.
- Commands such as `id`, `whoami`, `getent`, `ls -ln`, and `stat` help inspect accounts, IDs, and file ownership.
- Understanding UIDs and GIDs is essential for Linux administration, server security, and troubleshooting DevOps permission issues.

---

## 📚 References

- [Linux `id` command](https://man7.org/linux/man-pages/man1/id.1.html)
- [Linux `passwd` file format](https://man7.org/linux/man-pages/man5/passwd.5.html)
- [Linux `shadow` file format](https://man7.org/linux/man-pages/man5/shadow.5.html)
- [Linux `group` file format](https://man7.org/linux/man-pages/man5/group.5.html)
- [Linux `gshadow` file format](https://man7.org/linux/man-pages/man5/gshadow.5.html)
- [Linux `getent` command](https://man7.org/linux/man-pages/man1/getent.1.html)
- [Linux `chown` command](https://man7.org/linux/man-pages/man1/chown.1.html)

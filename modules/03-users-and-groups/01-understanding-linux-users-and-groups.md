# 👥 Understanding Linux Users and Groups

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:

- Understand Linux users and groups and why they are needed.
- Identify root, regular, and system users.
- Explain primary and supplementary groups.
- Understand the relationship between users, groups, and file ownership.
- Recognize how users and groups support Linux security and access control.
- Identify the basic commands used to inspect users and group memberships.

---

## 📖 Introduction

Linux is a multi-user operating system, meaning multiple users can access and use the same system while maintaining separate identities and permissions.

To manage access securely, Linux uses **users and groups**. Users represent individual accounts, while groups allow administrators to organize users and manage shared access to files, directories, and resources.

For example, on a Linux server, a developer may need access to application files, while a system administrator needs additional privileges to manage the server. Linux users, groups, and permissions help enforce these access boundaries.

Understanding users and groups is essential for Linux administration, server security, and DevOps tasks such as managing application services, deployment permissions, and shared directories.

---

## 👤 What Are Linux Users?

### 1. Definition of a User

A **Linux user** is an account that provides an identity for a person, application, or system service interacting with the operating system.

Linux uses user accounts to control access to files, directories, processes, and system resources.

Each user account has a unique **User ID (UID)**, which Linux uses internally to identify the user.

**Example:**

    whoami
    id

- `whoami` displays the effective username of the current shell.
- `id` displays the current user's UID, primary group, and supplementary groups.

Users help Linux maintain security by separating access and privileges between accounts.

### 2. Types of Users: Root, Regular, and System Users

Linux commonly has three categories of user accounts.

#### Root User

The **root user** is the superuser with UID `0`. It has extensive privileges to administer the system, including managing users, changing system configuration, and accessing protected files.

Example:

    id root

The root account should be used carefully because mistakes made with its privileges can affect the entire system.

#### Regular Users

Regular users are accounts intended for people who use the system for development, administration, or everyday tasks.

They typically have limited privileges and access to their own files and authorized resources.

Example:

    whoami
    id

#### System Users

System users are commonly created to run services or applications, such as web servers, databases, and other background processes.

They often have restricted permissions and may not have an interactive login shell.

Examples of common service accounts include `www-data` on Ubuntu and `sshd` on systems that use that account.

Inspect a service account with:

    id www-data

The account may not exist on every Linux distribution.

**Important:** UID ranges and the distinction between system and regular accounts vary by distribution and configuration. Do not rely on UID ranges alone to determine an account's purpose.

### 3. User Identity and Authentication

Linux uses user identity to determine which permissions and privileges apply to a process.

#### User Identity

Each account has a UID, while its username provides a human-readable name.

For example:

    id kiran

Illustrative output:

    uid=1000(kiran) gid=1000(kiran) groups=1000(kiran),27(sudo)

This indicates:

- `uid=1000(kiran)`: The user's UID is `1000`.
- `gid=1000(kiran)`: The primary group ID is `1000`.
- `groups=...`: The groups associated with the user in this context.

Your actual output may differ.

#### Authentication

**Authentication** is the process of verifying a user's identity before granting access.

Linux systems may use several authentication methods, including:

- Password authentication.
- SSH public-key authentication.
- Centralized identity services, such as LDAP or Active Directory integrations.
- Other configured authentication mechanisms.

For example, when connecting to a Linux server over SSH, the server verifies the login credentials according to its authentication configuration.

#### Authentication vs Authorization

These concepts serve different purposes:

| Concept        | Meaning                                        |
| -------------- | ---------------------------------------------- |
| Authentication | Verifies who or what is connecting             |
| Authorization  | Determines what that identity is allowed to do |

A user may successfully authenticate to a server but still lack permission to read a protected file or execute an administrative command.

**DevOps relevance:** Understanding user identity and authentication is essential when configuring SSH access, managing service accounts, securing deployments, and applying least-privilege access to Linux servers.

---

## 👥 What Are Linux Groups?

### 1. Definition of a Group

A **Linux group** is a collection of user accounts used to manage permissions for multiple users at once.

Instead of assigning permissions to every user individually, administrators can create a group and grant permissions to that group. All users who belong to the group can receive the same group-level access.

**Example:**

Suppose a development team has three users: `kiran`, `amit`, and `priya`. All three need access to the same project directory.

Instead of configuring access for each user separately, an administrator can create a group named `developers` and add all three users to it.

    sudo groupadd developers
    sudo usermod -aG developers kiran
    sudo usermod -aG developers amit
    sudo usermod -aG developers priya

Verify group membership:

    getent group developers

Example output:

    developers❌1002:kiran,amit,priya

Here:

- `developers` — group name.
- `x` — placeholder for legacy group-password information.
- `1002` — group ID (GID).
- `kiran,amit,priya` — supplementary members listed in the group database.

The GID may differ on your system.

**Key point:** A group makes it easier to manage permissions for multiple users without configuring each user separately.

### 2. Primary and Supplementary Groups

Every Linux user has a **primary group** and can belong to zero or more **supplementary groups**.

#### Primary Group

The primary group is the default group associated with a user's processes and, in typical configurations, newly created files.

Example:

    id kiran

Output:

    uid=1000(kiran) gid=1000(kiran) groups=1000(kiran),27(sudo),1002(developers)

In this example:

- `gid=1000(kiran)` — the primary group is `kiran`.
- The `groups=` list includes the primary group and supplementary groups.
- `sudo` and `developers` are supplementary groups.

A user's primary group does not have to share the user's username, although that is common on many Linux distributions.

#### Supplementary Groups

Supplementary groups provide additional permissions without changing the user's primary group.

For example, `kiran` may have:

- Primary group: `kiran`
- Supplementary group: `developers`
- Supplementary group: `sudo`

Each group may provide access to different files, directories, devices, or administrative capabilities, depending on system configuration.

To add a user to a supplementary group:

    sudo usermod -aG developers kiran

**Important:** The `-a` option means append, while `-G` specifies supplementary groups. Omitting `-a` when using `-G` can replace the user's existing supplementary group list.

You may need to start a new login session for the updated membership to take effect in your session.

To view group memberships:

    groups kiran
    id kiran

### 3. Why Groups Are Needed

Linux groups simplify permission management and improve security.

#### 1. Shared File and Directory Access

Multiple users can access the same project files through a shared group.

Example:

    sudo chown root:developers /opt/myapp
    sudo chmod 770 /opt/myapp

Here:

- `root` owns the directory.
- `developers` is the group owner.
- `770` grants read, write, and execute permissions to the owner and group, with no permissions for others.

For directories, execute permission allows users to traverse the directory. Users still need suitable permissions on files and subdirectories to perform specific operations.

#### 2. Easier Permission Management

Without groups, administrators may need to configure access separately for every user.

With groups, permissions can be assigned once to a group, and users can be added to or removed from that group as team membership changes.

#### 3. Better Security

Groups help administrators follow the **principle of least privilege** by granting only the access users need.

For example, developers might receive access to application files, while database administrators receive access to database-related resources.

Being a member of a group does not automatically grant administrative privileges; the permissions assigned to that group determine the access it provides.

#### 4. DevOps and Server Administration

Groups are useful in real-world environments for:

- Giving developers shared access to application directories.
- Managing access to logs and deployment files.
- Controlling access to selected devices and resources.
- Managing permissions for teams working on shared servers.
- Supporting controlled access for CI/CD deployment processes.

**Remember:**

- A **user** represents an account.
- A **group** organizes users for shared permission management.
- A **primary group** is the user's default group.
- **Supplementary groups** provide additional group-based access.
- File ownership and permissions determine what users can actually do.

---

## 🔐 User and Group Relationship

### 1. How Users Belong to Groups

In Linux, users can belong to one or more groups. Groups help administrators organize users and manage access to shared resources.

Each user has a **primary group** and may also belong to multiple **supplementary groups**.

- **Primary group:** The default group associated with a user's processes and, in typical configurations, newly created files.
- **Supplementary groups:** Additional groups that provide access to shared files, directories, or other resources.

View the current user's identity and group memberships:

    id

Display group memberships:

    groups

Display the groups of a specific user:

    groups kiran

Replace `kiran` with an existing username on your system.

### 2. User Ownership and Access Control

Linux associates files and directories with an owner user and an owner group. Permissions determine what the owner, members of the group, and other users can do.

For example, inspect a file:

    ls -l app.txt

Example output:

    -rw-r----- 1 kiran developers 120 Oct 22 10:00 app.txt

This indicates:

- `kiran`: The file owner.
- `developers`: The file's owner group.
- `rw-`: The owner can read and write.
- `r--`: Members of the owner group can read.
- `---`: Other users have no permission through these basic permission bits.

The permissions shown here are illustrative. Access can also be affected by ACLs and other security mechanisms.

Linux uses ownership and permissions to prevent unauthorized access and enable controlled collaboration.

### 3. Real-World Example

Suppose a development team has three users:

- `kiran`
- `amit`
- `priya`

All three need access to a shared application directory. Instead of managing permissions separately for every user, an administrator can create a group named `developers` and add the users to it.

Conceptually, the setup looks like this:

    developers
    ├── kiran
    ├── amit
    └── priya

The shared directory can be owned by the `developers` group:

    sudo chown -R root:developers /opt/myapp
    sudo chmod -R 770 /opt/myapp

With these example permissions, the owner and group receive read, write, and execute permissions, while others receive none. Directory execute permission allows traversal; file execute permission allows execution.

**Important:** Recursive permission changes affect every file and directory under the specified path. In a real deployment, inspect existing ownership and permissions before applying them.

This approach makes team access easier to manage. Administrators can grant or revoke shared access by changing group membership, while Linux permissions control what group members can do.

---

## 🆔 Introduction to UID and GID

Linux uses numeric identifiers to identify users and groups internally. These identifiers are called **UID (User ID)** and **GID (Group ID)**.

Usernames and group names make the system easier for humans to understand, while Linux uses UIDs and GIDs to manage ownership and permissions.

---

### 1. What Is UID (User ID)?

A **UID (User ID)** is a numeric identifier assigned to a Linux user account.

Every user account has a UID. Linux uses this number to identify the user when checking file permissions, process ownership, and access to system resources.

For example, run:

    id kiran

Example output:

    uid=1000(kiran) gid=1000(kiran) groups=1000(kiran),27(sudo)

Here:

- `uid=1000` — the user's UID is `1000`.
- `kiran` — the corresponding username.
- `gid=1000` — the primary group's GID is `1000`.
- `27(sudo)` — the user also belongs to the `sudo` group.

The numbers shown above are examples; actual IDs depend on your system.

#### Common UID Categories

| UID        | Typical purpose                          |
| ---------- | ---------------------------------------- |
| `0`      | Root user                                |
| `1–999` | Often used by system or service accounts |
| `1000+`  | Often used by regular users              |

**Note:** UID ranges vary by Linux distribution and system configuration. Do not rely on the number alone to determine an account's purpose.

To check the current user's UID:

    id -u

To check another user's UID:

    id -u kiran

---

### 2. What Is GID (Group ID)?

A **GID (Group ID)** is a numeric identifier assigned to a Linux group.

Groups organize users so that permissions can be assigned to multiple users together.

For example, run:

    getent group developers

Example output:

    developers❌1002:kiran,amit,priya

Here:

- `developers` — group name.
- `x` — placeholder for legacy group-password information.
- `1002` — the group's GID.
- `kiran,amit,priya` — members listed in the group database.

To display the current user's primary GID:

    id -g

To display all group IDs associated with the current user:

    id -G

To display group names along with IDs:

    id

To look up a group's GID:

    getent group developers

---

### 3. Difference Between UID and GID

| Feature         | UID                         | GID                                  |
| --------------- | --------------------------- | ------------------------------------ |
| Full form       | User ID                     | Group ID                             |
| Identifies      | A user account              | A group                              |
| Used for        | User identity and ownership | Group membership and group ownership |
| Example         | `1000`                    | `1002`                             |
| Related command | `id -u`                   | `id -g`                            |

---

### 4. How UID and GID Work with File Ownership

Linux files and directories store numeric ownership information for a user and a group.

Run:

    ls -l app.txt

Example output:

    -rw-r----- 1 kiran developers 120 Oct 22 10:00 app.txt

This output displays the owner and group **names**. Internally, Linux uses their numeric IDs to determine ownership and permissions.

For example:

- Owner: `kiran`, whose UID might be `1000`.
- Group: `developers`, whose GID might be `1002`.
- Owner permissions: `rw-` — read and write.
- Group permissions: `r--` — read only.
- Other users: `---` — no permissions through the standard Unix permission bits.

You can inspect the numeric ownership with:

    ls -ln app.txt

Example output:

    -rw-r----- 1 1000 1002 120 Oct 22 10:00 app.txt

Here, `1000` is the owner's UID and `1002` is the group's GID.

**Important:** File ownership is stored using numeric IDs. If an account's username or a group's name changes while its numeric ID remains the same, the underlying ownership does not change.

---

### 5. Why UID and GID Matter in DevOps

UIDs and GIDs are important for Linux administration, server security, Docker, and deployment workflows.

- **File permissions:** Linux uses numeric IDs to determine ownership and access.
- **Service accounts:** Applications often run under dedicated users with limited privileges.
- **Docker containers:** A process running as a particular UID/GID may encounter permission issues when accessing mounted files.
- **Shared directories:** Matching group ownership and permissions allows multiple users or processes to collaborate.
- **Security:** Dedicated IDs and restricted permissions help prevent applications from accessing resources they do not need.

For example, if a Docker container writes to a mounted directory as UID `1001`, but the host directory is writable only by another UID and its permitted group, the write operation may fail.

Understanding UID/GID mapping helps troubleshoot such permission errors.

---

### 6. Hands-on Practice

Run these commands in your Linux terminal:

    # Display the current username
    whoami

    # Display UID, primary GID, and group memberships
    id

    # Display the current user's UID
    id -u

    # Display the current user's primary GID
    id -g

    # Display all associated group IDs
    id -G

    # Display numeric ownership of files
    ls -ln

    # Look up a user account
    getent passwd "$USER"

    # Look up a group
    getent group sudo

**Practice task:**

1. Find your UID and primary GID.
2. Identify your supplementary groups.
3. Display the numeric ownership of a file.
4. Compare the numeric IDs with the owner and group names shown by `ls -l`.

---

## 🎯 Interview Questions

1. What is a UID in Linux?
2. What is a GID in Linux?
3. What UID is assigned to the root user?
4. What is the difference between a primary GID and supplementary groups?
5. How do you find your current UID and GID?
6. How does Linux use UID and GID when checking file permissions?
7. Why might a Docker container encounter permission errors when accessing a host-mounted directory?

---

## 📝 Summary

- **UID** identifies a Linux user account numerically.
- **GID** identifies a Linux group numerically.
- Root normally has UID `0`.
- Users have a primary group and may belong to supplementary groups.
- Linux uses numeric ownership IDs to enforce file and directory permissions.
- Understanding UID/GID is essential for troubleshooting permissions on Linux servers and in Docker environments.

---

## 📚 References

- [Linux `id` command](https://man7.org/linux/man-pages/man1/id.1.html)
- [Linux `passwd` file format](https://man7.org/linux/man-pages/man5/passwd.5.html)
- [Linux `group` file format](https://man7.org/linux/man-pages/man5/group.5.html)
- [Linux `chown` command](https://man7.org/linux/man-pages/man1/chown.1.html)

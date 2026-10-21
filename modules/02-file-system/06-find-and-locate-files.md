# 🔎 Finding and Locating Files in Linux

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:

- Understand how file searching works in Linux.
- Use different commands to find files and directories.
- Explain the difference between searching the filesystem and searching an index.
- Identify files using names, types, sizes, and modification times.
- Understand when to use `find`, `locate`, `which`, and `whereis`.
- Apply file-search commands to common Linux administration and DevOps tasks.

---

## 📖 Introduction

In Linux, files and directories are spread across many locations, such as `/etc`, `/home`, `/var`, `/usr`, and application-specific directories.

As a system administrator or DevOps engineer, you may need to locate configuration files, application logs, scripts, executables, or files consuming too much disk space.

Manually browsing every directory is inefficient. Linux provides command-line utilities that help you search for files quickly and accurately.

For example, you might need to:

- Find an application configuration file.
- Locate a log file that was recently modified.
- Identify large files consuming disk space.
- Search for a shell script in a project directory.
- Find the location of an installed command.

In this chapter, you will learn practical file-searching techniques and understand which command is appropriate for each situation.

---

## 🔍 Understanding File Searching in Linux

Linux provides several commands for finding files, but each works differently and serves a particular purpose.

### 1. Searching the Filesystem

Some commands search directories and inspect filesystem entries directly.

For example, `find` searches a specified directory tree and can filter results using conditions such as filename, file type, size, permissions, and modification time.

Example:

    find ~/linux-practice -name "*.txt"

This searches for files and directories whose names end in `.txt` within `~/linux-practice` and its subdirectories.

### 2. Searching an Index

Other commands use a previously created database or index of file paths.

For example, `locate` searches an index rather than normally traversing the entire filesystem for each query.

Example:

    locate passwd

This searches the locate database for paths matching `passwd`. Results depend on the database contents and configuration, so recently created files may not appear until the database is updated.

### 3. Searching for Executable Commands

Sometimes you are not looking for an ordinary file; you want to know where a command is installed.

Linux provides utilities such as `which` and `whereis` for this purpose.

Examples:

    which python3
    whereis python3

These commands help identify executable locations and, depending on the utility, related files such as manuals or source files.

### 4. Choosing the Right Search Method

| Requirement                                   | Common command |
| --------------------------------------------- | -------------- |
| Search directories using conditions           | `find`       |
| Search a file-path database quickly           | `locate`     |
| Find a command resolved through `PATH`      | `which`      |
| Find executable, source, and manual locations | `whereis`    |

**Key takeaway:** Choose the command based on what you need to find. Use `find` when you need filesystem-based searches with detailed conditions, `locate` for quick indexed searches, and `which` or `whereis` when investigating installed commands.

---

## 📂 Finding Files with `find`

### 1. What Is the `find` Command?

The `find` command is a powerful Linux utility used to search for files and directories based on specific conditions.

Unlike indexed search tools, `find` searches the specified directory tree directly, making it useful when you need to search for recently created files or apply detailed filters.

**Common use cases:**

- Finding files by name or extension.
- Searching for directories.
- Finding large files.
- Identifying recently modified files.
- Searching by permissions or ownership.
- Executing commands on matching files.

---

### 2. Basic `find` Command Syntax

The basic syntax is:

    find [path] [conditions] [actions]

Where:

- `path`: The directory where the search begins.
- `conditions`: Rules used to match files or directories.
- `actions`: Operations performed on matching results.

**Example:**

    find /home/user -name "notes.txt"

This searches `/home/user` and its subdirectories for entries named `notes.txt`.

**Search the current directory:**

    find . -name "notes.txt"

**Search your home directory:**

    find ~ -name "notes.txt"

**Search the entire filesystem:**

    sudo find / -name "notes.txt" 2>/dev/null

Here, `sudo` runs the search with elevated privileges, and `2>/dev/null` discards error messages written to standard error, such as permission-denied messages. Use elevated privileges only when necessary.

---

### 3. Finding Files by Name

Use `-name` to search for an exact filename or a pattern.

**Find a specific file:**

    find . -name "app.js"

**Find all files named `config.txt`:**

    find ~/linux-practice -name "config.txt"

**Find files matching a wildcard pattern:**

    find . -name "*.txt"

The `*` wildcard matches zero or more characters, so this finds entries whose names end in `.txt`.

**Case-insensitive search:**

    find . -iname "README.md"

The `-iname` option matches the name without distinguishing uppercase and lowercase letters.

For example, it can match `README.md`, `readme.md`, and `Readme.md`.

**Important:** Put wildcard patterns in quotes so your shell does not expand them before `find` receives them.

---

### 4. Finding Files by Type

Use `-type` to restrict results to particular filesystem entry types.

| Option      | Meaning             |
| ----------- | ------------------- |
| `-type f` | Regular files       |
| `-type d` | Directories         |
| `-type l` | Symbolic links      |
| `-type b` | Block devices       |
| `-type c` | Character devices   |
| `-type p` | Named pipes         |
| `-type s` | Unix domain sockets |

**Find regular files:**

    find . -type f

**Find directories:**

    find . -type d

**Find symbolic links:**

    find . -type l

**Find directories named `logs`:**

    find /var -type d -name "logs"

Combining `-type` and `-name` helps narrow searches and avoid unnecessary results.

---

### 5. Finding Files by Extension

Linux does not require files to have extensions, but extensions are commonly used to organize files by purpose.

You can search for extension patterns with `-name`.

**Find all `.txt` files:**

    find . -type f -name "*.txt"

**Find JavaScript files:**

    find . -type f -name "*.js"

**Find Python files:**

    find . -type f -name "*.py"

**Find multiple extensions:**

    find . -type f \( -name "*.js" -o -name "*.jsx" \)

Here:

- `\(` and `\)` group conditions.
- `-o` means OR.
- Both patterns are evaluated as alternatives.

This example finds regular files ending in `.js` or `.jsx`.

---

### 6. Finding Files by Size

Use `-size` to search for files based on their size.

| Expression      | Meaning                                                |
| --------------- | ------------------------------------------------------ |
| `-size +100M` | Larger than 100 MiB                                    |
| `-size -10M`  | Smaller than 10 MiB                                    |
| `-size 50M`   | Size falls within the corresponding 50 MiB block range |
| `-empty`      | Empty files or directories                             |

GNU `find` uses units such as `k`, `M`, and `G` for size-based searches. Its size comparisons use rounded-up filesystem blocks, so `-size 50M` does not mean an exact byte-for-byte size of 50 MiB.

**Find files larger than 100 MiB:**

    find . -type f -size +100M

**Find files smaller than 10 MiB:**

    find . -type f -size -10M

**Find empty files:**

    find . -type f -empty

**Find empty directories:**

    find . -type d -empty

**Real-world example:** A DevOps engineer can search for large log files or build artifacts that may be consuming disk space.

    sudo find /var/log -type f -size +100M 2>/dev/null

Inspect the results before deleting or modifying any files.

---

### 7. Finding Files by Modification Time

Linux allows you to search for files based on when they were last modified.

The `-mtime` option measures modification time in whole 24-hour periods.

| Expression               | Meaning                                           |
| ------------------------ | ------------------------------------------------- |
| `-mtime -1`            | Modified less than 24 hours ago                   |
| `-mtime +7`            | Modified at least eight whole 24-hour periods ago |
| `-mmin -30`            | Modified less than 30 minutes ago                 |
| `-newer reference.txt` | Modified more recently than the reference file    |

The distinction between `-mtime -7` and `-mtime +7` matters because `find` rounds elapsed time down to whole 24-hour periods.

**Find files modified within the last 24 hours:**

    find . -type f -mtime -1

**Find files whose modification time is at least eight whole days ago:**

    find . -type f -mtime +7

**Find files modified in the last 30 minutes:**

    find . -type f -mmin -30

**Find files modified more recently than `reference.txt`:**

    find . -type f -newer reference.txt

These searches are useful when investigating recent changes, troubleshooting deployments, or locating recently updated configuration files.

---

### 8. Finding Files by Permissions

Use `-perm` to search for files based on permission bits.

**Find files with exactly `644` permissions:**

    find . -type f -perm 644

**Find files with permissions that include all bits represented by `644`:**

    find . -type f -perm -644

**Find files with any execute permission bit set:**

    find . -type f -perm /111

The distinctions are important:

- `-perm 644` matches the specified permission bits exactly.
- `-perm -644` requires all permission bits represented by `644` to be set.
- `-perm /111` matches when any of the owner, group, or others execute bits is set.

**Find world-writable files:**

    find . -type f -perm -002

This can help identify files that all local users may be able to modify, depending on other filesystem controls.

**Security note:** Finding a permission pattern does not automatically mean a file is vulnerable. Review ownership, directory permissions, ACLs, and the application's requirements before changing permissions.

---

### 9. Finding Files by Owner

Use `-user` to find entries owned by a particular user.

**Find files owned by the current user:**

    find . -user "$(whoami)"

**Find files owned by `root`:**

    sudo find /var -user root 2>/dev/null

**Find files belonging to a particular group:**

    find . -group developers

Replace `developers` with an existing group on your system.

You can combine ownership with other conditions.

**Example: Find large files owned by root:**

    sudo find /var -user root -type f -size +100M 2>/dev/null

This is useful for auditing ownership, troubleshooting permission problems, and locating files created by services.

---

### 10. Executing Commands on Search Results

The `find` command can perform actions on matching entries, not just display their paths.

One important action is `-exec`.

**Syntax:**

    find [path] [conditions] -exec command {} \;

Where:

- `-exec` starts the command to run.
- `{}` is replaced by each matching path.
- `\;` marks the end of the command expression.

**Example: Display details of every `.txt` file:**

    find . -type f -name "*.txt" -exec ls -l {} \;

**Example: Display the first line of each `.txt` file:**

    find . -type f -name "*.txt" -exec head -n 1 {} \;

For commands that can handle multiple paths at once, use `+` instead of `\;` to reduce the number of command invocations.

    find . -type f -name "*.txt" -exec ls -l {} +

**Example: Find empty files and print their paths:**

    find . -type f -empty -print

**Using `-delete` carefully:**

You can delete matching files directly:

    find . -type f -name "*.tmp" -print

First, inspect the results. If they are exactly the files you intend to remove, you can run:

    find . -type f -name "*.tmp" -delete

The first command only lists matching files; the second deletes them. Be careful with the search path and conditions, especially when working under system directories.

**DevOps tip:** Prefer previewing results before performing destructive operations. This helps prevent accidental deletion of important application files or logs.

---

## ⚡ Locating Files with `locate`

The `locate` command is a Linux utility used to find files and directories quickly by searching a database of stored pathnames.

Unlike `find`, which searches directory trees directly, `locate` normally searches an index. This makes it convenient when you want to find a file by name without traversing the filesystem each time.

**Example:**

    locate passwd

This searches the locate database for paths containing `passwd`.

**Common use cases:**

- Finding configuration files.
- Locating application files by name.
- Searching for files across multiple directories.
- Quickly discovering paths on a Linux system.

---

### 2. How `locate` Works

The `locate` utility uses a database containing file paths. A scheduled task or administrator may update this database periodically.

The basic workflow is:

1. The system scans eligible filesystem paths.
2. File paths are stored in a database.
3. You run `locate` with a search pattern.
4. The command returns matching paths from the database.

For example, if the database contains:

    /home/user/project/app.js
    /etc/nginx/nginx.conf
    /var/log/nginx/access.log

You can search for Nginx-related paths:

    locate nginx

**Important:** Because the database may not be current, `locate` can return paths that no longer exist or miss files created after the last database update.

---

### 3. Searching Files by Name

**Search for a filename:**

    locate app.js

**Search for paths containing a word:**

    locate nginx

**Search for paths ending in `.conf`:**

    locate "*.conf"

Quote wildcard patterns so the shell passes the pattern to `locate`.

**Perform a case-insensitive search:**

    locate -i README.md

**Limit the number of results:**

    locate -n 10 "*.log"

The `-n` option limits the number of matching results displayed.

**Search for an exact path:**

    locate -b '\app.js'

The `-b` option matches the basename, and the escaped pattern helps match the literal filename `app.js` rather than paths containing that text elsewhere. Exact behavior can vary with the locate implementation.

---

### 4. Updating the Locate Database

The locate database may not immediately reflect filesystem changes.

On systems using the common `mlocate` or `plocate` implementations, the `updatedb` utility updates the database.

**Update the database:**

    sudo updatedb

After updating, search again:

    locate app.js

The update command may take some time, depending on the number of files and filesystem size.

**Check whether `updatedb` is available:**

    command -v updatedb

If the command is unavailable, your distribution may use a different package, database service, or update mechanism.

**DevOps example:** If you create a new configuration file and `locate` cannot find it, an outdated database may be the reason. You can update the database or use `find` to search the filesystem directly.

---

### 5. Installing `locate` on Ubuntu

On Ubuntu, `plocate` is a common implementation of the locate utility.

Update package information:

    sudo apt update

Install `plocate`:

    sudo apt install plocate

Check the command:

    locate --version

If the command is available, try searching for a known path:

    locate /etc/passwd

If results are missing or outdated, check whether the database exists and whether its update mechanism is configured.

Depending on the Ubuntu version and package configuration, the database may be updated through a systemd timer or another scheduled mechanism.

---

### 6. Limitations of `locate`

Although `locate` is fast, it has some limitations:

- **Database freshness:** Newly created files may not appear until the database is updated.
- **Stale results:** Deleted or renamed files may still appear in the database.
- **Permission filtering:** Results can be restricted according to the implementation and database configuration.
- **Limited metadata filtering:** It is not designed for complex searches based on current file size, ownership, permissions, or modification time.
- **Implementation differences:** Options and database behavior can vary between `plocate`, `mlocate`, and other implementations.

Use `find` when you need current filesystem results or complex filtering. Use `locate` when you want to search an existing index quickly.

---

## ⚖️ `find` vs `locate`

---

## 🔎 Searching for Files with `which` and `whereis`

Both commands help locate files, but their search mechanisms and capabilities differ.

| Feature                     | `find`                                                             | `locate`                                               |
| --------------------------- | -------------------------------------------------------------------- | -------------------------------------------------------- |
| Search method               | Traverses specified directory trees                                  | Searches a pathname database                             |
| Speed                       | Depends on search scope and filesystem                               | Usually very fast                                        |
| Newly created files         | Can find them if accessible and within the search path               | May miss them until the database updates                 |
| Deleted files               | Does not return removed entries from a normal live filesystem search | May return stale paths                                   |
| Search by size              | Supported                                                            | Not generally supported using current file metadata      |
| Search by modification time | Supported                                                            | Not generally supported using current file metadata      |
| Search by permissions       | Supported                                                            | Not generally supported using current file metadata      |
| Run commands on results     | Supported with `-exec`                                             | Does not provide the same filesystem-search action model |
| Best use                    | Detailed, condition-based searches                                   | Fast searches by pathname                                |

**Example: Find all `.log` files larger than 100 MiB:**

Using `find`:

    find /var/log -type f -name "*.log" -size +100M

Using `locate`:

    locate "*.log"

The `locate` command can quickly list indexed paths matching the name pattern, but it cannot directly apply the same current-size condition.

**Rule of thumb:**

- Use `find` for detailed searches and administrative tasks.
- Use `locate` for quick searches by pathname.

---

## 🔎 Searching for Files with `which` and `whereis`

Linux also provides commands for finding executable programs and related files.

### 1. Using `which`

The `which` command displays the executable that would be selected for a command name by searching the directories in `PATH`.

**Example:**

    which python3

Possible output:

    /usr/bin/python3

**Find the location of Git:**

    which git

**Check multiple commands:**

    which python3 git docker

The output depends on the commands installed and the environment's `PATH`.

**Important:** `which` is commonly an external utility and may behave differently across shells and distributions. In Bash, `command -v` is often a more reliable way to check how a command name is resolved.

    command -v python3
    command -v git

This can report an executable path, alias, function, or shell builtin, depending on the command.

### 2. Using `whereis`

The `whereis` command searches standard locations for an executable, source files, and manual pages associated with a command.

**Example:**

    whereis python3

Possible output:

    python3: /usr/bin/python3 /usr/lib/python3 /usr/share/man/man1/python3.1.gz

Actual output depends on the system and installed packages.

**Find information about Git:**

    whereis git

**Search specifically for the executable:**

    whereis -b git

**Search for manual pages:**

    whereis -m git

**Search for source files:**

    whereis -s git

Not every program has source files or a manual page installed locally.

### 3. Differences Between `which`, `whereis`, `find`, and `locate`

| Command     | Main purpose                                  | Typical example              |
| ----------- | --------------------------------------------- | ---------------------------- |
| `which`   | Find an executable selected through `PATH`  | `which python3`            |
| `whereis` | Find executable, source, and manual locations | `whereis python3`          |
| `find`    | Search filesystem entries using conditions    | `find /etc -name "*.conf"` |
| `locate`  | Search an indexed database of pathnames       | `locate nginx.conf`        |

**Practical DevOps example:**

Suppose you need to troubleshoot a Docker command.

Check whether the command can be resolved:

    command -v docker

Look for its executable and manual locations:

    whereis docker

Search for configuration files with a known name:

    find /etc -name "*docker*"

Search the locate database for matching paths:

    locate docker

Each command answers a different question, so using them together can help you investigate a system more efficiently.

---

## ⚠️ Common File Search Mistakes

When searching for files in Linux, small mistakes can produce missing results, unexpected matches, or unnecessary system load. Understanding these issues helps you search more efficiently and safely.

### 1. Searching the Wrong Directory

The `find` command searches from the path you specify.

For example:

    find /home/user -name "app.js"

This searches under `/home/user`, not the entire filesystem.

**Better approach:** Confirm your starting directory and search path before running the command.

    pwd
    find . -name "app.js"

### 2. Forgetting Wildcard Quotes

Consider this command:

    find . -name *.txt

If the shell expands `*.txt` before `find` receives it, the search may fail or behave unexpectedly.

**Correct approach:**

    find . -name "*.txt"

Quoting the pattern allows `find` to interpret the wildcard.

### 3. Confusing `find` and `locate`

The `find` command searches directory trees directly, while `locate` normally searches a database of stored paths.

A recently created file may be found by `find` but missing from `locate` if the database has not been updated.

**Better approach:** Use `find` when you need current filesystem results or detailed conditions. Use `locate` for quick pathname searches.

### 4. Ignoring Permission Errors

Searching system directories may produce messages such as:

    Permission denied

Some files and directories are inaccessible to your current user.

You can suppress permission-related error messages when appropriate:

    find /var -name "*.log" 2>/dev/null

This redirects standard error to `/dev/null`. It hides error messages but does not grant additional permissions or guarantee that every directory was searched.

Use `sudo` only when elevated access is necessary.

### 5. Using the Wrong Search Pattern

These commands have different meanings:

    find . -name "config.txt"
    find . -name "*.txt"
    find . -iname "config.txt"

- The first searches for the exact name `config.txt`.
- The second searches for names ending in `.txt`.
- The third performs a case-insensitive name match.

**Better approach:** Decide whether you need an exact filename, a wildcard pattern, or case-insensitive matching.

### 6. Misunderstanding Time-Based Searches

The `find` options `-mtime` and `-mmin` measure elapsed time in different units.

For example:

    find . -type f -mtime -1

This finds files modified less than 24 hours ago, not necessarily files modified since midnight today.

For a shorter interval:

    find . -type f -mmin -30

This finds files modified less than 30 minutes ago.

### 7. Running Destructive Commands Without Checking Results

Commands that delete files based on search criteria can remove important data if the pattern or directory is incorrect.

For example, do not immediately run:

    find . -type f -name "*.tmp" -delete

First preview the matching files:

    find . -type f -name "*.tmp" -print

Only delete them after verifying that the results are correct.

### 8. Searching the Entire Filesystem Unnecessarily

A command such as:

    sudo find / -name "*.log" 2>/dev/null

may take considerable time because it searches across many directories and mounted filesystems.

**Better approach:** Narrow the search to a relevant location, such as `/var/log`, whenever possible.

    find /var/log -type f -name "*.log"

### 9. Assuming `which` Finds Every Command

The `which` command searches for executables using `PATH`, but shell aliases, functions, and built-in commands may not be represented as executable file paths.

Use:

    command -v cd
    command -v python3

This is often more suitable for checking how the current shell resolves a command.

---

## 🌐 Why File Searching Matters in DevOps

File searching is an essential Linux skill for DevOps engineers because applications, logs, configuration files, scripts, and deployment artifacts are distributed across many directories.

### 1. Troubleshooting Application Issues

When an application fails, you may need to locate its logs or configuration files.

Example: Search for Nginx configuration files:

    find /etc/nginx -type f -name "*.conf"

This helps identify configuration files that may need inspection during troubleshooting.

### 2. Monitoring Disk Usage

Large log files, temporary files, and build artifacts can consume disk space.

Example: Find files larger than 100 MiB under `/var/log`:

    sudo find /var/log -type f -size +100M 2>/dev/null

Review the results before archiving or removing any files.

### 3. Investigating Recent Changes

When a deployment introduces an issue, recently modified files may provide useful clues.

Example: Find files modified within the last 24 hours in an application directory:

    find /opt/myapp -type f -mtime -1

This helps narrow down the files that may have changed recently, although it does not prove that those files caused the issue.

### 4. Managing Permissions and Security

Linux administrators sometimes need to identify files with unexpected permissions or ownership.

Example: Find files with any execute permission bit set:

    find /opt/myapp -type f -perm /111

Example: Find files writable by everyone:

    find /opt/myapp -type f -perm -002

These searches can support security reviews. Results should be evaluated against the application's requirements before making changes.

### 5. Automating Maintenance Tasks

The `find` command can be incorporated into shell scripts and maintenance jobs.

For example, list temporary files before cleanup:

    find /tmp -type f -name "*.tmp" -print

A scheduled maintenance script could use similar searches to identify old temporary files, collect reports, or locate artifacts. Any automated deletion should have carefully tested conditions and an appropriate retention policy.

### 6. Working with CI/CD Pipelines

Build and deployment pipelines often need to locate test reports, configuration files, generated artifacts, or application files.

Example: Find JSON files in a project:

    find . -type f -name "*.json"

Example: Find test files with common JavaScript test suffixes:

    find . -type f \( -name "*.test.js" -o -name "*.spec.js" \)

These commands can help with debugging build steps and verifying the expected project structure.

### 7. Choosing Efficient Search Tools

A DevOps engineer should know when to use each command:

- `find` — detailed searches by name, size, time, type, permissions, or ownership.
- `locate` — quick searches using an indexed database of paths.
- `which` — identify the executable selected through `PATH`.
- `whereis` — find executable, source, and manual-page locations.

**Key takeaway:** Effective file searching reduces troubleshooting time, supports system maintenance, and makes Linux administration and DevOps automation more reliable.

---


## 💻 Hands-on Practice

In this practical exercise, you will create sample files, search by different conditions, use `locate`, and identify executable commands. You will also practise safe file cleanup.

Use Ubuntu or WSL and follow the steps in order.

### 1. Create Practice Files and Directories

Create a dedicated practice directory:

    mkdir -p ~/linux-practice/file-search/{logs,configs,scripts,backups}
    cd ~/linux-practice/file-search

Create sample files:

    touch notes.txt README.md app.js server.py
    touch configs/app.conf configs/database.conf
    touch logs/app.log logs/error.log
    touch scripts/deploy.sh backups/old-backup.tmp

Add sample content:

    echo "Linux file search practice" > notes.txt
    echo "Application started" > logs/app.log
    echo "Database connection failed" > logs/error.log
    echo "PORT=3000" > configs/app.conf

Create a larger file for size-based searching:

    dd if=/dev/zero of=large-file.bin bs=1M count=5

Verify the directory structure:

    find . -type f

You now have files with different names, extensions, and sizes to practise searching.

### 2. Find Files by Name and Extension

Find a specific file:

    find . -name "notes.txt"

Find all text files:

    find . -type f -name "*.txt"

Find all JavaScript files:

    find . -type f -name "*.js"

Find all configuration files:

    find . -type f -name "*.conf"

Perform a case-insensitive search:

    find . -iname "readme.md"

**Challenge:** Find every file whose name ends in `.log` without searching outside the practice directory.

### 3. Find Files by Type and Size

Find all regular files:

    find . -type f

Find all directories:

    find . -type d

Find files larger than 1 MiB:

    find . -type f -size +1M

Find empty files:

    find . -type f -empty

Find symbolic links:

    find . -type l

There may be no symbolic links yet. That is expected because you have not created any.

**Challenge:** Find files larger than 1 MiB only inside the `logs` directory.

### 4. Find Recently Modified Files

Find files modified within the last 24 hours:

    find . -type f -mtime -1

Find files modified within the last 30 minutes:

    find . -type f -mmin -30

Modify a file:

    echo "Updated for testing" >> notes.txt

Search again:

    find . -type f -mmin -30

Observe how modifying a file changes its modification time.

**Note:** These commands use elapsed-time windows, not calendar-day boundaries. Other activity on your system may also affect the results.

### 5. Search for Files Using `locate`

Check whether `locate` is installed:

    command -v locate

If it is unavailable on Ubuntu, install `plocate`:

    sudo apt update
    sudo apt install plocate

Update the database if the `updatedb` command is available:

    sudo updatedb

Search for a known filename:

    locate notes.txt

Search for configuration files:

    locate "*.conf"

Compare the results with:

    find ~/linux-practice/file-search -name "*.conf"

**Observation:** `locate` searches its database, while `find` searches the specified directory tree. Depending on database configuration and permissions, `locate` may return stale paths or omit some paths.

### 6. Locate Executable Commands with `which` and `whereis`

Check the location of common commands:

    which bash
    which python3
    which ls

Check executable and manual-page locations:

    whereis bash
    whereis python3

Use `command -v` to see how your shell resolves a command:

    command -v cd
    command -v python3

Observe that `cd` is usually a shell builtin, so it does not need a standalone executable file.

**Challenge:** Compare the output of `which python3` and `whereis python3`. What additional information does `whereis` provide on your system?

### 7. Practice Safe Search and Cleanup

Find temporary files without deleting them:

    find . -type f -name "*.tmp" -print

Review the output. If it contains only the files you intend to remove, delete them:

    find . -type f -name "*.tmp" -delete

Verify that the temporary files are gone:

    find . -type f -name "*.tmp"

Remove the practice directory after completing the exercises, if you no longer need it:

    cd ~
    rm -r ~/linux-practice/file-search

**Safety reminder:** Always preview search results before using `-delete` or recursive removal commands. Double-check the target path before executing cleanup commands.

### 🧪 Mini Lab

Complete these tasks without copying the earlier commands directly:

1. Find all `.js` and `.py` files under your practice directory.
2. Find every directory named `configs`.
3. Identify files larger than 2 MiB.
4. Find files modified within the last hour.
5. Find files with `.conf` in their names using `locate`.
6. Identify the executable path for `bash` and `python3`.
7. Explain why a newly created file may appear in `find` results but not in `locate` results.
8. Explain why `command -v cd` can return useful information even though `cd` is not normally an executable file.

**Beginner tip:** Practise one condition at a time, then combine conditions. Being able to explain why a command returns a particular result is more valuable than memorizing its syntax.

---

## 🎯 Interview Questions

1. What is the purpose of the `find` command in Linux?
2. What is the basic syntax of the `find` command?
3. What is the difference between `-name` and `-iname`?
4. How do you find only regular files?
5. How do you find directories with a specific name?
6. How can you find files larger than 100 MiB?
7. What is the difference between `-mtime` and `-mmin`?
8. How can you find files with specific permissions?
9. What does the `-exec` option do in `find`?
10. What is the difference between `find` and `locate`?
11. Why might `locate` return a path to a file that no longer exists?
12. How can you update the locate database on a system that supports `updatedb`?
13. What is the purpose of the `which` command?
14. How does `whereis` differ from `which`?
15. Why is `command -v` useful when checking shell commands?
16. How can you search for files modified within the last 30 minutes?
17. Why should you preview results before using `find -delete`?
18. How can file-search commands help troubleshoot a Linux server?
19. How can you search for large log files that consume disk space?
20. Which command would you choose to find a configuration file quickly, and why?

---

## 📝 Summary

- `find` searches directory trees using conditions such as name, type, size, time, permissions, and ownership.
- `-name` matches filename patterns, while `-iname` performs case-insensitive matching.
- `-type f`, `-type d`, and `-type l` select regular files, directories, and symbolic links.
- `-size`, `-mtime`, and `-mmin` help locate files based on size and modification time.
- `-perm` and `-user` allow searches based on permissions and ownership.
- `-exec` runs commands on matching results.
- `locate` searches a database of pathnames and is generally faster for simple path searches.
- `which` helps identify executables resolved through `PATH`; `whereis` searches for executables and related files.
- `command -v` is useful for checking how the current shell resolves a command.
- Preview search results before running destructive operations.

File-searching skills are essential for Linux administration, troubleshooting, security checks, disk management, and DevOps automation.

---

## 📚 References

- [GNU Findutils — Finding Files](https://www.gnu.org/software/findutils/manual/html_mono/find.html)
- [GNU Findutils — `find` Reference](https://www.gnu.org/software/findutils/manual/html_node/find-Expressions.html)
- [Ubuntu Manpage — `plocate`](https://manpages.ubuntu.com/manpages/noble/en/man1/plocate.1.html)
- [Linux man page — `updatedb`](https://man7.org/linux/man-pages/man8/updatedb.8.html)
- [Linux man page — `which`](https://man7.org/linux/man-pages/man1/which.1.html)
- [Linux man page — `whereis`](https://man7.org/linux/man-pages/man1/whereis.1.html)
- [Bash Reference Manual — Bash Builtins](https://www.gnu.org/software/bash/manual/html_node/Bash-Builtins.html)

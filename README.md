 Managing Flatpak

**RHCSA EX200 Lab** | Sander van Vugt Sample Exam  
**Server:** server2 (EC2 AMI)  
**User:** conadm  
**Time Estimate:** 15–20 minutes

---

## 🎯 Objective

Install Flatpak, add the Flathub repository for a single user only, and install GIMP from Flathub — accessible to that user only, not system-wide.

---

## 🧠 Big Idea — What is Flatpak?

**Flatpak** is a universal package manager for Linux that:

- Runs apps in sandboxed containers
- Works across distros — not tied to `dnf` or `apt`
- Supports **system-wide** installs (all users) or **user-level** installs (one user only)

| Install Type | Flag | Who can use it |
|---|---|---|
| System-wide | no flag / `--system` | All users |
| User-only | `--user` | Only the user who installed it |

> This lab uses **user-level** install — Flathub repo and GIMP are only accessible to `conadm`.

---

## 📚 Command Decision Map

| Task | Command |
|---|---|
| Install flatpak | `sudo dnf install flatpak -y` |
| Switch to target user | `su - <username>` |
| Add repo user-only | `flatpak remote-add --user --if-not-exists flathub <url>` |
| Verify remotes | `flatpak remotes --user` |
| Install app user-only | `flatpak install --user flathub <app-id>` |
| List user-installed apps | `flatpak list --user` |
| List system-wide apps | `flatpak list` |

---

## 🔧 Steps

### Step 1 — Install Flatpak on the system

```bash
sudo dnf install flatpak -y
```

**Command explained:**

| Part | Meaning |
|---|---|
| `sudo` | Run as root — system-wide install requires elevated privileges |
| `dnf` | Red Hat's package manager (replaces `yum` on RHEL 8+) |
| `install flatpak` | Install the Flatpak runtime package |
| `-y` | Automatically answer "yes" to all prompts — no manual confirmation needed |

**Expected output (last lines):**
```
Installed:
  flatpak-x.x.x...
Complete!
```

---

### Step 2 — Switch to target user

```bash
su - conadm
```

**Command explained:**

| Part | Meaning |
|---|---|
| `su` | Switch User — change to another user account |
| `-` | Load the full login environment: home directory, PATH, shell config files (`.bashrc`, `.bash_profile`) |
| `conadm` | The user to switch to |

> ⚠️ Always use `su -` with the dash. Without it, you keep your current user's environment which can cause unexpected behavior.

---

### Step 3 — Add Flathub repository for this user only

```bash
flatpak remote-add --user --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```

**Command explained:**

| Part | Meaning |
|---|---|
| `flatpak` | The Flatpak package manager |
| `remote-add` | Subcommand to add a new remote repository source |
| `--user` | Add this repo for the current user only — NOT system-wide |
| `--if-not-exists` | If this remote is already configured, skip silently instead of throwing an error |
| `flathub` | The name you are giving this remote (used later when installing apps) |
| `https://dl.flathub.org/repo/flathub.flatpakrepo` | The URL of Flathub's repo config file — contains keys, URLs, and metadata |

---

### Step 4 — Verify the remote was added

```bash
flatpak remotes --user
```

**Command explained:**

| Part | Meaning |
|---|---|
| `flatpak` | The Flatpak package manager |
| `remotes` | List all configured remote repositories |
| `--user` | Show only user-level remotes, not system-wide ones |

**Expected output:**
```
Name    Options
flathub user,system
```

---

### Step 5 — Install GIMP from Flathub for this user only

```bash
flatpak install --user flathub org.gimp.GIMP
```

**Command explained:**

| Part | Meaning |
|---|---|
| `flatpak` | The Flatpak package manager |
| `install` | Subcommand to install an application |
| `--user` | Install for the current user only — not system-wide |
| `flathub` | Which remote repository to install from (named in Step 3) |
| `org.gimp.GIMP` | The Flatpak application ID for GIMP — uses reverse domain format to ensure uniqueness across all apps |

> Flatpak app IDs always use reverse domain format: `org.gimp.GIMP`, `com.spotify.Client`, `org.mozilla.firefox`

---

### Step 6 — Verify GIMP is installed for this user

```bash
flatpak list --user
```

**Command explained:**

| Part | Meaning |
|---|---|
| `flatpak` | The Flatpak package manager |
| `list` | Show all installed Flatpak applications |
| `--user` | Show only apps installed at the user level |

**Actual output:**
```
Name                 Application ID             Version Branch
Freedesktop SDK      …sktop.Platform.GL.default 26.0.5  25.08
Freedesktop SDK      …sktop.Platform.GL.default 26.0.5  25.08-extra
Freedesktop SDK      …top.Platform.codecs-extra         25.08-extra
The GIMP team        org.gimp.GIMP              3.2.4   stable
GNOME Application P… org.gnome.Platform                 50
```

**Output explained:**

| Entry | Meaning |
|---|---|
| `Freedesktop SDK` entries | Runtime dependencies GIMP needs to run — installed automatically |
| `org.gimp.GIMP 3.2.4` | GIMP version 3.2.4 installed on the `stable` branch ✅ |
| `org.gnome.Platform` | GNOME platform runtime — another dependency pulled in automatically |
| `Branch: stable` | Installed from the stable release channel |

---

### Step 7 — Exit back to original user

```bash
exit
```

**Output:**
```
logout
[ec2-user@ip-172-31-10-183 /]$
```

---

### Step 8 — Verify GIMP is NOT available system-wide

```bash
flatpak list
```

**Actual output:**
```
error: While opening repository /var/lib/flatpak/repo: opening repo: opendir(/var/lib/flatpak/repo): No such file or directory
```

**Output explained:**

| Detail | Meaning |
|---|---|
| `/var/lib/flatpak/repo` | Where system-wide Flatpak apps are stored |
| `No such file or directory` | This directory doesn't exist — no system-wide Flatpak repo was ever initialized ✅ |
| This error = success | Confirms GIMP was installed user-only and is completely absent from the system level |

> This error is the **expected and correct** result. It proves the user-only install worked exactly as required.

---

## ✅ Lab Checklist

- [x] `flatpak` installed via `dnf`
- [x] Switched to `conadm` with `su -`
- [x] Flathub remote added with `--user` flag
- [x] `flatpak remotes --user` shows flathub
- [x] GIMP installed with `--user` flag
- [x] `flatpak list --user` shows `org.gimp.GIMP 3.2.4`
- [x] System-wide `flatpak list` confirms no system-wide install

---

## ⚠️ Common Pitfalls

| Mistake | Fix |
|---|---|
| Added Flathub without `--user` | Remove system-wide: `sudo flatpak remote-delete flathub` |
| Installed GIMP without `--user` | Remove system-wide: `sudo flatpak uninstall org.gimp.GIMP` |
| `su conadm` without `-` | Wrong environment loaded — always use `su - username` |
| `flatpak --list` | Wrong syntax — correct is `flatpak list` |
| GIMP install times out | Check internet connectivity from EC2 |

---

## 📌 Exam Tips

- The exam will specify **user-only** vs **system-wide** — read carefully.
- `--user` flag is required on BOTH `remote-add` AND `install` for user-only access.
- Flatpak app IDs use reverse domain format: `org.gimp.GIMP`, `com.spotify.Client`.
- A missing `/var/lib/flatpak/repo` error on `flatpak list` means no system-wide repo exists — confirms user-only install succeeded.
- Always verify with `flatpak list --user` after installing.

---

## 🔗 Part of Linux Ops Mastery — Package Management Section

- [Linux Ops Mastery](https://github.com/kelvintechnical/linux-ops-mastery)

---

## 👤 Author

**Kelvin R. Tobias**  
[kelvinintech.com](https://kelvinintech.com) · [GitHub](https://github.com/kelvintechnical) · [LinkedIn](https://www.linkedin.com/in/kelvin-r-tobias-211949219)

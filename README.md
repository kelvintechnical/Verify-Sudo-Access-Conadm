# Lab 22-1c: Verify Sudo Access — `conadm`

**RHCSA EX200 Lab** | Series: Container Management (Lab 22)  
**Prerequisite:** Lab 22-1b complete (`conadm` added to `wheel` group)  
**Time Estimate:** 5 minutes

---

## 🎯 Objective

Confirm that `conadm` can execute privileged commands using `sudo` before proceeding with container operations.

---

## 📚 Command Decision Map

| Task | Command |
|---|---|
| Switch to conadm | `su - conadm` |
| Test sudo elevation | `sudo whoami` |
| Test sudo with real command | `sudo ls /root` |
| Check sudo permissions | `sudo -l` |
| Exit back to original user | `exit` |

---

## 🔧 Steps

### Step 1 — Switch to `conadm`

```bash
su - conadm
```

**Expected output:**

```
Password:
Last login: Wed May 20 15:57:42 EDT 2026 on pts/1
[conadm@ip-172-31-10-183 ~]$
```

**Output explained:**

| Line | Meaning |
|---|---|
| `Password:` | Linux is asking for `conadm`'s password (set in Lab 22-1a) |
| `Last login: ...` | Shows the last time this user logged in — confirms account is active |
| `[conadm@ip-172-31-10-183 ~]$` | You are now operating as `conadm`; `~` means you are in their home directory `/home/conadm` |

---

### Step 2 — Verify sudo elevation

```bash
sudo whoami
```

**Expected output:**

```
root
```

**Output explained:**

| Output | Meaning |
|---|---|
| `root` | The command ran as root — sudo elevation is working correctly |

> If you see `conadm` instead of `root`, sudo is not working — recheck Lab 22-1b.

---

### Step 3 — Test sudo with a real privileged command

```bash
sudo ls /root
```

**Expected output:**

```
config_files  httpd_listen.txt  Mail
```

**Output explained:**

| Detail | Meaning |
|---|---|
| Contents are listed | `conadm` successfully accessed `/root`, which is only readable by root |
| Empty output is also valid | `/root` may have no files — that's fine; no error = sudo works |
| `Permission denied` | sudo is NOT working — stop and recheck Lab 22-1b |

---

### Step 4 — List all sudo permissions for `conadm`

```bash
sudo -l
```

**Expected output:**

```
Matching Defaults entries for conadm on ip-172-31-10-183:
    !visiblepw, always_set_home, match_group_by_gid,
    always_query_group_plugin, env_reset, env_keep="COLORS DISPLAY
    HOSTNAME HISTSIZE KDEDIR LS_COLORS", env_keep+="MAIL PS1 PS2
    QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE", env_keep+="LC_COLLATE
    LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES",
    env_keep+="LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER
    LC_TELEPHONE", env_keep+="LC_TIME LC_ALL LANGUAGE LINGUAS
    _XKB_CHARSET XAUTHORITY",
    secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin

User conadm may run the following commands on ip-172-31-10-183:
    (ALL) ALL
```

**Output explained:**

| Section | Meaning |
|---|---|
| `Matching Defaults entries for conadm` | These are the sudo environment rules that apply to `conadm` |
| `!visiblepw` | Password will not be shown on screen when typing |
| `always_set_home` | sudo always sets `HOME` to root's home directory |
| `env_reset` | sudo resets environment variables for security |
| `env_keep="..."` | These specific variables are allowed to carry over into the sudo session |
| `secure_path=...` | When running sudo, only these directories are searched for commands |
| `(ALL) ALL` | **The key line** — `conadm` can run ALL commands as ANY user on this machine |

---

### Step 5 — Exit back to original user

```bash
exit
```

**Expected output:**

```
logout
[ec2-user@ip-172-31-10-183 /]$
```

**Output explained:**

| Line | Meaning |
|---|---|
| `logout` | The `conadm` shell session has ended |
| `[ec2-user@...]$` | You are back to your original user — confirmed by the username in the prompt |

---

## ✅ Lab Checklist

- [ ] `su - conadm` switches successfully
- [ ] `sudo whoami` returns `root`
- [ ] `sudo ls /root` executes without permission error
- [ ] `sudo -l` shows `(ALL) ALL`
- [ ] `exit` returns to original user

---

## ⚠️ Common Pitfalls

| Mistake | Fix |
|---|---|
| `sudo whoami` returns `conadm` | sudo not working — recheck Lab 22-1b |
| `sudo -l` shows no rules | `wheel` group not active — log out and back in |
| Permission denied on `/root` | sudo not elevating properly — recheck `/etc/sudoers` |
| Still showing `conadm` prompt after `exit` | Run `exit` again — may have nested shells |

---

## 📌 Exam Tips

- `sudo -l` is your best friend on the exam — always use it to confirm what a user can run.
- `su - conadm` (with the `-`) loads the full login environment — always use `-` when switching users.
- `su conadm` (without `-`) inherits your current environment — can cause unexpected behavior.

---
## ➡️ Next Lab
**[Lab 22-1d: Inspect ubi9 with skopeo](https://github.com/kelvintechnical/Inspect-ubi9-with-skopeo)**

---

## 🔗 Series Index
- ✅ [Lab 22-1a: Create conadm user](https://github.com/kelvintechnical/Create-User-Account-Conadm)
- ✅ [Lab 22-1b: Grant sudo rights](https://github.com/kelvintechnical/Grant-Conadm-Full-Rights)
- 👉 **Lab 22-1c: Verify sudo access** ← you are here
- [Lab 22-1d: Inspect ubi9 with skopeo](https://github.com/kelvintechnical/Inspect-ubi9-with-skopeo)
- [Lab 22-1e: Pull ubi9 image](https://github.com/kelvintechnical/Pull-ubi9-Image-with-podman)
- [Lab 22-1f: Launch container with port mapping](https://github.com/kelvintechnical/launch-container-interative-terminal)
- [Lab 22-1g: Run commands inside container](https://github.com/kelvintechnical/run-commands-inside-terminal)
- [Lab 22-1h: Verify port mapping from host](./lab-22-1h-verify-port.md)

---

## 👤 Author

**Kelvin R. Tobias**  
[kelvinintech.com](https://kelvinintech.com) · [GitHub](https://github.com/kelvintechnical) · [LinkedIn](https://www.linkedin.com/in/kelvin-r-tobias-211949219)

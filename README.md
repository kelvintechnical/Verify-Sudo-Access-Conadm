# Lab 22-1c: Verify Sudo Access — `conadm`

**RHCSA EX200 Lab** | Series: Container Management (Lab 22)  
**Prerequisite:** Lab 22-1b complete (`conadm` added to `wheel` group)  
**Time Estimate:** ~5 minutes

---

## 🎯 Objective

Confirm that `conadm` can execute privileged commands using `sudo` before proceeding with container operations. Verification is not optional — always confirm access works before building on top of it.

---

## 📚 Command Decision Map

| Task | Command |
|---|---|
| Switch to `conadm` | `su - conadm` |
| Test sudo elevation | `sudo whoami` |
| Test sudo with real privileged command | `sudo ls /root` |
| List all sudo permissions | `sudo -l` |
| Audit user's sudo rules from outside | `sudo -lU conadm` |
| Exit back to original user | `exit` |

---

## 🧠 Concept: Why Verify at All?

Adding a user to `wheel` doesn't guarantee sudo works. Several things can silently prevent it:

| Failure point | What goes wrong |
|---|---|
| Group change in same session | New group memberships only apply after a new login |
| `wheel` line commented in sudoers | The `/etc/sudoers` rule may be disabled |
| Wrong shell assigned | Login environment not loaded correctly |
| Nested shell confusion | You may still be operating under a parent session |

This lab exists to **catch those failures before they surface during container operations**.

---

## 🔧 Steps

### Step 1 — Switch to `conadm`

```bash
su - conadm
```

**Actual output:**

```
Password:
[conadm@ip-172-31-0-157 ~]$
```

#### Output explained

| Part | Meaning |
|---|---|
| `Password:` | Linux prompts for `conadm`'s own password (set in Lab 22-1a). Input is hidden. |
| `[conadm@ip-172-31-0-157 ~]$` | Your prompt changed — you are now operating as `conadm`. The `~` confirms you landed in `/home/conadm`. |

#### `su -` vs `su` — why the dash matters

| Command | Behavior |
|---|---|
| `su - conadm` | Starts a **login shell** — loads `.bash_profile`, resets `$PATH`, `$HOME`, and environment to `conadm`'s. ✅ Recommended. |
| `su conadm` | Starts a **non-login shell** — inherits your current environment. Group changes may not be active. ❌ Can cause unexpected behavior. |

> **Always use `su -`** when switching users on the exam. The login shell ensures the new session is fully initialized, including freshly loaded group memberships.

**Alternative if you don't know `conadm`'s password:**

```bash
sudo -u conadm -i
```

| Flag | Meaning |
|---|---|
| `-u conadm` | Run the shell as `conadm` |
| `-i` | Simulate a login shell (equivalent to `su -`) |

This lets `ec2-user` (who has sudo) open a login shell as `conadm` without needing `conadm`'s password.

---

### Step 2 — Test sudo elevation

```bash
sudo whoami
```

**Actual output:**

```
We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

For security reasons, the password you type will not be visible.

[sudo] password for conadm:
root
```

#### Output explained

| Section | Meaning |
|---|---|
| The lecture block | A one-time educational message displayed the **first time** a user invokes `sudo` on this system. It will not appear again in subsequent `sudo` calls during this session. |
| `[sudo] password for conadm:` | sudo asks for `conadm`'s **own** password — not root's. This is correct and expected behavior. |
| `root` | `whoami` printed the effective user identity after elevation. **This is the confirmation you need.** |

> **If you see `conadm` instead of `root`:** sudo is not working. Stop here and recheck Lab 22-1b — verify `wheel` membership with `id conadm` and confirm `/etc/sudoers` has `%wheel ALL=(ALL) ALL` uncommented.

---

### Step 3 — Test sudo with a real privileged command

```bash
sudo ls /root
```

**Actual output:**

```
[conadm@ip-172-31-0-157 ~]$
```

*(Empty output — no files in `/root` on this instance)*

#### Output explained

| Result | Meaning |
|---|---|
| Any directory listing | `conadm` accessed `/root` — a directory only readable by root. Sudo elevation confirmed. ✅ |
| Empty output (no files) | Also valid — `/root` may be empty on a fresh instance. No error = success. ✅ |
| `Permission denied` | sudo is **not** elevating correctly. Stop and recheck Lab 22-1b. ❌ |

> **Why `/root`?** The root user's home directory is protected with `700` permissions (`drwx------`). Only root can read it. Successfully listing it proves your command ran with root-level privileges — not just that sudo accepted the password.

---

### Step 4 — List all sudo rules for `conadm`

```bash
sudo -l
```

**Actual output:**

```
Matching Defaults entries for conadm on ip-172-31-0-157:
    !visiblepw, always_set_home, match_group_by_gid,
    always_query_group_plugin, env_reset, env_keep="COLORS DISPLAY HOSTNAME
    HISTSIZE KDEDIR LS_COLORS", env_keep+="MAIL PS1 PS2 QTDIR USERNAME LANG
    LC_ADDRESS LC_CTYPE", env_keep+="LC_COLLATE LC_IDENTIFICATION
    LC_MEASUREMENT LC_MESSAGES", env_keep+="LC_MONETARY LC_NAME LC_NUMERIC
    LC_PAPER LC_TELEPHONE", env_keep+="LC_TIME LC_ALL LANGUAGE LINGUAS
    _XKB_CHARSET XAUTHORITY", secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin

User conadm may run the following commands on ip-172-31-0-157:
    (ALL) ALL
```

#### Output explained — Defaults section

| Entry | Meaning |
|---|---|
| `!visiblepw` | Password characters are **not** echoed to the terminal while typing. |
| `always_set_home` | sudo always sets `$HOME` to root's home (`/root`), regardless of the calling user. |
| `match_group_by_gid` | Groups are matched by GID number, not just name — prevents group spoofing. |
| `env_reset` | sudo **clears** the environment before running the command — security measure to prevent environment variable injection. |
| `env_keep="..."` | These specific variables are **allowed to pass through** into the sudo session (locale, display settings, etc.). |
| `secure_path=...` | When running a command via sudo, only these directories are searched for binaries: `/sbin`, `/bin`, `/usr/sbin`, `/usr/bin`. Your custom `$PATH` is ignored for security. |

#### Output explained — Permission line

```
User conadm may run the following commands on ip-172-31-0-157:
    (ALL) ALL
```

| Field | Meaning |
|---|---|
| `(ALL)` | Can run commands **as any user** (including root) |
| `ALL` | Can run **any command** — no restrictions |
| No `NOPASSWD` | Password will be required each session — correct and secure behavior |

> **`sudo -l` is your most important diagnostic tool.** On the exam, if you're unsure whether a user has sudo access or what they're allowed to run, this is the first command to reach for. It shows exactly what the system will permit — no guessing.

---

### Step 5 — Audit sudo rules from outside the account

From `ec2-user` (or any sudo-capable user), you can inspect another user's permissions without switching to them:

```bash
sudo -lU conadm
```

**Actual output:**

```
Matching Defaults entries for conadm on ip-172-31-0-157:
    ...
User conadm may run the following commands on ip-172-31-0-157:
    (ALL) ALL
```

#### Why this matters

| Scenario | Use |
|---|---|
| Auditing multiple accounts quickly | No need to `su -` into each one |
| Verifying after `usermod` without switching sessions | Confirm the change took effect |
| Exam troubleshooting | Faster than switching users back and forth |

> `-lU <username>` is a sysadmin auditing shortcut. It reads the same sudoers rules as `sudo -l` but targets any specified user from your current session.

---

### Step 6 — Exit back to original user

```bash
exit
```

**Actual output:**

```
logout
[conadm@ip-172-31-0-157 ~]$
```

> **Notice:** The prompt still shows `conadm`. This is because `sudo -u conadm -i` was run inside the `conadm` session, creating a **nested shell**. You exited one layer — but you're still inside `conadm`'s session. Run `exit` again to return fully to `ec2-user`.

#### Shell nesting explained

```
ec2-user session
  └── su - conadm          ← Layer 1
        └── sudo -u conadm -i   ← Layer 2 (nested)
```

Each `exit` peels back one layer. When your prompt shows `[ec2-user@...]$`, you're fully out.

> **Exam habit:** Always confirm the username in your prompt after `exit`. Leaving an elevated or wrong-user session open mid-exam is a common source of errors.

---

## ✅ Lab Checklist

- [ ] `su - conadm` switches session and shows `conadm` in prompt
- [ ] `sudo whoami` returns `root`
- [ ] `sudo ls /root` executes without `Permission denied`
- [ ] `sudo -l` shows `(ALL) ALL` in the permissions section
- [ ] `sudo -lU conadm` shows same rules from outside the account
- [ ] `exit` (twice if needed) returns prompt to `ec2-user`

---

## ⚠️ Common Pitfalls

| Mistake | Symptom | Fix |
|---|---|---|
| `su conadm` without `-` | Group changes may not be active | Always use `su - conadm` |
| `sudo whoami` returns `conadm` | sudo not elevating | Recheck `id conadm` — confirm `wheel` is in groups |
| `sudo -l` shows no rules | `wheel` group not yet active for session | Log out fully and back in as `conadm` |
| `Permission denied` on `/root` | sudo not working | Recheck `%wheel ALL=(ALL) ALL` in `/etc/sudoers` |
| Prompt still shows `conadm` after `exit` | Nested shells | Run `exit` again until `ec2-user` prompt appears |

---

## 📌 RHCSA Exam Strategy

- **`sudo -l`** is the fastest way to confirm what a user can run — use it before and after any permission change.
- **`su -`** (with the dash) is always correct when switching users — loads the full login environment.
- **`sudo -lU <user>`** lets you audit any account from your own session — no switching required.
- An **empty `sudo ls /root`** is a passing result — no error message means success.
- **Nested shells** are easy to miss — always check your prompt username before running the next command.

---

## ➡️ Next Lab

**[Lab 22-1d: Inspect ubi9 with skopeo](https://github.com/kelvintechnical/Inspect-ubi9-with-skopeo)**

---

## 🔗 Series Index

| Lab | Topic |
|---|---|
| ✅ [22-1a](https://github.com/kelvintechnical/Create-User-Account-Conadm) | Create user `conadm` |
| ✅ [22-1b](https://github.com/kelvintechnical/Grant-Conadm-Full-Rights) | Grant `conadm` full sudo rights |
| 👉 **22-1c** | Verify sudo access ← *you are here* |
| [22-1d](https://github.com/kelvintechnical/Inspect-ubi9-with-skopeo) | Inspect ubi9 image with skopeo |
| [22-1e](https://github.com/kelvintechnical/Pull-ubi9-Image-with-podman) | Pull ubi9 image with podman |
| [22-1f](https://github.com/kelvintechnical/launch-container-interative-terminal) | Launch container with port mapping |
| [22-1g](https://github.com/kelvintechnical/run-commands-inside-terminal) | Run commands inside container |
| [22-1h](https://github.com/kelvintechnical/verify-port-mapping-from-host) | Verify port mapping from host |

---

## 👤 Author

**Kelvin R. Tobias**  
[kelvinintech.com](https://kelvinintech.com) · [GitHub](https://github.com/kelvintechnical) · [LinkedIn](https://www.linkedin.com/in/kelvin-r-tobias-211949219)

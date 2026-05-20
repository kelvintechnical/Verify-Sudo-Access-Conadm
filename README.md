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

**Expected prompt:**

```
[conadm@server30 ~]$
```

---

### Step 2 — Verify sudo elevation

```bash
sudo whoami
```

**Expected output:**

```
root
```

---

### Step 3 — Test sudo with a real privileged command

```bash
sudo ls /root
```

> `/root` is only accessible by root. If this returns contents (or empty), sudo is working. ✅

---

### Step 4 — List all sudo permissions for `conadm`

```bash
sudo -l
```

**Expected output (key line):**

```
(ALL) ALL
```

> This confirms `conadm` can run **all commands** as **any user**. ✅

---

### Step 5 — Exit back to original user

```bash
exit
```

**Expected prompt:**

```
[ec2-user@server30 /]$
```

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

**[Lab 22-1d: Inspect ubi9 with skopeo](./lab-22-1d-skopeo-inspect.md)**

---

## 🔗 Series Index

- ✅ [Lab 22-1a: Create conadm user](./lab-22-1a-create-conadm.md)
- ✅ [Lab 22-1b: Grant sudo rights](./lab-22-1b-sudo-conadm.md)
- 👉 **Lab 22-1c: Verify sudo access** ← you are here
- [Lab 22-1d: Inspect ubi9 with skopeo](./lab-22-1d-skopeo-inspect.md)
- [Lab 22-1e: Pull ubi9 image](./lab-22-1e-podman-pull.md)
- [Lab 22-1f: Launch container with port mapping](./lab-22-1f-launch-container.md)
- [Lab 22-1g: Run commands inside container](./lab-22-1g-container-commands.md)
- [Lab 22-1h: Verify port mapping from host](./lab-22-1h-verify-port.md)

---

## 👤 Author

**Kelvin R. Tobias**  
[kelvinintech.com](https://kelvinintech.com) · [GitHub](https://github.com/kelvintechnical) · [LinkedIn](https://www.linkedin.com/in/kelvin-r-tobias-211949219)

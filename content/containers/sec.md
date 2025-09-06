

## AppArmor with Docker and Kubernetes

---

## What is AppArmor?

- **AppArmor** is a Linux security module that restricts programs’ capabilities with per-program profiles.
- It controls what files, network, capabilities, and system calls a process (like a container) can access.
- Helps reduce the attack surface by limiting what containers can do.

---

## Using AppArmor with Docker

### Step 1: Check if AppArmor is enabled

```bash
sudo aa-status
```

# AppArmor on Amazon Linux?

## Why `aa-status` is not found?

- Amazon Linux **does not come with AppArmor** installed or enabled by default.
- It primarily uses **SELinux** (Security-Enhanced Linux) as its Linux security module.
- AppArmor is mainly found on Ubuntu, Debian, and SUSE-based distributions.
- Red Hat-based distros (including Amazon Linux) use SELinux.

---

## What does this mean?

- You **cannot use AppArmor commands** like `aa-status` on Amazon Linux.
- To use AppArmor, you would need a host running Ubuntu or Debian.
- On Amazon Linux, **SELinux** is the default Mandatory Access Control system.

---

## What should you do on Amazon Linux?

- Explore and use **SELinux** for security policies.
- Use SELinux with Docker and Kubernetes to enforce container security.
- SELinux provides similar (sometimes more granular) control compared to AppArmor.

---

## Want SELinux basics and examples?

I can help with:

- Checking SELinux status
- SELinux modes (Enforcing, Permissive, Disabled)
- Managing SELinux policies
- Using SELinux with Docker containers
- Using SELinux with Kubernetes pods



## AppArmor vs Seccomp

| Feature                 | AppArmor                                  | Seccomp                                   |
|-------------------------|------------------------------------------|-------------------------------------------|
| **Type of Security**    | Mandatory Access Control (MAC)            | System Call Filtering                      |
| **Scope**              | Controls file access, capabilities, network, and more | Filters and restricts Linux syscalls      |
| **Granularity**         | Fine-grained, based on file paths and permissions | Focused on syscall-level permissions       |
| **Configuration**       | Uses human-readable profiles defining allowed/denied file access, capabilities, and network | JSON-based profiles listing allowed or denied syscalls |
| **Kernel Module**       | Linux Security Module (LSM)                | Linux Kernel feature with seccomp-bpf filter |
| **Use Cases**           | Restrict container file system access, capabilities, and network usage | Limit system calls to reduce attack surface (e.g., prevent dangerous syscalls) |
| **Complexity**          | Moderate complexity; profiles can be detailed and lengthy | Simple syscall lists, easier to manage    |
| **Enforcement Modes**   | Enforce and complain modes (logs violations without blocking) | Only enforce mode (blocks or allows syscalls) |
| **Typical Users**       | Container runtimes (Docker, Kubernetes), host security | Container runtimes, sandboxed applications |
| **Effect on Performance** | Minimal overhead                          | Very low overhead                          |

---

### Summary

- **AppArmor** focuses on **file and resource access control**, governing what a container can read/write, network access, and capabilities.
- **Seccomp** focuses on **restricting system calls** a container can make, minimizing the attack surface at the syscall level.
- Both complement each other and are often used together for **defense in depth** in container security.

---

Would you like me to provide examples or best practices for using them together?

# Hardened Linux Bastion Host & Intrusion Prevention Lab
**Author:** Rao Tariq Jameel
**Role:** Computer Science Graduate | Aspiring Cloud Security Engineer

---

## Technical Proof of Work: Annotated Implementation Flow
This flow represents the exact terminal commands, configuration logic, and real-world attack simulation results.
```bash
# =================================================================
# 1. CRYPTOGRAPHIC IDENTITY SETUP (Ed25519 Keys)
# =================================================================
# I generated a modern Ed25519 key pair for high-strength encryption.
ssh-keygen -t ed25519 -C "bastion-host-key"
# ~/.ssh/id_ed25519 (Private Key) | ~/.ssh/id_ed25519.pub (Public Key)

# Transferring public key to the host:
ssh-copy-id -i ~/.ssh/id_ed25519.pub [USER]@[LOCAL_IP]


# =================================================================
# 2. SSH CONFIGURATION HARDENING (The "Rules")
# =================================================================
# Modified /etc/ssh/sshd_config to enforce the following security policy:
# -----------------------------------------------------------------
# PubkeyAuthentication yes        # Enable identity verification via keys
# PasswordAuthentication no       # Disable vulnerable password logins
# PermitEmptyPasswords no         # Block accounts with no credentials
# PermitRootLogin no              # Disable root login
# MaxAuthTries 3                  # Limit login attempts to 3
# -----------------------------------------------------------------
sudo sshd -t && sudo systemctl restart ssh


# =================================================================
# 3. INTRUSION PREVENTION (Fail2Ban Setup & Policy Tuning)
# =================================================================
sudo apt install fail2ban -y
sudo systemctl enable --now fail2ban

# Tuning: Modified /etc/fail2ban/jail.local to remove self-ignore (ignoreip)
sudo systemctl restart fail2ban


# =================================================================
# 4. ATTACK SIMULATION & LOG VERIFICATION (The Proof)
# =================================================================
# Simulated a brute-force attack from a separate terminal session.
ssh [USER]@[LOCAL_IP]  # Intentional wrong password entry

# VERIFYING THE JAIL (Live Output):
sudo fail2ban-client status sshd

# --- TERMINAL OUTPUT START ---
# Status for the jail: sshd
# |- Filter
# |  |- Currently failed: 1
# |  |- Total failed:     3
# |  `- File list:        /var/log/auth.log
# `- Actions
#    |- Currently banned: 1
#    |- Total banned:     1
#    `- Banned IP list:   127.0.0.1  <-- SUCCESS: Attack IP Jailed
# --- TERMINAL OUTPUT END ---

# Recovery:
sudo fail2ban-client set sshd unbanip 127.0.0.1


# =================================================================
# 5. NETWORK PERIMETER LOCKDOWN (UFW Firewall)
# =================================================================
sudo ufw default deny incoming          # RULE: Block EVERYTHING by default.
sudo ufw default allow outgoing         # RULE: Allow the server to initiate connections.
sudo ufw allow 22/tcp                   # RULE: Open a single "hole" for SSH traffic only.
sudo ufw --force enable                 # ACTION: Activate the security policy.


# =================================================================
# 6. CHALLENGES & TROUBLESHOOTING (Lessons Learned)
# =================================================================
# CHALLENGE 1: SSH SERVICE NOT FOUND
# Problem: initial connection refused. 
# Fix: Realized OpenSSH isn't always pre-installed; ran 'sudo apt install openssh-server'.

# CHALLENGE 2: FAIL2BAN "IGNOREIP" ISSUE
# Problem: Fail2Ban wouldn't ban my local test IP.
# Fix: Edited /etc/fail2ban/jail.local to remove 127.0.0.1 from ignoreip list.

# CHALLENGE 3: RECOVERING FROM SELF-JAIL
# Problem: Successfully banned myself and lost access.
# Fix: Used 'fail2ban-client set sshd unbanip' to restore administrative access.

# CHALLENGE 4: FIREWALL TRUST ISSUES
# Problem: UFW blocked my current session upon enablement.
# Fix: Explicitly allowed Port 22 BEFORE enabling the firewall to prevent lockout.

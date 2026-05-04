# Hardened Linux Bastion Host & Intrusion Prevention Lab
**Author:** Rao Tariq Jameel
**Role:** Computer Science Graduate | Aspiring Cloud Security Engineer

---

## Technical Proof of Work: Annotated Implementation Flow
This flow represents the exact terminal commands, configuration logic, and security measures applied to this Ubuntu environment.

```bash
# =================================================================
# 1. CRYPTOGRAPHIC IDENTITY SETUP (Ed25519 Keys)
# =================================================================
# I generated a modern Ed25519 key pair for high-strength encryption.
ssh-keygen -t ed25519 -C "bastion-host-key"              # Generate the Ed25519 private/public key pair
# ~/.ssh/id_ed25519 (Private Key) | ~/.ssh/id_ed25519.pub (Public Key)

# Transferring the public key to the server's authorized_keys file:
ssh-copy-id -i ~/.ssh/id_ed25519.pub [USER]@[LOCAL_IP]   # Securely install the public key on the host


# =================================================================
# 2. SERVICE PROVISIONING (OpenSSH Installation)
# =================================================================
sudo apt update                                         # Update local package index
sudo apt install openssh-server -y                       # Install OpenSSH server
sudo systemctl enable --now ssh                         # Enable and start SSH service


# =================================================================
# 3. SSH CONFIGURATION HARDENING (The "Rules")
# =================================================================
# Modified /etc/ssh/sshd_config to enforce the following security policy:
# -----------------------------------------------------------------
# PubkeyAuthentication yes        # Enable identity verification via keys
# PasswordAuthentication no       # Disable vulnerable password logins
# PermitEmptyPasswords no         # Block accounts with no credentials
# PermitRootLogin no              # Disable root login to prevent total system takeover
# MaxAuthTries 3                  # Limit login attempts to 3 to mitigate brute-force
# -----------------------------------------------------------------
sudo sshd -t                                            # Test configuration syntax for errors
sudo systemctl restart ssh                              # Restart service to apply hardening rules


# =================================================================
# 4. INTRUSION PREVENTION (Fail2Ban Setup & Policy Tuning)
# =================================================================
sudo apt install fail2ban -y                            # Install automated IPS
sudo systemctl enable --now fail2ban                    # Start the monitoring engine

# TUNING: Modified /etc/fail2ban/jail.local to remove self-ignore (ignoreip)
# This was a critical step to allow testing security policies against the local host.
sudo systemctl restart fail2ban                         # Apply the testing-friendly policy


# =================================================================
# 5. ATTACK SIMULATION (Verifying the IPS)
# =================================================================
# Simulated a brute-force attack to confirm the jail mechanism works.
ssh [USER]@[LOCAL_IP]                                   # Connection attempt with intentional wrong password
# (Result: Connection refused after 3 attempts due to MaxAuthTries and Fail2Ban)

sudo fail2ban-client status sshd                        # Confirm the IP was successfully jailed
# Output: Banned IP list: [ATTACKER_IP]

sudo fail2ban-client set sshd unbanip [ATTACKER_IP]      # Manually release the IP after test success


# =================================================================
# 6. NETWORK PERIMETER LOCKDOWN (UFW Firewall)
# =================================================================
sudo ufw default deny incoming                          # Block all unsolicited incoming traffic
sudo ufw default allow outgoing                         # Allow server to fetch updates
sudo ufw allow 22/tcp                                   # Open Port 22 specifically for SSH
sudo ufw --force enable                                 # Activate firewall and apply rules


# =================================================================
# 7. FINAL SYSTEM AUDIT
# =================================================================
sudo ufw status verbose                                 # Verify the "Default Deny" network posture
sudo fail2ban-client status sshd | grep "Currently"     # Check live jail status
systemctl is-enabled ssh fail2ban                       # Confirm security persists across reboots

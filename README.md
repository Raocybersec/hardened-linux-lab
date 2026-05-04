# Hardened Linux Bastion Host & Intrusion Prevention Lab
**Author:** Rao Tariq Jameel
**Role:** Computer Science Graduate | Aspiring Cloud Security Engineer

---

## Technical Proof of Work: Annotated Implementation Flow
This flow represents the exact terminal commands, configuration logic, and attack simulations performed on the Ubuntu system.
```bash
# =================================================================
# 1. SERVICE PROVISIONING (OpenSSH Installation)
# =================================================================
sudo apt update                                         # Update local package index to fetch latest software versions
sudo apt install openssh-server -y                       # Install OpenSSH server suite to enable remote connections
sudo systemctl enable --now ssh                         # Enable SSH service and start it immediately on the system
sudo systemctl status ssh | grep "Active"               # Check the service status to verify it is running successfully


# =================================================================
# 2. IDENTITY HARDENING (SSH Configuration)
# =================================================================
# Secure the identity layer by enforcing Key-Based authentication in /etc/ssh/sshd_config
# Key-only settings: PubkeyAuthentication yes | PasswordAuthentication no | PermitEmptyPasswords no
sudo systemctl restart ssh                              # Restart the SSH service to apply the hardening changes


# =================================================================
# 3. INTRUSION PREVENTION (Fail2Ban Setup & Policy Tuning)
# =================================================================
sudo apt install fail2ban -y                            # Install Fail2Ban to act as an automated security guard
sudo systemctl enable --now fail2ban                    # Enable and start the Fail2Ban engine to begin monitoring logs

# TUNING: Modify /etc/fail2ban/jail.local to remove self-ignore (ignoreip)
# This allows the system to ban my own IP for testing purposes
sudo systemctl restart fail2ban                         # Restart Fail2Ban to apply the new testing-friendly policy


# =================================================================
# 4. ATTACK SIMULATION (Verifying the IPS)
# =================================================================
# I simulated a brute-force attack by intentionally failing 5 password attempts

# Attacker Command:
ssh [USER]@[LOCAL_IP]                                   # Initiate an SSH connection to trigger authentication logs
# (Repeated until connection was refused)

# Verification on Host:
sudo fail2ban-client status sshd                        # Check the status of the SSH jail to see current bans
# Output showed Banned IP list: [ATTACKER_IP]

# Recovery Step:
sudo fail2ban-client set sshd unbanip [ATTACKER_IP]      # Manually release the blocked IP after successful test


# =================================================================
# 5. NETWORK PERIMETER LOCKDOWN (UFW Firewall)
# =================================================================
sudo ufw default deny incoming                          # Set global policy to block all unsolicited incoming traffic
sudo ufw default allow outgoing                         # Allow the server to send traffic out for updates and patches
sudo ufw allow 22/tcp                                   # Explicitly open only Port 22 for secure management traffic
sudo ufw --force enable                                 # Activate the firewall engine and apply the rule-set


# =================================================================
# 6. FINAL SYSTEM AUDIT
# =================================================================
sudo ufw status verbose                                 # Display the active firewall rules to verify the lockdown
sudo fail2ban-client status sshd | grep "Status"        # Confirm the IPS is still actively monitoring the SSH jail
systemctl is-enabled ssh fail2ban                       # Ensure all security services are set to persist across reboots

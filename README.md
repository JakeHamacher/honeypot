# Honeypot Deployment in AWS: End-to-End Guide

Deploying a honeypot in AWS let me safely attract, monitor, and analyze real attacker behavior. This guide documents my approach—from provisioning resources to uncovering actionable intelligence. It is not comprehensive, but instead intends to give the idea of my process and broadly how it can be done yourself. If you're looking to replicate my setup and learn from real-world intrusion attempts, you're in the right place.

---

## Prerequisites

Before diving in, I made sure to have the following in place:

- An active AWS account with **Administrator** or **PowerUser** privileges  
- AWS CLI set up locally (`aws configure`)  
- Working knowledge of Linux, networking, and basic security concepts  

---

## 1. AWS Network & Security Setup

Here’s the foundational architecture I created in AWS:

1. **VPC Setup**  
   - A dedicated VPC called `honeypot-vpc`  
   - One **public subnet** for the honeypot hosts  
   - One **private subnet** (no Internet access) for internal analysis servers  

2. **Security Groups**  
   - `sg-honeypot`: Opened inbound TCP/UDP to `0.0.0.0/0` to act as bait, with strict outbound limited to the analysis subnet  
   - `sg-analysis`: Allowed SSH only from my office IP, plus full access from `sg-honeypot` for traffic correlation  

3. **Network ACLs and Logging**  
   - Blocked all outbound traffic from the honeypot subnet except to the analysis subnet  
   - Enabled **VPC Flow Logs** to capture denied traffic events  

---

## 2. Provision Honeypot EC2 Instances

To serve as the trap:

1. Launched **Ubuntu Server 22.04 LTS** on an EC2 instance inside the public subnet  
2. Assigned an **Elastic IP** to ensure a consistent attack surface  
3. Attached the instance to `sg-honeypot` and tagged it as `Honeypot-01`  

---

## 3. Install & Configure Honeypot Services

### 3.1 Cowrie (SSH/Telnet Honeypot)

Cowrie served as my front-line decoy for SSH and Telnet attackers.

```bash
sudo apt update
sudo apt install python3-venv git -y
git clone https://github.com/cowrie/cowrie.git /opt/cowrie
cd /opt/cowrie
python3 -m venv cowrie-env
source cowrie-env/bin/activate
```

I went on to configure it with custom banners and added logging integration with my back-end collector.

---

## 4. My Findings from the Honeypot

After running the honeypot for several weeks, here are the trends and anomalies I discovered:

- **Frequent Offenders**  
  - Most hits originated from IPs based in Eastern Europe and Southeast Asia  
  - A handful of IPs attempted brute force every few minutes for hours on end

- **Credential Spray Attempts**  
  - Reused default credentials like `admin/admin`, `root/toor`, `pi/raspberry`  
  - Some attackers even tried username/password combos in multiple languages

- **Command Execution & Payloads**  
  - Download commands (`wget`, `curl`) aimed at pulling shell scripts from pastebins  
  - Detected attempts to drop crypto miners and even set up `reverse shells` via `nc`

- **Intrusion Timing**  
  - Spike in login attempts between 02:00–05:00 UTC, likely automation-driven  
  - Most manual/interactive sessions occurred over weekends

---

## 5. Lessons Learned & Remediation Actions I Took

From my observations, here is how I would refine production AWS architecture:

- **Network Hygiene**  
  - Restrict all production SSH access to known CIDRs  
  - Implement Security Group whitelists and disable password login in real systems

- **Improved Monitoring**  
  - Push indicators of compromise (IOCs) into my SIEM to correlate with live environments  
  - Flag suspicious IPs, ports, and command patterns for automated blocking

- **Awareness & Preparedness**  
  - Share findings with internal teams to increase awareness of brute-force trends  
  - Update internal training material with real attacker behavior

- **Honeypot Evolution**  
  - Add HTTP and SMB honeypots next to track web-based and Windows-based intrusions  
  - Added periodic log parsing scripts to surface new attack techniques weekly

---

This experience was eye-opening—it not only showed me what’s hitting the perimeter constantly, but how attackers behave once they believe they’ve breached a system. Highly recommend trying it, especially with the right guardrails in place.

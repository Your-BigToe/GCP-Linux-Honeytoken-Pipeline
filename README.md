# GCP-Linux-Honeytoken-Pipeline
Cloud based file monitoring and real time alerting pipeline.

# Cloud-based Threat Detection Pipeline

## Project Overview
This project demonstrates the deployment of a host-based intrusion detection mechanism within a cloud environment. By utilizing a Google Cloud Platform Linux instance, I configured the native Linux kernel auditing framework to monitor a deceptive high-value asset file. I engineered a custom bash automation script to continuously show live log streams and instantly generate administrative alerts upon detecting unauthorized file access events.

## Skills & Technologies Demonstrated
*  **Cloud Infrastructure:** Google Cloud Platform, Compute Engine via CLI.
*  **Operating Systems:** Linux(Ubuntu), Bash script engineering.
*  **Security Monitoring:** Host-based intrusion detection, File Integrity Monitoring, 'audit' configuration.
*  **Log Analysis:** Automation of real time log parsing ('/var/log/audit/audit.log').

---

## Deployment & Implementation Steps

### 1. Cloud Provisioning
The Ubuntu Linux enterprise environment was deployed instantly via the Google Cloud Shell utilizing the following automated infrastructure-as-code command:

'''bash
gcloud compute instances create monitored-linux-server \
  --zone=us-central1-a \
  --machine-type=e2-micro \
  --image-project=ubuntu-os-cloud \
  --image-family=ubuntu-2204-lts \
  --boot-disk-size=10GB \
  --boot-disk-type=pd-standard
  '''

### 2. Honeytoken Creation & Auditing Configuration
A fake sensitive file was planted to trap potential attackers scanning the file system. The 'audit' framework was initialized and tuned with custom rule constraints targeting Read, Write, and Attribute change system calls on the asset, tagged with the key 'honeytoken-access'.

# Create the decoy asset
echo "ADMIN_PASSWORD=UltraSecretPassword123!" > corporate_passwords.txt

# Install the monitoring engine
sudo apt update && sudo apt install auditd audispd-plugins -y

# Delpoy the real-time tracking rule
sudo auditctl -w /home/\$USER/corporate_passwords.txt -p rwa -k honeytoken-access
'''

### 3. Bash Alerting Script Logic
The following production script ('monitor_trap.sh') was written to handle live streaming data input from the central audit logs, parsing for the corresponding rule key to fire an administrative warning console notification.

'''bash
#!/bin/bash
LOG_FILE="/var/log/audit/audit.log"

echo "SOC monitor Active... Watching for unauthorized honeytoken access."

sudo tail -f "\$LOG_FILE" | while read -r LINE; do
  if [[ "\$LINE" ==*"honeytoken-access"* ]]; then
    echo "ALERT: UNAUTHORIZED FILE ACCESS DETECTED!"
    echo "Details: \$LINE"
    echo"-----------------------------------------"
  fi
done
'''

---

## Verification & Proof of Concept

To test the resilience of the detection pipeline, a separate user session simulated an adversarial file attack by reading the restricted decoy asset ('cat corporate_passwords.txt').

The pipeline successfully captured the system call event, categorized the log parameters, and triggered the automated logic stream immediately.

---


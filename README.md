# WN10-CC-000360
# 🛡️ Vulnerability Management Lab – WN10-CC-000360

**Control Title:** WinRM client must not use Digest authentication  
**STIG ID:** WN10-CC-000360  
**Compliance Frameworks:** DISA STIG, NIST 800-53 AC-17  
**Platform:** Azure + Windows 10 Pro + Tenable + PowerShell  
**Vulnerability Class:** Insecure Authentication Protocol  
**Remediation Method:** Registry Hardening via PowerShell

---

## 🎯 Objective

Simulate and remediate the use of **Digest authentication** in the **WinRM client**, which poses a risk due to its weak hash-based credential exposure. This guide supports **hands-on cybersecurity training** in vulnerability lifecycle management using enterprise-grade tools.

---

## 📚 Table of Contents

1. [Azure VM Setup](#azure-vm-setup)  
2. [Vulnerability Simulation](#vulnerability-simulation)  
3. [Tenable Scan Configuration](#tenable-scan-configuration)  
4. [Initial Scan Results](#initial-scan-results)  
5. [Remediation via PowerShell](#remediation-via-powershell)  
6. [Post-Remediation Verification](#post-remediation-verification)  
7. [Security Rationale](#security-rationale)  
8. [Cleanup & Decommissioning](#cleanup--decommissioning)  
9. [Appendix: Commands](#appendix-commands)

---

## ☁️ Azure VM Setup

### 🔸 VM Parameters

| Parameter            | Value                          |
|----------------------|---------------------------------|
| OS Image             | Windows 10 Pro (Gen2)          |
| VM Size              | Standard D2s v3                |
| Resource Group       | `vm-lab-winrm-digest`          |
| Region               | Closest Azure Region           |
| Admin Username       | Use a secure password (NOT `labuser/Cyberlab123!`) |

### 🔸 Network Security Group (NSG)

- ✅ Allow inbound **RDP (TCP 3389)**
- ✅ Optionally allow **WinRM (TCP 5985)** for remote Tenable access
- 🔒 Block all unnecessary inbound traffic

---

## 🔧 VM Configuration

### 🔹 Disable Windows Firewall

Run `wf.msc` → Disable Domain, Private, and Public profiles.

### 🔹 Enable Credentialed Scanning

```powershell
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" `
  -Name "LocalAccountTokenFilterPolicy" -Value 1 -Type DWord -Force
```

📸 **Screenshot Placeholder:** `Screenshot_01_VM_Config_WinRM.png`

---

## 💥 Vulnerability Simulation

### 🔸 Risk Context

Digest authentication allows weak, reversible hashes that can be captured and cracked during man-in-the-middle attacks or lateral movement.

### 🔸 Set Digest Auth to Enabled (Simulated Insecure State)

```powershell
New-Item -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\WinRM\Client" -Force
Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\WinRM\Client" `
  -Name "Digest" -Value 1
```

📸 **Screenshot Placeholder:** `Screenshot_02_Digest_Enabled_Registry.png`

---

## 🔍 Tenable Scan Configuration

### Template: **Advanced Network Scan**  
Compliance Audit: **DISA Microsoft Windows 10 STIG**

### 🔹 Key Settings

- Enable Remote Registry
- Enable Admin Shares (C$)
- Windows Credentials (local admin)

### 🔹 Discovery Options

- Ping remote host
- TCP full port scanner
- Thorough checks
- Use credentials only

📸 **Screenshot Placeholder:** `Screenshot_03_Tenable_Template_WinRM.png`

---

## 🧪 Initial Scan Results

| Vulnerability ID     | WN10-CC-000360                |
|----------------------|-------------------------------|
| Status               | ❌ Fail                        |
| Plugin Output        | Digest authentication is enabled |
| Expected Value       | 0                              |
| Actual Value         | 1                              |

📸 **Screenshot Placeholder:** `Screenshot_04_Tenable_Scan_Before.png`

---

## 🛠️ Remediation via PowerShell

### 🔹 Disable Digest Authentication in WinRM Client

```powershell
Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\WinRM\Client" `
  -Name "Digest" -Value 0
```

📸 **Screenshot Placeholder:** `Screenshot_05_Digest_Disabled_Registry.png`

### 🔹 Optional: Confirm Registry State

```powershell
Get-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\WinRM\Client" `
  -Name "Digest"
```

---

## ✅ Post-Remediation Verification

### 🔹 Step 1: Reboot (if needed)

```powershell
Restart-Computer
```

### 🔹 Step 2: Re-Scan with Tenable

| STIG Control         | Status       |
|----------------------|--------------|
| WN10-CC-000360       | ✅ Pass       |

📸 **Screenshot Placeholder:** `Screenshot_06_Tenable_Scan_After.png`

---

## 🔐 Security Rationale

### 🔸 Digest Authentication Risks

- Uses MD5 hash-based credential exchange
- Easily susceptible to brute-force and replay attacks
- Not considered secure in modern enterprise environments

### 🔸 Secure Alternatives

- **Negotiate**, **Kerberos**, or **Certificate-based** authentication
- These enforce mutual trust and stronger encryption

---

## 🧼 Cleanup & Decommissioning

### ✅ Post-Lab Cleanup

1. Document final scan results and archive screenshots.
2. Remove registry test entries if necessary.
3. Delete Azure resource group to save costs:

```bash
az group delete --name vm-lab-winrm-digest --yes --no-wait
```

---

## 📎 Appendix: Commands

| Task                         | PowerShell Command |
|------------------------------|--------------------|
| Simulate Insecure State      | `Set-ItemProperty -Name "Digest" -Value 1` |
| Remediate                    | `Set-ItemProperty -Name "Digest" -Value 0` |
| Verify                       | `Get-ItemProperty ... WinRM\Client` |
| Enable Token Filter Policy   | `Set-ItemProperty -Name "LocalAccountTokenFilterPolicy" -Value 1` |
| Restart VM                   | `Restart-Computer` |

---

✅ **Lab Complete**

You've successfully simulated and remediated **insecure Digest authentication** in the **WinRM client**, aligned with STIG control `WN10-CC-000360`. All steps are reproducible for enterprise vulnerability management training.


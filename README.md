# Analyze and Optimize Network Security

## Research Network Security Analysis Tools

There are many options for network vulnerability scanning. Most are paid services, but some free options do exist. However, very few of these scanners run natively on Windows Desktop OS. Most require a virtual machine running a Unix-based OS to perform scanning. Some of these options include:

- **Nessus Essentials**: Free account and software, runs natively in Windows, powerful vulnerability scanner, limited to 16 IP addresses for scanning.
- **Microsoft Defender**: Native to Windows OS, provides host scanning and built-in firewall, vulnerability assessments, and suggested mitigations. Not capable of scanning the network and other hosts.
- **GVM (formerly called OpenVAS)**: Does not run in Windows, able to run on Linux VM (Kali Linux) in a Windows environment.

We will set up VMs in our later steps, so for now, we will use Nessus Essentials running natively in Windows:

- Visit [https://www.tenable.com/products/nessus/nessus-essentials](https://www.tenable.com/products/nessus/nessus-essentials) and register for an activation code by providing your first and last name and email address.
- This downloads the .msi installer file.
- Run this file and install Nessus Essentials.
- Create a username and password for signing into Nessus Essentials.
- This allows you to fully install Nessus Essentials.
- Do not interact with the Nessus GUI while plugins are compiling. This takes a few minutes.

A pop-up to scan IP addresses for network hosts appears, and you may enter IP addresses individually or a range of IPs. I chose to scan the entire subnet and let it discover hosts independently. Nessus scanned the lab subnet (Subnet 2) `192.168.2.0/24`. It discovered all hosts on the subnet. The hosts discovered are listed below in descending order, starting with the host running the Nessus scan and ending at the subnet gateway (firewall):

- GMKtec NUCbox M5 – `192.168.2.105`
- Lenovo Laptop – `192.168.2.102`
- Netgear Managed Switch – `192.168.2.2`
- Netgate Firewall OPT port – `192.168.2.1`

<p align="center">
  <br/>
  <img src="https://imgur.com/Hu7OlX8.png" height="80%" width="80%" alt="Table1.1"/><br /><br />
</p>

The discovered hosts are shown in the figure above and are all included in the default scan that is reported on in the next section.

## Analyze Security Scans and Hardening Recommendations

After the scan was completed, the Nessus interface populated with a list of 37 vulnerabilities. Most elements of the Nessus output (colored blue in the ‘Severity’ column) were default flags that provided information on processes that occurred during the scan but pose no risk, such as host MAC detection and host OS detection. 

Host MAC and OS detection can also be done with basic tools like nmap or Zenmap. These entries are included in the Nessus output to provide information and are flagged as “info” in the severity column (colored blue) instead of “low”, “med”, or “high” (which are yellow, orange, and red, respectively).

Only 4 vulnerabilities received Common Vulnerability Scoring System (CVSS) scores. These ranged from 2.1 – 6.5 (low – med) shown in the figure below.

<p align="center">
  <br/>
  <img src="https://imgur.com/tGARO4u.png" height="80%" width="80%" alt="Table1.1"/><br /><br />
</p>

The first two scores listed in the figure above are for the firewall and are related to DHCP server detection and ICMP timestamp disclosure. Regarding the DHCP vulnerability, the report reads, “It does not demonstrate any vulnerability, but a local attacker may use DHCP to become intimately familiar with the associated network.”

The ICMP timestamp vulnerability refers to internal attackers again. Hosts are configured to respond to ICMP packets (pings) from other internal hosts on the network. This is beneficial for network troubleshooting and diagnostic data. The benefits outweigh the risks in this case. Additionally, the WAN interface of my firewall has a rule to block all ICMP packets, regardless of subtype; they are to be dropped silently, and no type of response is sent (echo, reset).

Next, the laptop host has two medium CVSS flags set. One is for the “untrusted SSL certificate” of the Nessus GUI itself, which is displayed in a browser window for the service running on the localhost. It can be trusted because we are using a certificate for an internal service and public trust is not necessary. We are accessing the Nessus GUI on “localhost,” meaning we have control over the server, and we can be sure we are connected to the intended service. This is normal practice for internal tools on a small, controlled network behind a firewall or VPN.

Our last vulnerability flag, the second medium CVSS, is an issue that we will address directly. Our Nessus report reads: “Signing is not required on the remote SMB server. An unauthenticated, remote attacker can exploit this to conduct man-in-the-middle attacks against the SMB server.” SMB (Server Message Block) is a network file-sharing protocol primarily used for providing shared access to files, printers, and serial ports among nodes on a network. It is common in Windows environments. The remediation to this vulnerability is detailed in the next section.

## Perform Hardening of Devices and Network

### Enable SMB Signing

- Press `Win + R` to open the run dialog.
- Type `secpol.msc` and press Enter.
  - This opens the Local Security Policy.
- In the left column, open **Local Policies > Security Options**.
- Locate the field **‘Microsoft network server: Digitally sign communications (always)’**.
- Double-click the field, which opens a pop-up window.
  - Set it to **Enabled** and click **OK**.

### Restart SMB Services

- Press `Win + R` to open the run dialog.
- Type `services.msc` and press Enter.
- Locate the **‘Server’** field and right-click it, then select **Restart**.
- Locate the **‘Workstation’** field and right-click it, then select **Restart**.

### Verify SMB is Enabled

- Open PowerShell (right-click Windows button, select **Terminal (Admin)**).
- Type `Get-SmbClientConfiguration | Select-Object EnableSecuritySignature`.
  - This should return the value of `= True` (see the figure below).

<p align="center">
  <br/>
  <img src="https://imgur.com/beAOQM2.png" height="80%" width="80%" alt="Table1.1"/><br /><br />
</p>

## Basic host and network hardening
•	Strong passwords (length, alphanumeric, special characters)
•	Configure firewall rules (completed in earlier sections)
•	Antivirus - antimalware software on each host (pre-existing)
•	Configure firewall rules (completed in earlier sections)
•	Log security events (may explore logging and automated scanning in the future)
•	Subnetting – segmentation – VLANs (completed in earlier sections)
•	Strong wireless encryption (WPA3)
•	Regular software / driver updates (AV – Windows – Driver Easy)

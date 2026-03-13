pfSense Hardening Checklist 2026

**Secure Installation & Initial Configuration**  
Verify installation media integrity using checksums or signatures. Deploy on dedicated or hardened hardware/virtualization platforms. Immediately change default credentials (`admin`/`pfsense`) after first boot. Restrict physical and hypervisor console access to prevent unauthorized tampering.

**Management Plane Security**  
Never expose the WebGUI to the WAN. Restrict access to management VLANs, bastion hosts, or VPN-only connections. Enforce strong TLS settings (e.g., TLS 1.3), disable unused services (SSH, API), and configure idle session timeouts (e.g., 30 minutes). Change the default HTTPS port (e.g., to `8443`) for added obscurity.

**Authentication, Authorization & RBAC**  
Enforce strong password policies and **mandatory MFA (TOTP, RADIUS, LDAP-backed)** for all administrative users. Use centralized authentication (LDAP/RADIUS). Separate roles: administrators, network operators, and read-only auditors. Disable the default `admin` user and create a custom admin account with no shell access.

**Network Interfaces & Segmentation**  
Clearly segment networks: WAN, LAN, DMZ, and management. Disable unused interfaces. Enable anti-spoofing and bogon filtering on WAN interfaces. Prevent private IP ranges and reserved networks from being routed from the WAN.

**Firewall Rules & Policy Design**  
Adopt a **default-deny policy**. Only allow explicitly defined traffic with specific source/destination and port rules. Avoid `ANY → ANY` rules. Log all security-relevant traffic. Implement **egress filtering** to block outbound malware communication and data exfiltration.

**VPN & Remote Access Security**  
Use strong cryptography (modern ciphers, SHA-2+). Enforce MFA for all VPN users. Separate admin and user VPNs. Restrict access by role and subnet. Disable split tunneling unless risk-assessed. Rotate keys and credentials regularly.

**Logging, Monitoring & Alerting**  
Enable firewall, system, and authentication logs. Forward logs to a **centralized SIEM** (e.g., ELK, Graylog). Monitor admin logins, configuration changes, and VPN activity. Use **Suricata** and **pfBlockerNG** for IDS/IPS and threat blocking. Regularly review logs for anomalies.

**Additional Hardening Measures**  
- **Disable SSH** unless required; if enabled, restrict access via IP or use key-based authentication.  
- **Change the default WebGUI port** to a non-standard value (e.g., `8443`).  
- **Enable login protection**: Set threshold to 10 failed attempts, block time to 3600 seconds, detection window to 86400 seconds.  
- **Regularly update pfSense** and apply patches.  
- **Automate backups**: Use encrypted auto-backup (cloud/local) with a backup count > 10.  
- **Install Service Watchdog** to auto-restart critical daemons.  
- Consider **Zenarmor** for next-gen firewall capabilities (advanced threat detection, policy enforcement).

> ✅ **Key Takeaway**: Hardening is ongoing. Combine **CIS benchmarks**, **PCI DSS standards**, and real-world best practices. Regular audits, updates, and log reviews are essential for resilience in 2026.

## Sources :point_down:
- [pfSense Secure Provisioning & Hardening Guide](https://socfortress.medium.com/pfsense-secure-provisioning-hardening-guide-dfbd3fdcc89b)
- [10 pfSense Setup Changes to Make Post Install](https://linuxblog.io/pfsense-setup-post-install/)
- [pfsense firewall + additional security measures](https://www.reddit.com/r/PFSENSE/comments/1erbw28/pfsense_firewall_additional_security_measures/)
- [Network Hardening – pfSense Intranet](https://eaglepubs.erau.edu/mastering-enterprise-networks-labs/chapter/pfsense-firewall-intranet-2/)
- [https://www.zenarmor.com/docs/network-security-tutorials/pfsense-security-and-hardening-best-practice-guide](https://www.zenarmor.com/docs/network-security-tutorials/pfsense-security-and-hardening-best-practice-guide)
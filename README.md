# Penetration Testing Report: Footprinting & Network Scanning

> **Disclaimer:** This assessment was conducted as part of an authorised cybersecurity training lab with Networkwalks Academy. It is for educational purposes only. No exploitation was performed. Never scan systems or networks without appropriate permission.

| Project Information         | Details                                                                 |
| --------------------------- | ----------------------------------------------------------------------- |
| **Prepared by**             | Kelechi Happiness Uzoma                                                 |
| **Batch**                   | B084C                                                                   |
| **Instructor**              | Waqas Karim                                                             |
| **Footprinting Target**     | networkwalks.com                                                        |
| **Network Scanning Target** | 192.168.1.0/24 (local network)                                          |
| **Testing Dates**           | 8–9 November 2026                                                       |
| **Platforms**               | Kali Linux and Windows                                                  |
| **Tools**                   | WHOIS, WhatWeb, nslookup, curl, wafw00f, dnsrecon, Zenmap and Nmap 7.98 |
| **Report Version**          | 1.1                                                                     |

---

## Table of Contents

1. Executive Summary
2. Scope and Authorisation
3. Methodology and Tools
4. Phase 1 – Footprinting Results
5. Phase 2 – Network Scanning Results
6. Summary of Findings
7. Recommendations
8. Limitations
9. Conclusion

---

## 1. Executive Summary

This project documents two initial phases of a penetration-testing assessment: footprinting of `networkwalks.com` and network scanning of the local subnet `192.168.1.0/24`.

The assessment focused on collecting publicly available information, identifying technologies, examining DNS records, reviewing HTTP response headers, detecting a Web Application Firewall (WAF), discovering active network hosts and identifying open ports.

No exploitation or destructive testing was performed.

### Key Findings

**Footprinting:**

- The domain resolved to IPv4 address `192.232.216.135`.
- The website was identified as using WordPress and Apache.
- WordPress and WordPress Download Manager version information was disclosed in the collected results.
- A ModSecurity (SpiderLabs) Web Application Firewall was detected.
- The HTTP service redirected requests to HTTPS.
- Several recommended HTTP security headers were not present in the captured response.
- DNS records disclosed name server information and an SPF policy ending in `~all`.

**Network Scanning:**

- Seven active hosts were identified on the `192.168.1.0/24` subnet.
- Four hosts had open TCP ports within the scanned port range.
- Ports commonly associated with SMB were detected on the gateway device.
- HTTP services were detected on two network devices.
- Additional services were identified on a device with an Arcadyan MAC vendor association.

### Overall Assessment

The assessment identified information-disclosure issues, configuration-hardening opportunities and network services requiring further investigation.

No directly exploitable vulnerability was confirmed during this assessment. Risk ratings should therefore be treated as preliminary until the affected services and configurations are validated.

---

## 2. Scope and Authorisation

| Item                    | Details                                                                       |
| ----------------------- | ----------------------------------------------------------------------------- |
| In-scope domain         | networkwalks.com                                                              |
| In-scope network        | 192.168.1.0/24                                                                |
| Assessment activities   | Footprinting, DNS enumeration, host discovery and TCP port scanning           |
| Out-of-scope activities | Exploitation, brute forcing, denial-of-service testing and social engineering |
| Authorisation           | Conducted within the Networkwalks cybersecurity training lab                  |
| Scanner IP              | 192.168.1.12 (inferred from scan results; confirmation recommended)           |

The assessment was limited to information gathering and scanning activities.

---

## 3. Methodology and Tools

The project followed a structured information-gathering and network-discovery methodology.

| Phase            | Tool      | Purpose                                               |
| ---------------- | --------- | ----------------------------------------------------- |
| Footprinting     | WHOIS     | Domain registration information                       |
| Footprinting     | WhatWeb   | Website technology identification                     |
| Footprinting     | nslookup  | DNS resolution                                        |
| Footprinting     | curl      | HTTP response header inspection                       |
| Footprinting     | wafw00f   | Web Application Firewall detection                    |
| Footprinting     | dnsrecon  | DNS record enumeration                                |
| Network Scanning | Zenmap    | Graphical network scanning and topology visualisation |
| Network Scanning | Nmap 7.98 | Host discovery and TCP port scanning                  |

Footprinting output was saved in the `~/week2-pm1` directory using the `tee` command.

---

## 4. Phase 1 – Footprinting Results

### 4.1 WHOIS Lookup

**Command:**

```bash
whois networkwalks.com | tee 1_whois.txt
```

**Results:**

| Field                 | Value                                                                                           |
| --------------------- | ----------------------------------------------------------------------------------------------- |
| Domain                | NETWORKWALKS.COM                                                                                |
| Registry Domain ID    | 2452319255_DOMAIN_COM-VRSN                                                                      |
| Registrar             | GoDaddy.com, LLC                                                                                |
| IANA ID               | 146                                                                                             |
| Creation Date         | 2019-11-06                                                                                      |
| Updated Date          | 2025-11-12                                                                                      |
| Expiry Date           | 2027-11-06                                                                                      |
| Domain Status         | clientDeleteProhibited, clientRenewProhibited, clientTransferProhibited, clientUpdateProhibited |
| Registry Name Servers | NS6135.HOSTGATOR.COM, NS6136.HOSTGATOR.COM                                                      |
| DNSSEC                | Unsigned                                                                                        |

**Analysis:**

- The domain is registered through GoDaddy.
- The registry records list HostGator name servers.
- Several domain protection statuses are enabled.
- Registrant contact information is privacy-protected.
- Additional name server entries appeared in the registrar record and should be checked against the authoritative delegation.

**Evidence:**



### 4.2 Web Technology Fingerprinting (WhatWeb)

**Command:**

```bash
whatweb networkwalks.com | tee 2_whatweb.txt
```

**Results:**

| Item               | Result                                                 |
| ------------------ | ------------------------------------------------------ |
| HTTP Response      | 301 Moved Permanently                                  |
| HTTPS Response     | 200 OK                                                 |
| IP Address         | 192.232.216.135                                        |
| Web Server         | Apache                                                 |
| CMS                | WordPress 7.1.4                                        |
| Plugin             | WordPress Download Manager 3.3.76                      |
| JavaScript Library | jQuery 3.7.1                                           |
| Page Title         | Networkwalks Academy                                   |
| Email Address      | [info@networkwalks.com](mailto\:info@networkwalks.com) |

**Analysis:**

- The website redirects HTTP traffic to HTTPS.
- WordPress and plugin version information was disclosed.
- Exposed version information can help an attacker identify software that may require security updates.
- WhatWeb also reported Bootstrap 7.1.4. This identification may be a false positive and should be verified independently.
- The collected headers included caching-related information.

**Evidence:**



### 4.3 DNS Resolution (nslookup)

**Command:**

```bash
nslookup networkwalks.com | tee 3_nslookup.txt
```

**Results:**

| Field         | Value             |
| ------------- | ----------------- |
| DNS Resolver  | 8.8.8.8           |
| Resolver Port | 53                |
| Response Type | Non-authoritative |
| A Record      | 192.232.216.135   |

**Analysis:**

The DNS lookup successfully resolved the domain to an IPv4 address.

The collected result returned one A record. This does not establish whether additional load balancing, CDN services or origin-protection mechanisms are present.

**Evidence:**



### 4.4 HTTP Security Header Analysis (curl)

**Command:**

```bash
curl -I https://networkwalks.com | tee 4_headers.txt
```

**Results:**

| Header                  | Observation                           |
| ----------------------- | ------------------------------------- |
| HTTP Status             | HTTP/2 200                            |
| Server                  | Apache                                |
| Content-Type            | text/html; charset=UTF-8              |
| Set-Cookie              | \_\_wpdm_client (Secure and HttpOnly) |
| Referrer-Policy         | no-referrer-when-downgrade            |
| Permissions-Policy      | Present                               |
| Link                    | WordPress REST API endpoint           |
| X-Nginx-Cache           | WordPress                             |
| X-Endurance-Cache-Level | 0                                     |

**Security Observations:**

The captured response did not include the following recommended security headers:

- Strict-Transport-Security (HSTS)
- Content-Security-Policy (CSP)
- X-Frame-Options
- X-Content-Type-Options

The captured cookie also did not show a SameSite attribute.

The WordPress REST API endpoint was advertised through the Link header. Its presence alone does not establish a vulnerability or confirm that sensitive information is exposed.

**Positive Observations:**

- HTTPS was successfully accessed.
- The cookie included Secure and HttpOnly attributes.
- The Apache version was not disclosed in the captured response.

**Evidence:**



### 4.5 Web Application Firewall Detection (wafw00f)

**Command:**

```bash
wafw00f networkwalks.com | tee 5_waf.txt
```

**Results:**

| Field         | Result                   |
| ------------- | ------------------------ |
| WAF Detection | ModSecurity (SpiderLabs) |
| Requests Sent | 2                        |

**Analysis:**

The tool identified ModSecurity as the Web Application Firewall protecting the website.

A WAF provides an additional security layer by inspecting and filtering web traffic. Its detection does not guarantee that the website is protected against every attack.

**Evidence:**



### 4.6 DNS Enumeration (dnsrecon)

**Command:**

```bash
dnsrecon -d networkwalks.com | tee 6_dnsrecon_fixed.txt
```

**Results:**

| Record    | Name                                  | Value                                              |
| --------- | ------------------------------------- | -------------------------------------------------- |
| SOA       | ns6135.hostgator.com                  | 50.87.144.87                                       |
| NS        | ns6135.hostgator.com                  | 50.87.144.87                                       |
| NS        | ns6136.hostgator.com                  | 192.232.216.131                                    |
| MX        | mail.networkwalks.com                 | 192.232.216.135                                    |
| A         | networkwalks.com                      | 192.232.216.135                                    |
| TXT       | networkwalks.com                      | Google site verification                           |
| TXT (SPF) | networkwalks.com                      | v=spf1 +a +mx +ip4:50.87.144.87 +include.com \~all |
| SRV       | \_autodiscover.\_tcp.networkwalks.com | cpanelemaildiscovery.cpanel.net:443                |
| DNSSEC    | —                                     | Reported as unsigned / no answer                   |

**Analysis:**

- The DNS enumeration identified two name servers.
- The collected results disclosed a BIND version of 9.16.23-RH.
- The website and mail hostnames resolved to the same IP address in the collected results. This does not prove that they share the same physical server.
- The SPF record ends with `~all`, which represents a soft-fail policy.
- No DMARC record was shown in the collected output. A dedicated DMARC lookup is required to confirm its presence or absence.
- DNSSEC was reported as unsigned in the collected WHOIS information. This should be verified with an authoritative DNSSEC check.

**Tool Note:**

The initial dnsrecon execution encountered an error. The tool was upgraded to version 1.6.3, after which the enumeration completed successfully.

**Evidence:**



---

## 5. Phase 2 – Network Scanning Results

### 5.1 Scan Details

| Item               | Details                         |
| ------------------ | ------------------------------- |
| Tool               | Zenmap / Nmap 7.98              |
| Platform           | Windows                         |
| Scan Profile       | Quick Scan                      |
| Command            | `nmap -T4 -F 192.168.1.0/24`    |
| Start Time         | 9 November 2026, 13:35 (+01:00) |
| Network Range      | 192.168.1.0/24                  |
| Total IP Addresses | 256                             |
| Active Hosts       | 7                               |
| Completion Time    | 8.56 seconds                    |

**Command Explanation:**

- `nmap` launches the network scanning tool.
- `-T4` selects Nmap's faster timing template.
- `-F` enables Fast Scan mode, reducing the default port list to 100 common ports per scanned protocol.
- `192.168.1.0/24` specifies the target subnet.

The scan identified seven responsive hosts.

**Evidence:**



### 5.2 Host and Port Discovery Results

MAC addresses have been partially masked for privacy.

| IP Address   | MAC / Vendor   | Open Ports             | Other Port Results | Possible Device Role              |
| ------------ | -------------- | ---------------------- | ------------------ | --------------------------------- |
| 192.168.1.1  | China Mobile   | 53, 80, 139, 445       | 96 closed          | Gateway router                    |
| 192.168.1.2  | Randomised MAC | None                   | 100 closed         | Client device                     |
| 192.168.1.3  | Randomised MAC | None                   | 100 closed         | Client device                     |
| 192.168.1.6  | Huawei         | 53, 80, 443            | 97 filtered        | Router, modem or access point     |
| 192.168.1.8  | Randomised MAC | None                   | 100 closed         | Client device                     |
| 192.168.1.9  | Arcadyan       | 3000, 8008, 8009, 8443 | 96 closed          | Possible media or network device  |
| 192.168.1.12 | Not shown      | 135, 139, 445          | 97 closed          | Possible Windows scanning machine |

**Important:** Port numbers and Nmap service labels provide initial clues, not definitive identification of the software running on a device.

**Evidence:**





### 5.3 Host Analysis

**192.168.1.1 – Gateway Router**

- TCP port 53 is commonly associated with DNS.
- TCP port 80 is commonly associated with HTTP.
- TCP ports 139 and 445 are commonly associated with NetBIOS and SMB.
- The HTTP service may provide a router management interface, but this was not confirmed.
- The SMB-related services should be investigated to establish whether file sharing is enabled and which SMB version is running.

**192.168.1.6 – Huawei Device**

- TCP ports 53, 80 and 443 were identified as open.
- Nmap classified 97 scanned ports as filtered.
- Filtered ports indicate that Nmap could not determine w**9 – Arcadyan Device**
- Ports 8008, 8009 and 8443 were identified as open.
- These ports may be associated with media streaming or device-management services.
- Nmap labelled port 8009 as AJP13 and port 3000 as PPP based on port associations.
- The actual applications running on these ports were not confirmed.
- Further service detection is recommended.

**192.168.1.12 – Possible Windows Scanning Machine**

- TCP ports 135, 139 and 445 were identified as open.
- These ports are commonly associated with Windows RPC and file-sharing services.
- The host appears to be the scanning machine based on the scan context, but this should be confirmed.

**192.168.1.2, 192.168.1.3 and 192.168.1.8 – Client Devices**

- No open ports were identified within the 100 TCP ports scanned.
- All 100 scanned ports were reported as closed.
- The MAC addresses were randomised or locally administered, which can occur when devices use private Wi-Fi addresses.
- The exact device types were not confirmed.

### 5.4 Network Topology Analysis

Zenmap was used to visualise the discovered network devices and their scan results.

The topology view helps present the discovered hosts and their apparent network relationships.

However, a topology diagram alone does not prove that every device can communicate directly with every other device or that network segmentation is absent.

**Evidence:**



### 5.5 Scan Limitations

The network scan had the following limitations:

- Only the 100 common TCP ports selected by Fast Scan were examined.
- Other TCP ports were not assessed.
- UDP scanning was not performed.
- Service version detection was not performed.
- Operating system detection was not performed.
- Device roles were estimated from MAC vendor information and common port associations.
- The scan did not establish whether discovered services were vulnerable.

A follow-up assessment should validate the identified services and their versions before assigning confirmed vulnerability ratings.

---

## 6. Summary of Findings

| ID   | Finding                                                                  | Evidence            | Preliminary Assessment      |
| ---- | ------------------------------------------------------------------------ | ------------------- | --------------------------- |
| F-01 | WordPress and plugin version information disclosed                       | WhatWeb             | Low                         |
| F-02 | Recommended HTTP security headers absent from captured response          | curl                | Low                         |
| F-03 | SPF policy uses soft fail (`~all`)                                       | dnsrecon            | Low                         |
| F-04 | BIND version information disclosed                                       | dnsrecon            | Informational               |
| F-05 | DNSSEC reported unsigned                                                 | WHOIS / dnsrecon    | Requires verification       |
| F-06 | WordPress REST API endpoint advertised                                   | curl                | Informational               |
| F-07 | Website and mail hostnames resolve to the same IP                        | nslookup / dnsrecon | Informational               |
| F-08 | Public contact email identified                                          | WhatWeb             | Informational               |
| F-09 | Different name server entries appeared in registrar and registry records | WHOIS               | Requires verification       |
| F-10 | TCP ports 139 and 445 open on gateway router                             | Nmap                | Requires service validation |
| F-11 | HTTP services detected on 192.168.1.1 and 192.168.1.6                    | Nmap                | Requires validation         |
| F-12 | Multiple ports open on Arcadyan device, including unidentified port 3000 | Nmap                | Requires validation         |
| F-13 | Windows RPC and SMB-associated ports open on 192.168.1.12                | Nmap                | Requires validation         |
| F-14 | Seven responsive devices identified within the scanned subnet            | Nmap                | Informational               |

### Positive Security Observations

- HTTPS access was successful.
- HTTP redirected to HTTPS.
- The website used a Secure and HttpOnly cookie.
- ModSecurity WAF was detected.
- Domain protection statuses were enabled.
- The Apache version was not disclosed in the captured response.
- Three discovered client devices had no open ports within the scanned TCP port range.

---

## 7. Recommendations

### 7.1 Website Security Recommendations

1. **Keep software updated:** Regularly update WordPress, plugins and themes. Verify installed versions and check relevant security advisories.

2. **Improve HTTP security headers:** Review the configuration of HSTS, Content-Security-Policy, X-Frame-Options or CSP frame-ancestors, and X-Content-Type-Options.

3. **Review cookie security:** Consider setting an appropriate SameSite attribute on cookies.

4. **Strengthen email authentication:** Review SPF configuration, confirm all legitimate sending services before changing the policy to `-all`, and configure DKIM and DMARC.

5. **Review DNSSEC:** Confirm DNSSEC support and enable it where appropriate.

6. **Review DNS information disclosure:** Ask the hosting provider whether unnecessary version banners can be suppressed.

7. **Review WordPress REST API exposure:** Ensure that user information and other sensitive resources are not accessible to unauthorised users.

8. **Verify DNS configuration:** Confirm that registrar and registry name server information is consistent with the intended DNS delegation.

### 7.2 Internal Network Security Recommendations

9. **Review SMB services:** Identify why ports 139 and 445 are open on the gateway. Disable unnecessary file-sharing services and avoid SMBv1.

10. **Secure router management:** Use strong administrator passwords, update firmware and use HTTPS for management where supported.

11. **Investigate port 3000:** Identify the application running on 192.168.1.9 and determine whether it is required.

12. **Protect Windows file sharing:** Apply appropriate Windows Firewall rules and restrict file-sharing access to trusted devices.

13. **Review network segmentation:** Consider separating guest devices and IoT equipment from trusted computers using supported guest-network or VLAN controls.

14. **Perform further service enumeration:** Use authorised service and version detection to identify the applications behind the discovered ports.

---

## 8. Limitations

This assessment was limited to footprinting and basic network scanning.

The following activities were not performed:

- Exploitation or proof-of-concept testing.
- Vulnerability scanning and confirmation.
- Password auditing or brute-force testing.
- UDP port scanning.
- Full TCP port scanning.
- Detailed operating system fingerprinting.
- Authenticated configuration reviews.
- Network traffic interception.
- Testing of lateral movement between discovered devices.

The findings represent observations from the tools and commands used during the assessment. They should not be interpreted as proof that a particular system is vulnerable or secure.

---

## 9. Conclusion

This project provided practical experience in two important penetration-testing activities: footprinting and network scanning.

During the footprinting phase, I used WHOIS, WhatWeb, nslookup, curl, wafw00f and dnsrecon to gather information about the target domain, identify website technologies, examine DNS records, inspect HTTP response headers and detect a Web Application Firewall.

During the network scanning phase, I used Zenmap and Nmap to discover active devices, identify open ports and visualise the local network.

The assessment identified several areas that require further investigation, particularly the SMB-associated ports on the gateway device, the HTTP services on network devices and the security configuration of the public website.

This project strengthened my practical understanding of reconnaissance, network discovery, port scanning, information gathering and security assessment.

The results provide a foundation for further learning in service enumeration, vulnerability assessment and defensive security monitoring.

---

## Author

**Kelechi Happiness Uzoma**

Cybersecurity Student | Aspiring Cybersecurity Analyst

Networkwalks Academy – Batch B084C

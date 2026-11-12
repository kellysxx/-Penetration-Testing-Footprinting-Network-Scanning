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
- Four hosts returned open TCP ports within the 100 common TCP ports scanned by Nmap.
- The gateway device (`192.168.1.1`) was confirmed as the default gateway using the Windows network configuration.
- The Windows scanning computer was identified as `192.168.1.12`, with TCP ports 135, 139 and 445 reported as open.
- The Huawei-associated device (`192.168.1.6`) returned open ports 53, 80 and 443.
- The Arcadyan-associated device (`192.168.1.9`) returned open ports 3000, 8008, 8009 and 8443. The exact services were not confirmed.

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
| Scanner IP | 192.168.1.12 (confirmed Windows scanning computer) |           

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
<img width="1920" height="892" alt="1_whois" src="https://github.com/user-attachments/assets/e73fe399-a66f-4c99-9ed4-ebcdf9e09798" />


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
<img width="1920" height="891" alt="2_whatweb" src="https://github.com/user-attachments/assets/14ff1174-de48-4744-b2a5-4db9df3b150a" />



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
<img width="1920" height="891" alt="3_nslookup" src="https://github.com/user-attachments/assets/dc4a9d13-ab0d-4d21-a762-a18c217e9918" />



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
<img width="1920" height="891" alt="4_headers" src="https://github.com/user-attachments/assets/c03f5f36-1a08-4a62-b95e-7f2bff76618b" />




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
<img width="1920" height="891" alt="5_waf" src="https://github.com/user-attachments/assets/25c01c6c-cce3-4bb4-9054-c93869e78fcc" />



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
| TXT (SPF) | networkwalks.com | `v=spf1 +a +mx +ip4:50.87.144.87 +include:websitewelcome.com ~all` |
| SRV       | \_autodiscover.\_tcp.networkwalks.com | cpanelemaildiscovery.cpanel.net:443                |
| DNSSEC    | —                                     | Reported as unsigned / no answer                   |

**Analysis:**

- The DNS enumeration identified two name servers
-  DNSRecon identified BIND version 9.16.23-RH on both discovered name server IP addresses (192.232.216.131 and 50.87.144.87).
- The website and mail hostnames resolved to the same IP address in the collected results. This does not prove that they share the same physical server.
- The SPF record ends with `~all`, which represents a soft-fail policy.
- No DMARC record was shown in the collected output. A dedicated DMARC lookup is required to confirm its presence or absence.
 - WHOIS reported the domain as unsigned for DNSSEC. DNSRecon returned no answer to its DNSSEC query. An authoritative DNSSEC check is recommended to confirm the domain's DNSSEC status.

**Tool Note:**

The initial dnsrecon execution encountered an error. The tool was upgraded to version 1.6.3, after which the enumeration completed successfully.

**Evidence:**
<img width="1920" height="891" alt="image" src="https://github.com/user-attachments/assets/89d75e5a-be18-4025-a35d-0e14750996c3" />




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

<img width="951" height="517" alt="Screenshot 2026-11-09 131600" src="https://github.com/user-attachments/assets/ace6779b-5abc-4759-8d3a-31cafbb616d6" />


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
| 192.168.1.12 | Windows computer | 135, 139, 445 | 97 closed | Windows scanning machine |

**Important:** Port numbers and Nmap service labels provide initial clues, not definitive identification of the software running on a device.

**Evidence:**
<img width="951" height="517" alt="Screenshot 2026-11-09 131715" src="https://github.com/user-attachments/assets/0bd8bff0-4c07-4e06-86c5-bf51dc8b04c7" />





### 5.3 Host Analysis

The Nmap scan identified four devices with open TCP ports within the scanned port range. The following analysis is based on the recorded scan results. Device roles and service descriptions are preliminary and require further verification.

**192.168.1.1 – Gateway Router**

- TCP ports 53, 80, 139 and 445 were identified as open.
- Port 53 is commonly associated with DNS.
- Port 80 is commonly associated with HTTP.
- Port 139 is commonly associated with NetBIOS Session Service.
- Port 445 is commonly associated with Microsoft-DS/SMB.
- The SMB-associated ports should be investigated to determine why file-sharing services are accessible on the gateway device.

**192.168.1.6 – Huawei Device**

- TCP ports 53, 80 and 443 were identified as open.
- Port 53 is commonly associated with DNS.
- Port 80 is commonly associated with HTTP.
- Port 443 is commonly associated with HTTPS.
- Nmap classified 97 scanned ports as filtered.
- The device may be a router, modem or wireless access point, based on the recorded vendor information. Its exact role has not been confirmed.

**192.168.1.9 – Arcadyan Device**

- TCP ports 3000, 8008, 8009 and 8443 were identified as open.
- These ports are associated with different application and web-service conventions.
- The exact applications running on these ports were not confirmed by the Fast Scan.
- Further service and version detection is recommended to identify the services and determine whether they are required.

**192.168.1.12 – Confirmed Windows Scanning Machine**

- Confirmed as the Windows computer used for the network scan, based on its Wi-Fi IPv4 configuration.
- TCP ports 135, 139 and 445 were identified as open.
- Port 135 is commonly associated with Microsoft RPC.
- Ports 139 and 445 are commonly associated with NetBIOS and SMB.
- Further investigation is recommended to determine whether these services are necessary and appropriately restricted.

**Other Discovered Hosts**

- 192.168.1.2, 192.168.1.3 and 192.168.1.8 responded to host discovery.
- No open ports were identified on these devices within the 100 TCP ports scanned.
- This does not mean that the devices have no services running on other ports or protocols.
### 5.4 Network Topology Analysis

Zenmap was used to visualise the discovered network devices and their scan results.

The topology view helps present the discovered hosts and their apparent network relationships.

However, a topology diagram alone does not prove that every device can communicate directly with every other device or that network segmentation is absent.

**Evidence:**

<img width="960" height="504" alt="Screenshot 2026-11-11 160704" src="https://github.com/user-attachments/assets/6e7051ed-6758-4381-af2c-ed57917fe12e" />


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
| F-05 | WHOIS reported DNSSEC as unsigned; DNSRecon returned no answer to its DNSSEC query | WHOIS / dnsrecon | Requires verification |
| F-06 | WordPress REST API endpoint advertised                                   | curl                | Informational               |
| F-07 | Website and mail hostnames resolve to the same IP                        | nslookup / dnsrecon | Informational               |
| F-08 | Public contact email identified                                          | WhatWeb             | Informational               |
| F-09 | Different name server entries appeared in registrar and registry records | WHOIS               | Requires verification       |
| F-10 | TCP ports 53, 80, 139 and 445 open on confirmed gateway device                            | Nmap                | Requires service validation |
| F-11 | TCP port 80 open on 192.168.1.1 and 192.168.1.6; service not confirmed | Nmap | Requires validation |
| F-12 | Multiple open TCP ports (3000, 8008, 8009 and 8443) on Arcadyan-associated device; services unconfirmed | Nmap                | Requires validation         |
| F-13 | TCP ports 135, 139 and 445 open on confirmed Windows scanning computer (`192.168.1.12`)                | Nmap                | Requires validation         |
| F-14 | Seven responsive devices identified within the scanned subnet            | Nmap                | Informational               |

### Positive Security Observations

- HTTPS access was successful.
- HTTP redirected to HTTPS.
- The website used a Secure and HttpOnly cookie.
- ModSecurity WAF was detected.
- Domain protection statuses were enabled.
- The Apache version was not disclosed in the captured response.
- Three discovered client devices (`192.168.1.2`, `192.168.1.3` and `192.168.1.8`) had no open ports within the 100 common TCP ports scanned.
---

## 7. Recommendations

### 7.1 Website Security Recommendations

1. **Keep software updated:** Regularly update WordPress, plugins and themes. Verify installed versions and check relevant security advisories.

2. **Improve HTTP security headers:** Review the configuration of HSTS, Content-Security-Policy, X-Frame-Options or CSP frame-ancestors, and X-Content-Type-Options.

3. **Review cookie security:** Verify that cookies use appropriate Secure, HttpOnly and SameSite attributes. The captured response showed Secure and HttpOnly attributes; the SameSite configuration should be confirmed.

4. **Strengthen email authentication:** Review SPF configuration, confirm all legitimate sending services before changing the policy to `-all`, and configure DKIM and DMARC.

5. **Review DNSSEC:** Confirm DNSSEC support and enable it where appropriate.

6. **Review DNS information disclosure:** Ask the hosting provider whether unnecessary version banners can be suppressed.

7. **Review WordPress REST API exposure:** Ensure that user information and other sensitive resources are not accessible to unauthorised users.

8. **Verify DNS configuration:** Confirm that registrar and registry name server information is consistent with the intended DNS delegation.

### 7.2 Internal Network Security Recommendations

9. **Review SMB services:** Investigate why ports 139 and 445 are open on the gateway. Identify the services using these ports, disable unnecessary file-sharing services and verify that SMBv1 is not enabled.
10. **Review gateway security:** Verify that gateway administration is restricted to trusted devices, use strong administrator credentials, keep firmware updated and use HTTPS for management where supported.
11. **Review remote administration:** Disable remote management from the internet unless it is required, and restrict access to trusted administrators.
12. **Investigate port 3000:** Identify the application running on 192.168.1.9 and determine whether it is required.

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

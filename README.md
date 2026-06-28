<div align="center">

<h1>Alireza Fazlollahi</h1>

<h3>Offensive Security Engineer</h3>

[![LinkedIn](https://img.shields.io/badge/LinkedIn-arkanzasfeziii-0A66C2?style=flat-square&logo=linkedin&logoColor=white)](https://linkedin.com/in/arkanzasfeziii)
[![Email](https://img.shields.io/badge/Email-contact-D14836?style=flat-square&logo=gmail&logoColor=white)](mailto:alireza.fazlollahi.cybersec@gmail.com)
[![Open to](https://img.shields.io/badge/Open%20to-Roles%20in%20Europe-2ea44f?style=flat-square)](-)

</div>

---

## Why I Build Offensive Frameworks

I'm a penetration tester. I test enterprise networks, cloud environments, and web applications. And I build frameworks because I got tired of the gap between how security tools work and how adversaries actually operate.

Most security tools answer one question: *is this vulnerable?* That's useful, but it's not how attacks work. An attacker doesn't run a scanner, file a report, and go home. They take the SSRF finding from hour one, use it to pull cloud credentials from IMDS, escalate those credentials through misconfigured IAM policies, and by hour four they're downloading production databases from S3.

The problem I kept hitting during engagements was **context loss between attack phases.** You discover a JWT with weak signing in one tool, then manually copy it into another tool to test IDOR. You harvest AWS credentials via SSRF, then switch to a completely different framework for privilege escalation. Every tool switch is a context switch. Every context switch is where findings get lost and attack chains break.

That's the problem these 13 platforms solve. Each one owns a complete attack surface. Each one carries discovered credentials, tokens, and findings from one module to the next through a shared `EngagementSession`. The JWT forged in the auth module is the same token used to probe IDOR endpoints. The credentials harvested from IMDS flow directly into IAM privilege escalation — no clipboard, no glue scripts, no context loss.

---

## Design Decisions

Every framework makes tradeoffs. These are mine and why I made them.

**Python, not Go or Rust.**
Not a performance decision — an adoption decision. Security teams read Python. When a finding needs manual verification during an engagement, the operator shouldn't need to learn a new language to understand the detection logic. The tradeoff is clear: these tools won't outperform compiled scanners on raw throughput. But throughput isn't the bottleneck in a real engagement — context loss is.

**13 separate packages, not a monorepo.**
Each platform targets a different assessment type with different authorization boundaries. A client who authorizes web application testing shouldn't need to install Active Directory attack modules on their network. Separate packages mean separate scopes, separate dependencies, and cleaner authorization documentation.

**`EngagementSession` as the core abstraction.**
Traditional security tools are stateless: scan, output, done. Real engagements are stateful — what you discover in phase one determines what you test in phase two. Every platform is built around a session object (`EngagementSession` or `EngagementContext`) that wraps an HTTP session with retry logic and carries cookies, tokens, discovered credentials, and confirmed findings across all modules. This is what makes attack chaining possible without manual glue.

**Module registry pattern across all platforms.**
Every platform uses the same dispatch: a `MODULE_REGISTRY` dict mapping names to classes, a `BaseModule` ABC, and a CLI that accepts `--modules all` or `--modules ssrf auth`. An operator who learns one platform already knows all thirteen. New attack techniques drop in as modules without touching CLI or reporting code.

---

## Architecture — How the Platforms Connect

These aren't 13 isolated tools. They map to the stages of adversary simulation:

```
Recon ──→ Initial Access ──→ Credential Access ──→ Privilege Escalation ──→ Lateral Movement ──→ Persistence ──→ Exfiltration
```

| Stage | Platforms | What Gets Tested |
|---|---|---|
| **Reconnaissance** | Spectre, Sentinel | OSINT, subdomain enumeration, stack fingerprinting, API endpoint discovery |
| **Initial Access** | Sentinel, Mirage, Aegis | Web/API exploits, phishing simulation, network-level attacks |
| **Credential Access** | Nebula, VaultBreaker, Sentinel | IMDS harvest, database credentials, JWT forge, OAuth bypass, API keys |
| **Privilege Escalation** | Nebula, Sovereign, Kraken | 14 IAM escalation paths, Kerberoast/DCSync, Kubernetes RBAC abuse |
| **Lateral Movement** | Aegis, Nebula, Sovereign | Network pivot, C2 tunneling, cross-account role assumption, AD lateral |
| **Persistence & Evasion** | Ghost, Nebula, BlackForge | AMSI bypass, process injection, IAM backdoors, CI/CD pipeline persistence |
| **Exfiltration** | Nebula, VaultBreaker | S3 keyword-targeted download, database extraction |

**Specialized surfaces** — Tempest (wireless), Pulse (mobile), Forge (physical/hardware) extend coverage to engagement types beyond the standard network kill chain.

**Training** — [SecurityQuestAcademy](https://github.com/arkanzasfeziii/SecurityQuestAcademy) is a separate project: 700 hands-on challenges across 7 domains with an interactive terminal game engine.

---

## Platform Suite

| Platform | Attack Surface | Key Capabilities |
|---|---|---|
| [**Sentinel**](https://github.com/arkanzasfeziii/Sentinel) | Web & API | SSRF → cloud creds · IDOR · JWT/OAuth attacks · SQL/NoSQL/SSTI · GraphQL |
| [**Nebula**](https://github.com/arkanzasfeziii/Nebula) | Cloud (AWS/Azure/GCP) | 14 IAM escalation paths · IMDS harvest · Secrets Manager · IAM backdoors · S3 exfil |
| [**Sovereign**](https://github.com/arkanzasfeziii/Sovereign) | Active Directory | Kerberoast · AS-REP roast · DCSync · ACL abuse · Pass-the-Hash · AD kill chain |
| [**Kraken**](https://github.com/arkanzasfeziii/Kraken) | Kubernetes | RBAC misconfig · secret extraction · container escape · SA token abuse · etcd access |
| [**Aegis**](https://github.com/arkanzasfeziii/Aegis) | Network | SMB/LDAP/DNS/SNMP enum · credential attacks · C2 tunneling · IoT/OT |
| [**Spectre**](https://github.com/arkanzasfeziii/Spectre) | OSINT & Recon | Subdomain enum · email harvest · DNS intel · cert transparency · org footprinting |
| [**Mirage**](https://github.com/arkanzasfeziii/Mirage) | Social Engineering | Phishing simulation · credential harvest · SPF/DKIM/DMARC · MFA bypass |
| [**BlackForge**](https://github.com/arkanzasfeziii/BlackForge) | CI/CD & Supply Chain | GitHub Actions injection · Jenkins RCE · GitLab CI · dependency confusion |
| [**Ghost**](https://github.com/arkanzasfeziii/Ghost) | Evasion & Payloads | AMSI bypass · AV evasion · process injection · LOLBaS · EDR fingerprinting |
| [**VaultBreaker**](https://github.com/arkanzasfeziii/VaultBreaker) | Database | SQLi exploitation · MongoDB/Redis/Elasticsearch unauth · credential extraction |
| [**Pulse**](https://github.com/arkanzasfeziii/Pulse) | Mobile | APK analysis · Frida hooking · SSL pinning bypass · traffic interception |
| [**Tempest**](https://github.com/arkanzasfeziii/Tempest) | Wireless | WPA2 capture · evil twin · PMKID · RADIUS/802.1X exploitation |
| [**Forge**](https://github.com/arkanzasfeziii/Forge) | Physical & Hardware | BadUSB · RFID/NFC cloning · physical recon · lock bypass · hardware implants |
| [**SecurityQuestAcademy**](https://github.com/arkanzasfeziii/SecurityQuestAcademy) | Training | 7 quest games · 700 challenges · Python/Bash/Windows/Cisco/Crypto/RE/Web |

---

## Technology

![Python](https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white)
![Bash](https://img.shields.io/badge/Bash-4EAA25?style=flat-square&logo=gnubash&logoColor=white)
![Kali Linux](https://img.shields.io/badge/Kali-557C94?style=flat-square&logo=kalilinux&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-232F3E?style=flat-square&logo=amazonaws&logoColor=white)
![Azure](https://img.shields.io/badge/Azure-0078D4?style=flat-square&logo=microsoftazure&logoColor=white)
![GCP](https://img.shields.io/badge/GCP-4285F4?style=flat-square&logo=googlecloud&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white)
![MITRE](https://img.shields.io/badge/MITRE_ATT%26CK-CC0000?style=flat-square)

---

## Current Work

- Hardening the platform suite — integration tests, engagement demos, and benchmarks against established tools
- Writing an offensive security methodology book in Persian — structured adversary simulation for the regional security community
- Researching cloud attack path chaining across IAM, compute, and storage layers

Open to offensive security and red team engineering roles in **Germany and Europe**.

---

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0A66C2?style=flat-square&logo=linkedin&logoColor=white)](https://linkedin.com/in/arkanzasfeziii)
[![Email](https://img.shields.io/badge/Email-alireza.fazlollahi.cybersec%40gmail.com-D14836?style=flat-square&logo=gmail&logoColor=white)](mailto:alireza.fazlollahi.cybersec@gmail.com)

---

<div align="center">

*Frameworks, not scripts. Attack chains, not isolated findings.*

</div>

# Linux Vulnerability Scanner — Complete VAPT Report

> **Project:** Linux Vulnerability Scanner  
> **Assessment Type:** Vulnerability Assessment and Penetration Testing (VAPT)  
> **Assessment Environment:** Kali Linux laboratory environment  
> **Prepared By:** Chandraprakash G  
> **Role:** Cybersecurity Enthusiast  
> **Repository:** https://github.com/mike-rk/vuln-scanner  
> **Assessment Status:** Completed  
> **Scope:** Authorized laboratory environment only

---

## 1. Executive Summary

This report presents the complete security assessment of a custom **Linux Vulnerability Scanner** developed in Python. The project is designed as a minimal, offline, deterministic vulnerability scanner for Docker containers and small Linux hosts.

The scanner performs local package inventory and configuration-security checks, compares installed package versions against an offline signature database, identifies security-sensitive configurations, and produces structured JSON reports. The repository also supports baseline comparison and ignore-list functionality.

The assessment was conducted in a controlled Kali Linux laboratory environment across six technical phases:

1. Project Setup and Architecture
2. Scanner Development and Target Discovery
3. Vulnerability Detection and JSON Reporting
4. Vulnerability Validation and Testing
5. Remediation and Re-scanning
6. Advanced Detection and Signature Validation

During the documented scan, the scanner reported:

| Severity | Findings |
|---|---:|
| Critical | 1 |
| High | 2 |
| Medium | 20 |
| Low | 0 |
| Informational | 0 |
| **Total** | **23** |

The most significant finding was **Docker socket exposure**, which was validated through controlled, non-destructive testing. The assessment user was subsequently removed from the Docker group and the resulting security state was verified.

The `SSH_AUTH_SOCK` and `XAUTHORITY` findings were investigated and treated as **context-dependent / potential false positives**, rather than automatically declaring them exploitable. SUID findings were similarly reviewed as security-sensitive configurations requiring contextual analysis.

The assessment did not perform destructive exploitation, persistence, data destruction, or unauthorized access.

---

## 2. Project Overview

The project is a Python-based, local/offline vulnerability scanner. The repository describes it as a minimal scanner for Docker containers and small Linux hosts with deterministic JSON output and no network calls during scan execution.

### Core capabilities

- Local Linux/Docker filesystem scanning
- Debian `dpkg` and Alpine `apk` package inventory support
- Offline package-version/CVE signature matching
- Docker socket exposure detection
- Root execution detection
- World-writable directory detection
- SUID binary detection
- Secret-like environment-variable detection
- JSON reporting
- Baseline comparison
- Ignore/suppression support
- Configurable offline signature database
- CI-friendly exit codes

### Repository architecture

```text
vuln-scanner/
├── src/
│   ├── scanner.py
│   ├── reporter.py
│   ├── checks/
│   │   ├── packages.py
│   │   └── config.py
│   └── db/
│       └── signatures.json
├── Dockerfile
├── requirements.txt
├── scan.json
├── packages.py
├── reporter.py
├── scanner.py
└── README.md
```

The repository's documented architecture identifies `scanner.py` as the orchestration/CLI component, `reporter.py` as the JSON/baseline/ignore reporting component, `packages.py` as the package inventory and version-check component, `config.py` as the configuration-check component, and `signatures.json` as the offline signature database.

---

# 3. Assessment Objectives

The assessment objectives were:

1. Prepare a reproducible Linux security-testing environment.
2. Verify the scanner's runtime requirements.
3. Verify network and Docker prerequisites.
4. Understand and validate the scanner architecture.
5. Execute vulnerability scans against the authorized Linux environment.
6. Generate and inspect structured JSON findings.
7. Validate important findings using safe manual checks.
8. Analyze potential false positives.
9. Remediate the highest-priority confirmed exposure.
10. Re-scan after remediation.
11. Validate the offline signature database.
12. Produce evidence suitable for technical documentation and GitHub publication.

---

# 4. Scope

## 4.1 In Scope

- Kali Linux assessment host
- Local Linux filesystem
- Installed package metadata
- Docker daemon/socket configuration
- Docker group membership
- SUID binaries
- Selected environment variables
- Scanner source and configuration
- Offline vulnerability signature database
- JSON scan reports
- Remediation and post-remediation verification

## 4.2 Out of Scope

- Unauthorized external systems
- Production infrastructure
- Destructive exploitation
- Persistence mechanisms
- Credential theft
- Data destruction
- Denial-of-service testing
- Attacks against systems without explicit authorization

---

# 5. Tools and Technologies

| Category | Technology |
|---|---|
| Operating System | Kali Linux |
| Programming Language | Python |
| Python Version | 3.13.12 in assessment environment |
| Package Manager | pip |
| Container Platform | Docker |
| Scanner Output | JSON |
| Vulnerability DB | Offline `signatures.json` |
| Package Detection | `dpkg` / `apk` logic |
| Network Validation | `ip`, `ping` |
| File/Permission Analysis | `find`, `ls`, `dpkg -S` |
| Source Repository | GitHub |

---

# 6. Assessment Methodology

The assessment followed a practical VAPT lifecycle:

```text
Environment Preparation
        ↓
Architecture & Target Identification
        ↓
Automated Vulnerability Detection
        ↓
Manual Validation
        ↓
False-Positive Analysis
        ↓
Remediation
        ↓
Post-Remediation Re-scan
        ↓
Signature Database Validation
        ↓
Final Documentation
```

The methodology prioritised evidence-based validation. Scanner output was treated as an indicator requiring verification rather than automatically treating every detection as an exploitable vulnerability.

---

# 7. Phase 1 — Project Setup and Architecture

## Objective

Prepare the assessment environment and verify the required software components.

### 7.1 Kali Linux Environment

The Kali Linux release and kernel were verified using `/etc/os-release` and `uname -a`.

**Evidence:** `phase1-step1-kali-environment.png`

![Kali environment](evidence/phase1-step1-kali-environment.png)

### 7.2 Python and pip

Python and pip versions were verified before creating the project environment.

**Evidence:** `phase1-step2-python-pip.png`

![Python and pip](evidence/phase1-step2-python-pip.png)

### 7.3 Docker

Docker installation and service state were verified. The Docker daemon was active and running.

**Evidence:** `phase1-step4-docker.png`

![Docker](evidence/phase1-step4-docker.png)

### 7.4 Python Virtual Environment

A dedicated Python virtual environment was created and activated for the project.

**Evidence:** `phase1-step7-python-venv.png`

![Virtual environment](evidence/phase1-step7-python-venv.png)

### Phase 1 Result

The assessment environment was successfully prepared and the project was executed inside an isolated Python environment.

**Status: COMPLETE**

---

# 8. Phase 2 — Scanner Development and Target Discovery

## Objective

Verify the assessment host's network configuration, connectivity, and scanner project structure.

### 8.1 Network Interface Discovery

The `ip -br addr` output identified the assessment interfaces.

The captured environment included:

- Loopback: `127.0.0.1/8`
- Ethernet interface: `192.168.56.104/24`
- Docker bridge: `172.17.0.1/16`

**Evidence:** `phase2-step1-network.png`

![Network configuration](evidence/phase2-step1-network.png)

### 8.2 Connectivity Testing

Connectivity was verified against `1.1.1.1` and `google.com`. The captured tests showed successful ICMP replies with 0% packet loss.

**Evidence:** `phase2-step2-connectivity.png`

![Connectivity](evidence/phase2-step2-connectivity.png)

### 8.3 Project Structure

The project structure was reviewed to identify scanner, reporter, checks, signature database, Dockerfile, and supporting files.

**Evidence:** `phase2-step3-project-structure.png`

![Project structure](evidence/phase2-step3-project-structure.png)

### Phase 2 Result

The assessment environment and scanner architecture were successfully identified.

**Status: COMPLETE**

---

# 9. Phase 3 — Vulnerability Detection and JSON Reporting

## Objective

Verify scanner functionality, execute scans, identify security findings, and generate structured JSON reports.

## 9.1 Scanner CLI Verification

The scanner help output confirmed the supported options:

- `--target`
- `--output`
- `--format`
- `--ignore`
- `--baseline`
- `--show-suppressed`
- `--log-level`
- `--sig-db`

**Evidence:** `phase3-step1-scanner-help.png`

![Scanner CLI](evidence/phase3-step1-scanner-help.png)

## 9.2 Scanner Execution

The scanner was executed against the authorized Linux target.

**Evidence:** `phase3-step2-scanner-execution.png`

![Scanner execution](evidence/phase3-step2-scanner-execution.png)

## 9.3 JSON Report

The generated JSON report contained 23 findings.

**Evidence:** `phase3-step3-scan-json.png`

![JSON report](evidence/phase3-step3-scan-json.png)

### Severity distribution

```text
CRITICAL  █ 1
HIGH      ██ 2
MEDIUM    ████████████████████ 20
LOW       0
INFO      0
```

## 9.4 Finding Details

The report structure includes fields such as:

- Check name
- Affected path/resource
- Severity
- Finding details
- Remediation

**Evidence:** `phase3-step4-json-findings.png`

![Detailed findings](evidence/phase3-step4-json-findings.png)

## 9.5 Scanner Warning/Detection Output

The scanner logs showed detection of Docker socket exposure, environment-variable findings, and SUID binaries.

**Evidence:** `phase3-step5-scanner-output.png`

![Scanner output](evidence/phase3-step5-scanner-output.png)

### Phase 3 Result

The scanner successfully performed automated vulnerability detection and generated structured JSON evidence.

**Status: COMPLETE**

---

# 10. Phase 4 — Vulnerability Validation and Testing

## Objective

Manually validate important scanner findings and determine whether they represented confirmed security exposures or context-dependent detections.

---

## 10.1 Finding F-01 — Docker Socket Exposure

**Severity:** CRITICAL  
**Check:** `docker_socket_exposed`  
**Resource:** `/var/run/docker.sock`

The scanner identified Docker socket exposure.

Controlled validation demonstrated that Docker access was available to the assessment user and that Docker commands could be executed without `sudo` while the user remained in the Docker group.

The validation was performed safely without modifying the host filesystem.

**Evidence:** `phase4-step4.2-critical-validation.png`

![Docker validation](evidence/phase4-step4.2-critical-validation.png)

### Impact

Access to the Docker daemon is highly privileged. Depending on daemon configuration and accessible operations, Docker access can result in significant container-management capabilities and potential host-level impact.

### Validation Status

**CONFIRMED**

---

## 10.2 Finding F-02 — SSH_AUTH_SOCK

**Severity:** HIGH  
**Check:** `secret_in_env`

The scanner identified `SSH_AUTH_SOCK` as a secret-like environment variable.

Manual verification confirmed that the variable existed and pointed to an SSH agent socket.

**Evidence:** `phase4-step4.4-false-positive-analysis.png`

![Environment variables](evidence/phase4-step4.4-false-positive-analysis.png)

### Assessment

The presence of `SSH_AUTH_SOCK` alone does not establish that a credential or secret has been exposed to an attacker. Its security impact depends on filesystem permissions, process isolation, and whether an untrusted process can access the agent socket.

### Validation Status

**CONTEXT-DEPENDENT / POTENTIAL FALSE POSITIVE**

---

## 10.3 Finding F-03 — XAUTHORITY

**Severity:** HIGH  
**Check:** `secret_in_env`

`XAUTHORITY` was also detected.

Manual verification confirmed the variable existed and referenced the user's Xauthority file.

### Assessment

The presence of the variable is not, by itself, proof of credential compromise. The impact depends on permissions and access by untrusted processes.

### Validation Status

**CONTEXT-DEPENDENT / POTENTIAL FALSE POSITIVE**

---

## 10.4 Finding F-04 — SUID Binaries

**Severity:** MEDIUM  
**Check:** `suid_binary`

A SUID inventory was performed using:

```bash
find /usr/bin /usr/sbin -perm -4000 -type f -ls 2>/dev/null
```

**Evidence:** `phase4-step4.3-suid-validation.png`

![SUID inventory](evidence/phase4-step4.3-suid-validation.png)

Selected binaries were reviewed using `ls -l`.

**Evidence:** `phase4-step4.5-controlled-validation.png`

### Assessment

SUID is a security-sensitive privilege mechanism, but the presence of a SUID bit does not automatically mean the binary is vulnerable. Standard system binaries may legitimately require SUID.

Package ownership was checked for selected binaries using `dpkg -S`.

### Validation Status

**CONFIGURATION CONFIRMED; EXPLOITABILITY REQUIRES CONTEXT**

---

## 10.5 False-Positive Analysis

The scanner findings were not accepted blindly.

The following were explicitly reviewed:

| Finding | Scanner Severity | Assessment |
|---|---|---|
| Docker socket | Critical | Confirmed |
| SSH_AUTH_SOCK | High | Context-dependent |
| XAUTHORITY | High | Context-dependent |
| SUID binaries | Medium | Configuration confirmed; context required |

**Evidence:** `phase4-step4.6-validation-documentation.png`

![Validation documentation](evidence/phase4-step4.6-validation-documentation.png)

### Phase 4 Result

The critical Docker exposure was confirmed. Other findings were reviewed and appropriately qualified rather than overstated.

**Status: COMPLETE**

---

# 11. Phase 5 — Remediation and Re-scanning

## Objective

Remediate the confirmed Docker access exposure, verify the change, review remaining findings, and perform a final scan.

---

## 11.1 Docker Group Remediation

The assessment user was removed from the Docker group using:

```bash
sudo gpasswd -d "$USER" docker
```

**Evidence:** `phase5-step3-docker-remediation.png`

![Docker remediation](evidence/phase5-step3-docker-remediation.png)

The command confirmed removal of the user from the group.

---

## 11.2 Group Verification

The Docker group was checked using:

```bash
getent group docker
```

The captured output showed:

```text
docker:x:967:
```

indicating that the assessment user was no longer listed in the group.

**Evidence:** `phase5-step4.1-group-verification.png`

![Group verification](evidence/phase5-step4.1-group-verification.png)

---

## 11.3 Session/Access Verification

The user's group state was reviewed after the remediation process.

**Evidence:** `phase5-step4-group-state.png`

![Group state](evidence/phase5-step4-group-state.png)

The remediation was intended to remove direct Docker-group access from the user's effective session.

---

## 11.4 Environment Review

The environment variables were reviewed again for contextual assessment.

**Evidence:** `phase5-step5-environment-review.png`

![Environment review](evidence/phase5-step5-environment-review.png)

---

## 11.5 SUID Review

The scanner's SUID findings were reviewed rather than blindly removing SUID permissions.

**Evidence:** `phase5-step8-suid-review.png`

![SUID findings](evidence/phase5-step8-suid-review.png)

Selected permissions were verified:

**Evidence:** `phase5-step9-suid-permissions.png`

![SUID permissions](evidence/phase5-step9-suid-permissions.png)

### Security decision

No indiscriminate SUID removal was performed because removing SUID from legitimate system utilities can break normal system functionality.

---

## 11.6 Final Re-scan

A post-remediation scan was executed against the same Linux target scope:

```bash
python3 scanner.py --target / --output phase5-final-scan.json --format json --log-level info
```

The scan completed successfully and generated `phase5-final-scan.json`.

**Evidence:** `phase5-step10-final-rescan.png`

![Final re-scan](evidence/phase5-step10-final-rescan.png)

### Important reporting note

An earlier comparison between a scan of `.` and a scan of `/` was not treated as a valid before/after metric because the target scopes were different. The final report therefore does **not** claim a numerical reduction based on that invalid comparison.

### Phase 5 Result

The highest-priority confirmed Docker access issue was remediated and the post-remediation state was re-evaluated.

**Status: COMPLETE**

---

# 12. Phase 6 — Advanced Detection and Signature Validation

## Objective

Review, test, and validate the scanner's offline vulnerability signature database.

---

## 12.1 Package Signatures

The signature database contains package-version rules for packages including:

- OpenSSL
- libssl1.1
- curl
- bash
- zlib1g

**Evidence:** `phase6-step6.1-signature-database.png`

![Package signatures](evidence/phase6-step6.1-signature-database.png)

---

## 12.2 Configuration Signatures

The database also contains configuration-security signatures including:

- `docker_socket_exposed`
- `running_as_root`
- `world_writable_dir`
- `suid_binary`
- `secret_in_env`
- `no_seccomp_profile`

**Evidence:** `phase6-step6.2-config-signatures.png`

![Configuration signatures](evidence/phase6-step6.2-config-signatures.png)

---

## 12.3 Signature Database Loading

An initial Python loading attempt produced a syntax/variable error.

**Evidence:** `phase6-step6.2-loading-attempt.png`

![Initial loading attempt](evidence/phase6-step6.2-loading-attempt.png)

This screenshot is retained for development history but is **not considered successful validation evidence**.

The database was subsequently loaded and used successfully by the scanner.

---

## 12.4 Signature-Based Scan

The scanner was executed with the signature database and generated findings.

**Evidence:** `phase6-step6.3-signature-scan.png`

![Signature scan](evidence/phase6-step6.3-signature-scan.png)

---

## 12.5 Package Signature Verification

The package-related entries were verified within `signatures.json`.

**Evidence:** `phase6-step6.4-package-signatures.png`

![Package signature verification](evidence/phase6-step6.4-package-signatures.png)

---

## 12.6 Configuration Signature Verification

All documented configuration/security signature names were verified.

**Evidence:** `phase6-step6.5-config-signatures.png`

![Configuration signature verification](evidence/phase6-step6.5-config-signatures.png)

---

## 12.7 JSON Syntax Validation

The final signature database validation returned:

```text
Signature DB JSON: VALID
```

**Evidence:** `phase6-step6.6-json-validation.png`

![JSON validation](evidence/phase6-step6.6-json-validation.png)

### Phase 6 Result

The offline signature database was reviewed, exercised during scanning, checked for expected entries, and validated as syntactically valid JSON.

**Status: COMPLETE**

---

# 13. Detailed Findings Register

## F-01 — Docker Socket Exposure

| Attribute | Value |
|---|---|
| ID | F-01 |
| Severity | Critical |
| Check | `docker_socket_exposed` |
| Resource | `/var/run/docker.sock` |
| Status | Confirmed |
| Remediation | Applied |
| Validation | Controlled Docker access testing |

### Description

The Docker daemon socket was accessible in the assessed environment.

### Security Impact

Docker daemon access is highly privileged and can provide substantial control over containers and, depending on configuration and accessible operations, can create a path toward host-level impact.

### Recommendation

- Minimize Docker group membership.
- Do not mount the Docker socket into untrusted containers.
- Restrict daemon access.
- Consider rootless Docker where appropriate.
- Apply least privilege to container workloads.

---

## F-02 — SSH_AUTH_SOCK Detected

| Attribute | Value |
|---|---|
| ID | F-02 |
| Severity | High |
| Check | `secret_in_env` |
| Variable | `SSH_AUTH_SOCK` |
| Status | Context-dependent |

### Description

The scanner classifies secret-like environment variable names as potential security findings.

### Assessment

`SSH_AUTH_SOCK` is normally an SSH agent socket reference. Its presence is not equivalent to exposing an SSH private key.

### Recommendation

- Restrict socket permissions.
- Avoid sharing the SSH agent with untrusted containers/processes.
- Use agent forwarding carefully.
- Improve scanner logic to distinguish secret values from legitimate socket paths.

---

## F-03 — XAUTHORITY Detected

| Attribute | Value |
|---|---|
| ID | F-03 |
| Severity | High |
| Check | `secret_in_env` |
| Variable | `XAUTHORITY` |
| Status | Context-dependent |

### Description

The scanner detected an environment variable referencing X11 authentication data.

### Assessment

The variable's presence does not automatically demonstrate an exploitable authentication weakness.

### Recommendation

- Restrict access to the Xauthority file.
- Avoid exposing X11 authentication material to untrusted containers/processes.
- Improve scanner classification of environment variables.

---

## F-04 — SUID Binaries

| Attribute | Value |
|---|---|
| ID | F-04 |
| Severity | Medium |
| Check | `suid_binary` |
| Status | Configuration confirmed |
| Exploitability | Requires individual binary/context analysis |

### Description

Multiple SUID binaries were identified.

### Assessment

SUID is an intentional Linux privilege mechanism used by legitimate system utilities. A SUID finding should therefore be treated as an attack-surface indicator, not automatic proof of vulnerability.

### Recommendation

- Maintain an approved SUID inventory.
- Investigate unexpected SUID files.
- Keep privileged packages updated.
- Remove SUID only where demonstrably unnecessary.

---

# 14. Risk Summary

| Risk Area | Severity | Business/Technical Impact | Priority |
|---|---|---|---|
| Docker daemon/socket access | Critical | Potential highly privileged container/host control | Immediate |
| SSH agent socket classification | High | Potential access if untrusted processes can reach socket | Review |
| Xauthority classification | High | Potential X11 authentication exposure depending on permissions | Review |
| SUID configuration | Medium | Privilege-sensitive attack surface | Review |
| Other scanner findings | Medium | Configuration/privilege exposure depending on context | Review |

---

# 15. Remediation Summary

| Finding | Action | Result |
|---|---|---|
| Docker socket access | Removed user from Docker group | Access restriction applied |
| SSH_AUTH_SOCK | Contextual review | Not automatically classified as exploitable |
| XAUTHORITY | Contextual review | Not automatically classified as exploitable |
| SUID binaries | Inventory and package ownership review | No unsafe blanket removal |
| Signature DB | Validation and testing | JSON validated successfully |

---

# 16. Security Recommendations

## High Priority

1. Restrict Docker daemon access to only trusted administrators/processes.
2. Avoid exposing `/var/run/docker.sock` to untrusted containers.
3. Review Docker group membership regularly.
4. Maintain least-privilege access to container-management interfaces.

## Medium Priority

5. Review SUID binaries periodically.
6. Maintain an approved baseline of privileged binaries.
7. Patch packages associated with vulnerability signatures.
8. Review environment-variable findings based on actual permissions and process access.

## Scanner Improvements

9. Distinguish secret values from legitimate socket/path environment variables.
10. Add package-version parsing support for package versions that currently produce parsing warnings.
11. Expand signature coverage using trusted and pinned vulnerability intelligence.
12. Add richer finding metadata such as CWE/CVSS when reliable mappings are available.
13. Add unit tests for each check.
14. Add regression tests for false-positive scenarios.
15. Add CI testing for scanner exit codes and JSON schema validation.

---

# 17. Limitations

This assessment has several limitations:

- The project is a custom laboratory vulnerability scanner.
- Detection quality depends on the available package metadata and signature database.
- SUID detection identifies privileged configuration, not automatic exploitability.
- Environment-variable detection is heuristic.
- Package-version matching depends on successful version parsing.
- No destructive exploitation was performed.
- No unauthorized systems were tested.
- The scan was performed against a controlled assessment environment.
- A quantitative remediation comparison should only be made between equivalent target scopes and equivalent scan conditions.
- The offline signature database is only as current as the entries maintained in it.

---

# 18. Evidence Index

| Evidence ID | Phase | Evidence |
|---|---|---|
| E-01 | 1 | Kali environment |
| E-02 | 1 | Python/pip |
| E-03 | 1 | Docker |
| E-04 | 1 | Python virtual environment |
| E-05 | 2 | Network interfaces |
| E-06 | 2 | Connectivity |
| E-07 | 2 | Project structure |
| E-08 | 3 | Scanner CLI |
| E-09 | 3 | Scanner execution |
| E-10 | 3 | JSON report |
| E-11 | 3 | Detailed findings |
| E-12 | 3 | Scanner output |
| E-13 | 4 | Critical Docker validation |
| E-14 | 4 | SUID validation |
| E-15 | 4 | False-positive analysis |
| E-16 | 4 | Validation documentation |
| E-17 | 5 | Docker remediation |
| E-18 | 5 | Group verification |
| E-19 | 5 | Group state |
| E-20 | 5 | Environment review |
| E-21 | 5 | SUID review |
| E-22 | 5 | SUID permission verification |
| E-23 | 5 | Final re-scan |
| E-24 | 6 | Package signatures |
| E-25 | 6 | Configuration signatures |
| E-26 | 6 | Initial loading attempt |
| E-27 | 6 | Signature-based scan |
| E-28 | 6 | Package signature verification |
| E-29 | 6 | Configuration signature verification |
| E-30 | 6 | JSON validation |

---

# 19. Conclusion

The Linux Vulnerability Scanner project successfully demonstrated an end-to-end vulnerability assessment workflow.

The assessment covered:

- Linux security assessment
- Python security-tool development
- Package vulnerability detection
- Configuration analysis
- Docker security
- SUID analysis
- Environment-variable analysis
- Manual validation
- False-positive analysis
- Remediation
- Re-scanning
- Offline signature management
- JSON reporting
- Security documentation

The **Docker socket exposure** represented the highest-priority confirmed security issue and was successfully validated and followed by access remediation.

The assessment also demonstrated an important VAPT principle: **scanner output must be validated before a finding is reported as an exploitable vulnerability**. This was particularly relevant to the environment-variable and SUID detections.

Overall, the project provides a practical demonstration of vulnerability-management concepts from discovery through remediation and verification.

---

# 20. GitHub Publication Structure

Recommended repository documentation structure:

```text
vuln-scanner/
├── README.md
├── VAPT-REPORT.md
├── src/
│   ├── scanner.py
│   ├── reporter.py
│   ├── checks/
│   │   ├── packages.py
│   │   └── config.py
│   └── db/
│       └── signatures.json
├── reports/
│   ├── scan.json
│   ├── baseline.json
│   └── phase5-final-scan.json
└── docs/
    └── evidence/
        ├── phase1-*.png
        ├── phase2-*.png
        ├── phase3-*.png
        ├── phase4-*.png
        ├── phase5-*.png
        └── phase6-*.png
```

## Recommended GitHub report title

**Linux Vulnerability Scanner — Vulnerability Assessment & Penetration Testing Report**

---

## 21. Assessment Declaration

This report documents security testing performed in an authorized laboratory environment for educational, development, and cybersecurity portfolio purposes.

**Prepared by:**  
**Chandraprakash G**  
Cybersecurity Enthusiast

# Detection Catalogue and Response Playbooks

## How to read this catalogue

Each proposed sensor is documented as a detection hypothesis rather than a guaranteed attack detector.

- **Signal:** the observable condition.
- **Hypothesis:** the suspicious or harmful behaviour the signal may help identify.
- **Required context:** the data needed to decide whether the alert matters.
- **False positives:** legitimate activity that can resemble the signal.
- **Initial action:** the first safe investigation steps.
- **Validation status:** whether operational testing evidence exists.

All thresholds are starting points from the academic project and require environment-specific validation.

---

## D-01 — HTTP response-time and availability anomaly

### Detection objective

Surface abnormal web-service degradation quickly enough to investigate availability, capacity, application, dependency, network, or attack-related causes.

### Original threshold hypothesis

- Warning when response time exceeds **3 seconds**
- Alert when response time exceeds **5 seconds**

### Required telemetry

- request timestamp and duration;
- HTTP method, host, path category, and status code;
- request and error rate;
- client/source information available at the edge;
- web server, load balancer, CDN, WAF, and application health;
- DNS-resolution timing and authoritative DNS change logs;
- CPU, memory, connection, worker, and dependency saturation;
- deployment and maintenance events.

### What the signal may indicate

- application or database slowdown;
- dependency failure;
- resource exhaustion;
- abnormal request volume;
- denial-of-service conditions;
- DNS or routing problems; or
- a failed deployment.

### What it cannot prove

Response time alone cannot establish that a DDoS attack or DNS hijacking occurred. Those conclusions require corroborating traffic, DNS, edge, configuration, and infrastructure evidence.

### Common benign explanations

- planned deployment;
- backup or maintenance activity;
- slow downstream dependency;
- traffic growth;
- synthetic-monitor location issues;
- network congestion;
- cache expiry.

### Initial response

1. Confirm whether the alert is isolated or sustained.
2. Compare latency with request rate, error rate, saturation, and dependency health.
3. Check recent releases and maintenance.
4. Review edge/WAF/network data for unusual sources or request patterns.
5. Review DNS configuration and change records if redirection is suspected.
6. Escalate based on service impact and corroborating evidence.

### Recommended tuning

Use rolling baselines, service-level objectives, endpoint classes, and sustained-duration conditions. A static five-second threshold may be useful for a small lab but is not sufficient for all applications.

---

## D-02 — MySQL activity anomaly

### Detection objective

Identify database activity that may represent misuse, application malfunction, unauthorized bulk access, or a performance condition requiring investigation.

### Original threshold hypothesis

- Query duration above **3 seconds**
- Query volume above **1.5 times normal**

### Required telemetry

- authenticated database user and application identity;
- source host/service;
- query fingerprint or normalized statement;
- target database/table;
- query duration, rows examined, rows returned, and bytes transferred;
- authentication and privilege changes;
- database errors;
- application and WAF events;
- approved jobs, reports, backups, and maintenance windows.

### What the signal may indicate

- expensive or abnormal queries;
- new bulk access;
- failed application behaviour;
- account misuse;
- unauthorized reporting or export;
- attempted exploitation when correlated with application-layer evidence.

### What it cannot prove

Slow or numerous queries alone cannot prove SQL injection or data exfiltration. SQL injection prevention should rely primarily on secure query construction—especially parameterized queries—plus least privilege and testing.

### Common benign explanations

- scheduled reporting;
- schema migration;
- index maintenance;
- backup;
- batch processing;
- newly deployed inefficient query;
- seasonal workload.

### Initial response

1. Identify the account, application, source, and affected tables.
2. Compare activity with the user's role and approved job schedules.
3. Review normalized query patterns and application/WAF events.
4. Determine whether sensitive data was accessed or exported.
5. Check for privilege changes or account compromise.
6. Preserve relevant database and application logs before remediation.

### Recommended preventive controls

- parameterized queries;
- least-privilege database accounts;
- separate identities by application or service;
- secure configuration and patching;
- sensitive-data access logging;
- controlled administrative access.

---

## D-03 — SSH authentication anomaly

### Detection objective

Surface brute-force attempts, password spraying, use of compromised credentials, or unusual remote administration.

### Original threshold hypothesis

- More than **5 failed authentication attempts**
- Successful login from an unknown IP address

### Required telemetry

- timestamp, source IP, destination host, username, authentication method, and outcome;
- VPN, proxy, NAT, and bastion context;
- asset role and owner;
- approved administrative sources;
- account privilege;
- geolocation and threat-intelligence context, when appropriate;
- session commands, file transfer, and privilege elevation where authorized and logged;
- maintenance and change records.

### What the signal may indicate

- brute-force attack;
- password spraying;
- compromised administrator account;
- unauthorized remote access;
- reconnaissance against exposed SSH.

### Common benign explanations

- mistyped credentials;
- automation with an expired key;
- administrator on a new approved network;
- VPN egress change;
- monitoring or configuration tool;
- lab or penetration-test activity.

### Initial response

1. Confirm whether the source is approved and whether the destination should accept SSH.
2. Review failures by source, username, and destination over time.
3. Validate the successful session with the account owner or change record.
4. Review subsequent commands, transfers, privilege escalation, and related host activity.
5. Restrict or revoke access when evidence justifies containment.
6. Search for the same source/account across other systems.

### Recommended controls

- keys or stronger authentication appropriate to the environment;
- MFA through an approved access gateway where feasible;
- restricted source networks or bastion access;
- disabled direct root login;
- rate limiting;
- short-lived or managed credentials;
- prompt revocation and log review.

---

## D-04 — Endpoint-security control health

### Detection objective

Identify endpoints whose security control has been disabled, degraded, outdated, or tampered with.

### Original threshold hypothesis

- Protection disabled for more than **5 minutes**
- Definitions or protection content more than **7 days** out of date

### Required telemetry

- agent heartbeat and health state;
- real-time protection status;
- content/engine/platform version;
- policy and exclusion changes;
- tamper-protection events;
- endpoint identity, owner, criticality, and network state;
- EDR detections and process telemetry;
- approved maintenance and troubleshooting.

### What the signal may indicate

- control failure;
- failed update;
- unsupported device;
- deliberate tampering;
- administrative troubleshooting;
- a device that has stopped reporting.

### What it cannot prove

A disabled or outdated control does not establish that malware is present. Conversely, a healthy control does not prove that the endpoint is uncompromised.

### Initial response

1. Confirm device identity, owner, criticality, and last contact.
2. Check whether maintenance or troubleshooting explains the state.
3. Review policy, exclusion, service, and process changes.
4. Review recent detections and related identity/network activity.
5. Restore protection or isolate based on risk and evidence.
6. Determine whether the issue affects multiple devices.

---

## D-05 — Critical-file integrity anomaly

### Detection objective

Identify unexpected changes to critical system, application, configuration, or security files.

### Original threshold hypothesis

- **2 or more** critical-file changes within **10 minutes**

### Required telemetry

- file path, operation, timestamp, prior/new hash, user, process, parent process, and host;
- approved change, patch, deployment, and configuration-management records;
- file criticality and expected change frequency;
- script, registry, service, and process telemetry;
- endpoint and network context.

### What the signal may indicate

- unauthorized configuration change;
- tampering;
- persistence;
- destructive activity;
- unexpected deployment;
- compromised administrative tooling.

### What it cannot prove

A file-integrity sensor does not directly detect memory-only or “fileless” malware. Detection of those behaviours needs process, script, command-line, registry, memory, and network telemetry.

### Common benign explanations

- patching;
- deployment;
- configuration management;
- antivirus quarantine;
- administrator change;
- log rotation or application-generated updates if the watch list is poorly selected.

### Initial response

1. Identify the changed files, hashes, process, and account.
2. Compare with authorized changes.
3. Assess whether the file is executable, security-sensitive, or part of persistence.
4. Review related processes, scripts, services, registry changes, and network activity.
5. Preserve evidence before restoring a known-good version.
6. Expand the search to systems with the same change.

### Recommended tuning

Use a curated critical-file list, approved-change enrichment, path-specific severity, hash allowlists, and process/user context. A simple count without context will generate noise.

---

## D-06 — Windows event and service anomaly

### Detection objective

Surface suspicious connection patterns, service manipulation, repeated service failures, or other event sequences that merit host investigation.

### Original threshold hypothesis

- **5 or more** connection attempts to a suspicious IP
- **3 or more** failed service-start attempts within **5 minutes**

### Required telemetry

- Windows event provider and event ID;
- host, user, process, parent process, service name, binary path, and result;
- source/destination IP, port, protocol, and direction;
- endpoint detection telemetry;
- approved software installation, service management, and maintenance records;
- asset role and criticality;
- DNS/proxy/firewall context.

### What the signal may indicate

- malicious or unwanted outbound communication;
- repeated exploitation or persistence attempts;
- service tampering;
- failed malware installation;
- broken deployment or configuration.

### Common benign explanations

- unstable legitimate service;
- application update;
- administrator troubleshooting;
- blocked dependency;
- stale threat-intelligence label;
- security scanner;
- endpoint-management tool.

### Initial response

1. Validate the event IDs, service, account, process, and host role.
2. Confirm the IP classification and determine whether contact succeeded.
3. Review the surrounding event sequence rather than one event in isolation.
4. Compare with software deployment and maintenance records.
5. Inspect related process, file, registry, authentication, and network activity.
6. Escalate when multiple sources support unauthorized behaviour.

---

## D-07 — Bandwidth and network-connection anomaly

### Detection objective

Surface abnormal transfer volume or connection behaviour that may represent exfiltration, scanning, malware communication, misuse, or service degradation.

### Original threshold hypothesis

- Traffic above **2 times normal**
- **500–1,000** attempts to non-standard ports

### Required telemetry

- source and destination IP/port, protocol, direction, bytes, packets, duration, and action;
- asset identity, role, owner, and criticality;
- DNS, proxy, firewall, VPN, and cloud flow logs;
- process/user context where available;
- approved backup, replication, update, scanning, and transfer schedules;
- historical baseline by asset, service, time, and direction.

### What the signal may indicate

- bulk data transfer;
- network scanning;
- malware or command-and-control traffic;
- misconfiguration;
- backup or replication failure;
- unexpected application behaviour.

### What it cannot prove

High bandwidth is not synonymous with exfiltration, and “non-standard port” is not automatically malicious. Attackers may use standard ports, while legitimate applications may use uncommon ports.

### Initial response

1. Identify the systems, owner, process or service, destination, and transfer direction.
2. Compare with the asset's normal behaviour and approved schedules.
3. Review DNS, proxy, firewall, endpoint, and identity evidence.
4. Determine the type and sensitivity of any transferred data.
5. Check for scanning patterns across many destinations or ports.
6. Block or isolate only when evidence and impact justify the action.

### Recommended tuning

Create baselines per asset class and time window. Use peer groups, transfer direction, destination reputation, data sensitivity, rare destinations, denied/allowed ratios, and connection fan-out rather than a single global threshold.

---

## Correlation opportunities

Single-source alerts become more useful when combined. Examples:

- HTTP latency + request-rate spike + edge saturation + many sources
- Database volume anomaly + unusual account + sensitive table + large outbound transfer
- SSH success from new source + privilege elevation + file change + new outbound connection
- Endpoint-control disabled + policy change + suspicious process + alert-source silence
- Critical-file change + new service + encoded command line + external connection
- Bandwidth spike + rare destination + endpoint process + identity anomaly

Correlation should improve confidence without hiding the underlying evidence.

## Severity model

| Severity | Suggested condition |
|---|---|
| Informational | Data-quality or low-confidence signal with no current impact |
| Low | Isolated anomaly on a low-criticality asset with a plausible benign explanation |
| Medium | Repeated or correlated anomaly requiring timely investigation |
| High | Strong evidence of unauthorized activity, sensitive exposure, or material service risk |
| Critical | Confirmed or highly credible active compromise with severe business, safety, or data impact |

Severity should combine confidence, asset criticality, scope, impact, and urgency—not threshold magnitude alone.

## Validation status

The available project artifacts do not provide evidence of live deployment or measured true-positive/false-positive performance. Every entry is therefore marked **design complete; operational validation required**.

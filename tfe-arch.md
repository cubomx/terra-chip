# TFE Components
- App Layer
    - **TFE Core**: Rails app for web & background workers
    - **TFE Services & Terra workers**: Go services (isolated)
- Coordination Layer
    - **Redis**: caching & coord bet web & background workers in the app layer
    - **RabbitMQ**: TF worker coord
- Storage Layer
    - **PostgreSql DB**: app data for workspace/user settings
    - **Blob Storage**: state files, logs
    - **Vault**: Encrypts
    - **Config Data**: replicated admin console config.
- Network Service - NGINX


## Operational Mode Decision
- Demo: all data on the instance.
- **Prod - Mounted Disk**: performant block storage. Store in a separate directory on the host. 
External disk.
- **Production - External Services**: majority of data in external DB & external object storage. 
Managed services.

## Arch Components
- **Compute**: replicated container, sys config & app config
- **External PostgreSQL DB**: secure app config data...
- **S3 kind/blob**: artifacts (plan results, state, logs)
- **External Vault Instance** (if used): encryption of PostgreSQL data. Cluster.

### Compute Requirements
TFE Instance:
- 2 CPU (4/8 core), 16-32 GB RAM, 50 GB storage
- Linux

PostgreSQL:
- 2 CPU (4/8 core), 16-32 GB RAM, 50 GB storage
- Version: 9.4, 9.5, 9.6, 10.x, 11.x

### Object Storage
S3, Google Cloud Storage, Azure Blob Storage, Minio or Ceph. HA.

## Encrypted
- Vault Transit: VCS Data, Plan Result, State, Logs, Env Vars, 2F4 Recovery Codes, SSH Keys,
OAuth Client ID = Secret, OAuth User Tokens, Twilio Account Config, SAML Config, SMTP Config
- bcrypt: Account pass
- HMAC SHA512: User/Team/Org tokens
- Vault Unseal Key: ChaCha20+Poly1305
- no: org settings.

## Failure Tolerance
TFE App Servers:
-  2 VMS in diff clusters/zones -> HA & Reliability
- LB

PostgreSQL: cluster
Object Storage: HA, use that config.
Vault Servers: supports HA cluster model.

## TFE Installation
**Replicated** container-based plat for deploying cloud-native apps. License management, online & airgap
installation, control panel.

Checklist:
- Type: airgapped, offline, online?
- A proxy?
- TLS config. CA chain of trust (many orgs, dozens)
    - Certificate Authorities: root CAs & intermediate CAs. An authority trusted by the device. 
    Identity & secure comm.
- Custome TFE Worker Image: preload, creation pipeline, storage, installation
- Operational Mode.

## Net requirements
- User/Client/VCS
    - 443: access the TFE app via HTTPS
- Admins
    - 22: access instance via SSH.
    - 8800: installer dashboard
- TFE Servers:
    - 9870 - 9880 (range): internal comm on the host, not public
    - 23000 - 23100 (range): intern comm ...

### Proxy
- Config can be update via the **Replicated console**
- Also Docker
- Edit on the host if running installer or validation to pass

### TLS config
Required a TLS certificate & private key (PEM-encoded). Match TFE hostname (FQDN, wildcard certificate). Public or
private CA, trusted by all: VCS Provider, CI Systems, Notifications sys TFE, build sys TFE.

3 options:
- Self signed
- Server path
- Upload Files

TFE runs in Docker containers.

## Custom Worker
- base image `ubuntu:xenial`
- artifact
- exist on the TFE host
- CA certificates
- no Terra on the iamge. TFE provides at runtime.
- script: initialize (before `terraform init`) or finalize (after `plan` or `apply`)

## Install 
Online or Airgapped (Docker pre-installed, required artifacts at `.airgap` file with Replicated,
providers must be provided as `bundles` after install).
- Should be able to acces via HTTPS.
- Internal CA to issue bootstrap certificates
- Remainde of install via dashboard (port 8800)
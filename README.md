## OpenSSL Certificate Generation & CA Template

A comprehensive, production-ready tool for managing your own Certificate Authority (CA) and generating SSL/TLS certificates for services, servers, and client authentication. This tool simplifies the complex process of certificate generation while maintaining security best practices and providing flexibility for various use cases.

**Key Features:**
- Create and manage your own Certificate Authority
- Generate server, client, and combined certificates
- Automatic keystore (PKCS12) and truststore (JKS) generation for Java applications
- Smart validation of certificate extensions with minimum requirements
- Template generation for quick setup
- Certificate revocation and cleanup management
- Configurable defaults via config file or environment variables
- Organized directory structure with automatic CSR archival

---

**Table of Contents:**

1. [Why Use This Tool?](#why-use-this-tool)
   - [When Should You Use Your Own CA?](#when-should-you-use-your-own-ca)
   - [Why Not Use Public Certificate Authorities?](#why-not-use-public-certificate-authorities)
   - [What This Tool Provides](#what-this-tool-provides)
2. [Getting Started](#getting-started)
   - [Prerequisites](#prerequisites)
   - [Installation](#installation)
   - [Quick Start](#quick-start)
3. [Usage](#usage)
   - [Setting Up Your Certificate Authority](#setting-up-your-certificate-authority)
   - [Understanding Certificate Distinguished Names](#understanding-certificate-distinguished-names)
   - [Generating Templates](#generating-templates)
   - [Generating Service Certificates](#generating-service-certificates)
   - [Certificate Types](#certificate-types)
   - [Useful Flags](#useful-flags)
   - [Using Generated Certificates](#using-generated-certificates)
4. [What You Can Do](#what-you-can-do)
   - [Secure Microservices Communication](#secure-microservices-communication)
   - [Kafka Cluster with SSL/TLS](#kafka-cluster-with-ssltls)
   - [Database Server Security](#database-server-security)
   - [Web Server SSL/TLS](#web-server-ssltls)
   - [Container Orchestration](#container-orchestration)
   - [VPN and Remote Access](#vpn-and-remote-access)
   - [IoT Device Authentication](#iot-device-authentication)
   - [Message Queue Security](#message-queue-security)
   - [Development and Testing](#development-and-testing)
   - [Monitoring Stack Security](#monitoring-stack-security)
5. [Configuration](#configuration)
   - [Default Configuration File](#default-configuration-file)
   - [Environment Variables](#environment-variables)
   - [Extension File Structure](#extension-file-structure)
6. [Manual Certificate Operations](#manual-certificate-operations)
   - [Generating a Root CA Manually](#generating-a-root-ca-manually)
   - [Generate Server Certificate Manually](#generate-server-certificate-manually)
   - [Generate Client Certificate Manually](#generate-client-certificate-manually)
   - [Creating Keystores Manually](#creating-keystores-manually)
   - [Creating Truststores Manually](#creating-truststores-manually)
   - [Verifying Certificates and Keystores](#verifying-certificates-and-keystores)
   - [Common keytool Operations](#common-keytool-operations)
7. [Application Integration](#application-integration)
   - [Java Applications](#java-applications)
   - [Apache Kafka](#apache-kafka)
   - [PostgreSQL](#postgresql)
   - [Nginx](#nginx)
   - [Docker Registry](#docker-registry)
   - [MongoDB](#mongodb)
   - [Redis](#redis)
   - [Grafana](#grafana)
   - [Prometheus](#prometheus)
8. [Troubleshooting](#troubleshooting)

---

### Why Use This Tool?

#### When Should You Use Your Own CA?

Operating your own Certificate Authority is ideal for:

**Development and Testing Environments:**
- Creating certificates for local development without purchasing commercial certificates
- Testing SSL/TLS configurations before production deployment
- Simulating production certificate infrastructure in staging environments

**Internal Infrastructure:**
- Securing communication between internal microservices
- Implementing mutual TLS (mTLS) for service-to-service authentication
- Encrypting internal databases, message queues, and other infrastructure components
- Securing internal web applications and APIs that don't face the public internet

**IoT and Embedded Systems:**
- Managing certificates for large fleets of IoT devices
- Implementing device authentication and secure communication
- Creating hierarchical certificate structures for device management

**Private Networks:**
- Securing communication in air-gapped or isolated networks
- Enterprise internal applications where public CAs aren't practical
- VPN and private network authentication

#### Why Not Use Public Certificate Authorities?

While public CAs (Let's Encrypt, DigiCert, etc.) are excellent for public-facing websites, they have limitations:

- **Control**: You don't control the root of trust
- **Internal Use**: Public CAs won't issue certificates for internal domains like `*.internal.local`
- **Renewal Complexity**: Managing renewals for hundreds of internal services can be cumbersome
- **Network Requirements**: Public CAs require domain validation, which doesn't work for isolated networks

#### What This Tool Provides

This tool automates the entire certificate lifecycle while maintaining security best practices:

1. **Simplified CA Management**: Set up your CA once, issue unlimited certificates
2. **Consistent Configuration**: Ensures all certificates follow your organization's standards
3. **Automation**: Reduces manual errors in certificate generation
4. **Flexibility**: Supports server, client, and combined (mTLS) certificates
5. **Integration Ready**: Automatically generates keystores and truststores for Java applications
6. **Validation**: Ensures certificates meet minimum security requirements
7. **Organization**: Maintains clean directory structure with automatic archival

---

### Getting Started

#### Prerequisites

Before using this certificate generation tool, you need to have the following software installed:

**1. OpenSSL** - Required for all certificate operations

Check if installed:
```bash
openssl version
```

Installation:
```bash
# Ubuntu/Debian
sudo apt update
sudo apt install openssl

# macOS
brew install openssl

# Windows (WSL)
sudo apt update
sudo apt install openssl
```

**2. Java Development Kit (JDK)** - Required for keytool (keystores/truststores)

Check if installed:
```bash
keytool -version
java -version
```

Installation:
```bash
# Ubuntu/Debian
sudo apt update
sudo apt install default-jdk
# OR for just the JRE (smaller)
sudo apt install default-jre

# macOS
brew install openjdk

# Windows (WSL)
sudo apt update
sudo apt install default-jdk
```

**3. Optional: tree** - For better directory visualization
```bash
# Ubuntu/Debian
sudo apt install tree

# macOS
brew install tree
```

#### Installation

There are two ways to install the certificate generation script:

**Option 1: Install from GitHub Releases (Recommended)**

1. Download the latest release:

   Visit the [releases page](https://github.com/parkermmr/ca-template/releases) and download the latest version, or use `curl`:

   ```bash
   # Download latest release
   curl -LO https://github.com/parkermmr/ca-template/releases/latest/download/generate-certs

   # Make it executable
   chmod +x generate-certs
   ```

2. Install to local bin directory:

   ```bash
   # Create local bin directory if it doesn't exist
   mkdir -p ~/.local/bin

   # Move the script
   mv generate-certs ~/.local/bin/

   # Verify installation
   which generate-certs
   ```

3. Add to PATH (if not already):

   Add this line to your shell configuration file (`~/.bashrc`, `~/.zshrc`, or `~/.profile`):

   ```bash
   export PATH="$HOME/.local/bin:$PATH"
   ```

   Then reload your shell configuration:

   ```bash
   # For bash
   source ~/.bashrc

   # For zsh
   source ~/.zshrc

   # Or simply restart your terminal
   ```

4. Verify the installation:

   ```bash
   generate-certs --version
   ```

   Expected output:
   ```
   Certificate Generation Script v1.0.0
   ```

**Option 2: Manual Installation from Source**

1. Clone the repository:

   ```bash
   git clone https://github.com/parkermmr/ca-template.git
   cd ca-template
   ```

2. Install the script:

   ```bash
   # Create local bin directory if it doesn't exist
   mkdir -p ~/.local/bin

   # Copy the script
   cp bin/generate-certs ~/.local/bin/

   # Make it executable
   chmod +x ~/.local/bin/generate-certs
   ```

3. Add to PATH (if not already):

   Add this line to your shell configuration file (`~/.bashrc`, `~/.zshrc`, or `~/.profile`):

   ```bash
   export PATH="$HOME/.local/bin:$PATH"
   ```

   Then reload your shell configuration:

   ```bash
   # For bash
   source ~/.bashrc

   # For zsh
   source ~/.zshrc

   # Or simply restart your terminal
   ```

4. Verify the installation:

   ```bash
   generate-certs --version
   ```

#### Quick Start

Once installed, set up your Certificate Authority:

```bash
# Initialize the CA (first time only)
generate-certs --setup-ca
```

You'll be prompted to enter details for your root CA certificate. After setup, your CA will be located at `~/CA/`.

Example output:
```
==========================================
Certificate Authority Setup
==========================================

[INFO] Creating CA directory structure...
[OK] Directory structure created

[INFO] Initializing CA database...
[OK] Database initialized

[INFO] Creating default configuration file...
[OK] Configuration file created at: /home/user/CA/.generate-certs.conf

[INFO] Creating OpenSSL configuration...
[OK] Configuration file created

[INFO] Generating Root CA certificate...

You will be prompted for CA details:

Country Name (2 letter code) [AU]:
Organization Name [Community]:
Organizational Community [OpenSource]:
Organizational Unit [Cloud]:
Organizational Type [Self Signed]:
Organizational Entity (eg, People, Services) [Services]:
Common Name (e.g. server FQDN or YOUR name) []:Development Root CA
Email Address []:

[OK] Root CA certificate generated

==========================================
CA Setup Complete!
==========================================

CA Directory: /home/user/CA

Root CA Certificate:
-------------------
subject=C = AU, O = Community, OU = OpenSource, OU = Cloud, OU = Self Signed, OU = Services, CN = Development Root CA
issuer=C = AU, O = Community, OU = OpenSource, OU = Cloud, OU = Self Signed, OU = Services, CN = Development Root CA
notBefore=Feb  7 01:33:20 2026 GMT
notAfter=Feb  5 01:33:20 2036 GMT
```

**Next steps:**
1. Edit `~/CA/.generate-certs.conf` to customize default certificate values
2. Generate certificates for your services (see [Usage](#usage) section)

---

### Usage

#### Setting Up Your Certificate Authority

Before generating any certificates, you must first set up your Certificate Authority:

```bash
generate-certs --setup-ca
```

This will prompt you for CA details such as:
- Country Name (2 letter code)
- Organization Name
- Organizational Unit
- Common Name (e.g., "My Organization Root CA")
- Email Address

The CA will be created at `~/CA/` with the following structure:
```
~/CA/
├── root-ca.cert              # CA public certificate (distribute to clients)
├── private/root-ca.key       # CA private key (KEEP SECURE!)
├── openssl-ca.cnf            # CA configuration
├── .generate-cerst.conf      # Certificate generation defaults
├── index.txt                 # Certificate database
├── serial.txt                # Serial number counter
├── certs/                    # Issued certificates (.pem files)
├── csr/                      # Archived certificate requests
└── crl/                      # Certificate revocation lists
```

#### Understanding Certificate Distinguished Names

When generating certificates, the Distinguished Name (DN) format differs between person and service certificates:

**Person Certificates:**
- Format: `CN={name-input}`
- Example: `CN=Firtname Lastname`
- Full DN: `/C=AU/O=Community/OU=OpenSource/OU=Cloud/OU=Self Signed/OU=Services/CN=Firtname Lastname`

**Service Certificates (Single Instance):**
- Format: `CN=svc-{service-name}`
- Example: `CN=svc-grafana`
- Full DN: `/C=AU/O=Community/OU=OpenSource/OU=Cloud/OU=Self Signed/OU=Services/CN=svc-grafana`

**Service Certificates (Multiple Instances):**
- Format: `CN=svc-{service-name}-{instance-number}`
- Example: `CN=svc-kafka-0`, `CN=svc-kafka-1`, `CN=svc-kafka-2`
- Full DN: `/C=AU/O=Community/OU=OpenSource/OU=Cloud/OU=Self Signed/OU=Services/CN=svc-kafka-0`

You can verify the DN of any certificate with:
```bash
openssl x509 -in {certificate-file}.crt -text -noout
```

#### Generating Templates

Before generating certificates, create template `.ext` files with proper structure:

```bash
# {service-name} - The name of your service (e.g., kafka, grafana, nginx)
# {instance-count} - Number of instances to generate (default: 1)
# {cert-type} - Certificate type: server, client, or combined

# Generate templates for multiple server instances (combined certs)
generate-certs --service {service-name} --instances {instance-count} --type combined --template

# Generate template for single server
generate-certs --service {service-name} --type server --template

# Templates are optional for client certificates
generate-certs --person {full-name} --type client --template
```

Example:
```bash
# Generate templates for 3 Kafka brokers
generate-certs --service kafka --instances 3 --type combined --template
```

Expected output:
```
[INFO] Generating template files for: kafka

[OK] Created: kafka-0.ext
[OK] Created: kafka-1.ext
[OK] Created: kafka-2.ext
[OK] Template files generated in: /home/user/.secrets/kafka

Next steps:
1. Edit the .ext files to add your DNS names, IPs, etc.
2. Run the certificate generation command without --template
```

After running `--template`, edit the generated `.ext` files to add your DNS names, IP addresses, etc.

#### Generating Service Certificates

**Single Server Certificate:**

```bash
# {service-name} - Name of the service (e.g., grafana, nginx, prometheus)
# {cert-type} - Type of certificate: server, client, or combined

# 1. Generate template
generate-certs --service {service-name} --type {cert-type} --template

# 2. Edit ~/.secrets/{service-name}/{service-name}.ext to add your DNS/IPs

# 3. Generate certificate
generate-certs --service {service-name} --type {cert-type}
```

Example:
```bash
# Generate Grafana server certificate
generate-certs --service grafana --type server --template
# Edit ~/.secrets/grafana/grafana.ext
generate-certs --service grafana --type server
```

**Multiple Server Instances:**

```bash
# {service-name} - Name of the service
# {instance-count} - Number of instances
# {cert-type} - Certificate type

# 1. Generate templates for multiple brokers
generate-certs --service {service-name} --instances {instance-count} --type {cert-type} --template

# 2. Edit each .ext file

# 3. Generate all certificates
generate-certs --service {service-name} --instances {instance-count} --type {cert-type} --keystore --truststore --cleanup
```

Example:
```bash
# Generate 3 Kafka broker certificates
generate-certs --service kafka --instances 3 --type combined --template
# Edit kafka-0.ext, kafka-1.ext, kafka-2.ext
generate-certs --service kafka --instances 3 --type combined --keystore --truststore --cleanup
```

Expected output:
```
[INFO] Validating CA setup...
[OK] CA validation passed
[INFO] Validating extension files...
[OK] Extension file validation passed
[WARN] Cleaning up existing certificates for kafka...
[INFO] Starting certificate generation...

[INFO] Generating certificate: kafka-0
[OK] Generated private key and CSR
[INFO] Signing certificate with CA...
[OK] Signed certificate
[OK] Archived CSR: kafka-0-20260207-113349.csr
[INFO] Creating PKCS12 keystore...
[OK] Created keystore: kafka-0-keystore.p12
[INFO] Creating JKS truststore...
[OK] Created truststore: kafka-0-truststore.jks
[OK] Certificate generation complete: kafka-0

[INFO] Generating certificate: kafka-1
...

==========================================
Certificate Generation Summary
==========================================

Service: kafka
Type: combined
Subject Type: service
Instances: 3
Output: /home/user/.secrets/kafka

Passwords saved to: /home/user/.secrets/kafka/passwords.env
```

**Client Certificate:**

```bash
# {person-name} - Full name of the person (e.g., "Firtname Lastname", etc.)
# Client certificates don't require .ext files
generate-certs --person "{person-name}" --type client --keystore
```

Example:
```bash
generate-certs --person "Firtname Lastname" --type client --keystore
```

Expected output:
```
[INFO] Validating CA setup...
[OK] CA validation passed
[INFO] Validating extension files...
[INFO] Client certificates: .ext files are optional
[INFO] No .ext file found, will use default client_cert extension
[OK] Extension file validation passed
[INFO] Starting certificate generation...

[INFO] No .ext file found for client cert, using default client_cert extension
[INFO] Generating certificate: Firtname Lastname
[OK] Generated private key and CSR
[INFO] Signing certificate with CA...
[OK] Signed certificate
[OK] Archived CSR: Firtname-Lastname-20260207-113349.csr
[INFO] Creating PKCS12 keystore...
[OK] Created keystore: Firtname-Lastname-keystore.p12
[OK] Certificate generation complete: Firtname Lastname
```

**Custom Output Directory:**

```bash
# {service-name} - Name of the service
# {output-directory} - Custom path for certificates (e.g., /opt/certs, /etc/ssl/myapp)
generate-certs --service {service-name} --type server --output {output-directory} --keystore --truststore
```

Example:
```bash
generate-certs --service prometheus --type server --output /opt/certs --keystore --truststore
```

**All Instances in Single Directory:**

```bash
# {service-name} - Name of the service
# {instance-count} - Number of instances
# By default, each instance gets its own directory (kafka-0/, kafka-1/, kafka-2/)
# Use --single-dir to put all files in one directory
generate-certs --service {service-name} --instances {instance-count} --type combined --keystore --truststore --single-dir
```

**With PEM Bundle:**

```bash
# {service-name} - Name of the service
# PEM bundle combines: private key + certificate + CA certificate in one file
# Useful for applications like Nginx, MongoDB that accept PEM bundles
generate-certs --service {service-name} --type server --pem-bundle
```

Example:
```bash
generate-certs --service nginx --type server --pem-bundle
```

#### Certificate Types

**Server Certificate (`--type server`):**
- Used by servers to prove their identity to clients
- Requires minimum 2 DNS names in `.ext` file
- Includes `serverAuth` extension
- Use for: Web servers, API servers, databases
- Example DN: `CN=svc-nginx`

**Client Certificate (`--type client`):**
- Used by clients to authenticate to servers
- `.ext` file is optional
- Includes `clientAuth` extension
- Use for: User authentication, service accounts, API clients
- Example DN: `CN=Firtname Lastname` (person) or `CN=svc-app-client` (service)

**Combined Certificate (`--type combined`):**
- Can function as both server and client certificate
- Requires minimum 2 DNS names in `.ext` file
- Includes both `serverAuth` and `clientAuth` extensions
- Use for: Microservices, Kafka brokers, mutual TLS scenarios
- Example DN: `CN=svc-kafka-0`

#### Useful Flags

**`--cleanup`** - Revoke and delete existing certificates before generating new ones:
```bash
# Automatically revokes old certificates from CA database and removes old files
generate-certs --service {service-name} --instances 3 --keystore --truststore --cleanup
```

**`--no-archive`** - Don't archive CSR files:
```bash
# By default, CSR files are archived in ~/CA/csr/ with timestamps
# Use this flag to skip archiving
generate-certs --service {service-name} --no-archive
```

**`--keystore-pass`** - Use specific keystore password instead of auto-generated:
```bash
# {keystore-password} - Password for the PKCS12 keystore (recommended: strong, random password)
# If not provided, a cryptographically secure random password is generated
generate-certs --service {service-name} --keystore --keystore-pass {keystore-password}
```

**`--truststore-pass`** - Use specific truststore password:
```bash
# {truststore-password} - Password for the JKS truststore
# If not provided, a cryptographically secure random password is generated
generate-certs --service {service-name} --truststore --truststore-pass {truststore-password}
```

#### Using Generated Certificates

After generation, your certificates will be organized in directories:

**Single instance:**
```
~/.secrets/{service-name}/
├── {service-name}.key                    # Private key (keep secure!)
├── {service-name}.csr                    # Certificate signing request
├── {service-name}.crt                    # Signed certificate
├── {service-name}-keystore.p12           # PKCS12 keystore (if --keystore)
├── {service-name}-truststore.jks         # JKS truststore (if --truststore)
├── {service-name}-bundle.pem             # PEM bundle (if --pem-bundle)
├── root-ca.cert                          # CA certificate
└── passwords.env                         # Generated passwords
```

**Multiple instances (with --instances 3):**
```
~/.secrets/{service-name}/
├── {service-name}-0/
│   ├── {service-name}-0.key
│   ├── {service-name}-0.crt
│   ├── {service-name}-0-keystore.p12
│   └── {service-name}-0-truststore.jks
├── {service-name}-1/
│   └── ...
├── {service-name}-2/
│   └── ...
└── passwords.env                         # Shared passwords file
```

Load passwords for use:
```bash
# Source the passwords file to load them into environment variables
source ~/.secrets/{service-name}/passwords.env

# Access specific password
echo $KEYSTORE_PASSWORD_{service_name}_0
```

---

### What You Can Do

This tool enables a wide range of certificate-based security implementations:

#### Secure Microservices Communication

Implement mutual TLS (mTLS) between microservices to ensure both client and server authentication:

```bash
# Generate certificates for each service
generate-certs --service users --type combined --keystore --truststore --template
generate-certs --service payments --type combined --keystore --truststore --template
generate-certs --service orders --type combined --keystore --truststore --template
```

Each service can now authenticate to others using its client certificate while also accepting connections as a server.

#### Kafka Cluster with SSL/TLS

Secure a Kafka cluster with SSL encryption and client authentication:

```bash
# Generate certificates for 3 Kafka brokers
generate-certs --service kafka --instances 3 --type combined --keystore --truststore --template --cleanup

# Generate client certificates for producers/consumers
generate-certs --person producer-app --type client --keystore
generate-certs --person consumer-app --type client --keystore
```

Configure Kafka brokers with the keystores for their identity and truststore to validate clients.

#### Database Server Security

Secure database connections (PostgreSQL, MySQL, MongoDB):

```bash
# PostgreSQL server certificate (no keystore needed - PostgreSQL uses PEM files)
generate-certs --service postgres --type server --pem-bundle --template

# Client certificates for database users
generate-certs --person db-admin --type client
generate-certs --person application-user --type client
```

#### Web Server SSL/TLS

Create certificates for internal web servers (Nginx, Apache, Grafana):

```bash
# Grafana (no keystore needed - uses .crt and .key files)
generate-certs --service grafana --type server --template

# Nginx reverse proxy (uses PEM bundle)
generate-certs --service nginx --type server --pem-bundle --template

# Internal API server
generate-certs --service api-internal --type server --template
```

#### Container Orchestration

Secure Docker Registry or Kubernetes components:

```bash
# Docker Registry (uses .crt and .key files)
generate-certs --service docker-registry --type server --template

# Kubernetes API server
generate-certs --service kubernetes-api --type server --template

# etcd cluster
generate-certs --service etcd --instances 3 --type combined --template
```

#### VPN and Remote Access

Create certificates for VPN servers and clients:

```bash
# VPN server (uses PEM bundle)
generate-certs --service vpn-server --type server --pem-bundle --template

# VPN clients (PEM bundles for easy distribution)
generate-certs --person employee-laptop --type client --pem-bundle
generate-certs --person contractor-device --type client --pem-bundle
```

#### IoT Device Authentication

Generate certificates for IoT devices and gateways:

```bash
# IoT gateway
generate-certs --service iot-gateway --type combined --template

# Individual devices (can generate in bulk)
for i in {1..100}; do
    generate-certs --person "iot-device-$i" --type client --pem-bundle
done
```

#### Message Queue Security

Secure RabbitMQ, Redis, or other message queues:

```bash
# RabbitMQ cluster (Java-based, needs keystores)
generate-certs --service rabbitmq --instances 3 --type combined --keystore --truststore --template

# Redis with TLS (uses PEM files)
generate-certs --service redis --type server --pem-bundle --template
```

#### Development and Testing

Create certificates for local development environments:

```bash
# Local development services
generate-certs --service localhost --type server --template
# Edit localhost.ext to add: DNS.1 = localhost, DNS.2 = *.localhost, IP.1 = 127.0.0.1

generate-certs --service localhost --type server
```

#### Monitoring Stack Security

Secure Prometheus, Grafana, and exporters:

```bash
# Prometheus server (Go-based, uses .crt and .key files)
generate-certs --service prometheus --type combined --template

# Node exporters (multiple hosts, use PEM files)
generate-certs --service node-exporter --instances 5 --type server --template

# Grafana (uses .crt and .key files)
generate-certs --service grafana --type server --template
```

---

### Configuration

#### Default Configuration File

After running `--setup-ca`, edit `~/CA/.generate-certs.conf` to set defaults:

```bash
# Certificate Generation Configuration

# Certificate Subject Defaults
CERT_COUNTRY="AU"
CERT_ORGANIZATION="My Company"
CERT_OU_COMMUNITY="Engineering"
CERT_OU_UNIT="Infrastructure"
CERT_OU_TYPE="Internal"
CERT_OU_ENTITY="Services"

# Output Defaults
DEFAULT_OUTPUT_DIR="$HOME/.secrets"

# Certificate Defaults
DEFAULT_CERT_TYPE="combined"
DEFAULT_KEY_SIZE="4096"
DEFAULT_VALIDITY_DAYS="1000"
```

#### Environment Variables

Override configuration on a per-command basis using environment variables:

```bash
# Use different organization for this certificate
CERT_ORGANIZATION="Partner Company" generate-certs --service partner-api --type server

# Change default output location
DEFAULT_OUTPUT_DIR="/opt/certificates" generate-certs --service webapp --type server
```

#### Extension File Structure

Extension files (`.ext`) define Subject Alternative Names (SANs) and must match the certificate type requirements:

**Server Certificate (.ext) - Minimum 2 DNS names required:**
```ini
basicConstraints                   = CA:FALSE
nsCertType                         = server
nsComment                          = "OpenSSL Generated Server Certificate"
subjectKeyIdentifier               = hash
authorityKeyIdentifier             = keyid,issuer:always
keyUsage                           = critical, digitalSignature, keyEncipherment
extendedKeyUsage                   = serverAuth
subjectAltName                     = @alt_names

[alt_names]
DNS.1 = grafana.internal.local      # Required
DNS.2 = grafana.example.com         # Required
DNS.3 = monitoring.example.com      # Optional
IP.1  = 192.168.1.100               # Optional
```

**Client Certificate (.ext) - Optional:**
```ini
basicConstraints                   = CA:FALSE
nsCertType                         = client, email
nsComment                          = "OpenSSL Generated Client Certificate"
subjectKeyIdentifier               = hash
authorityKeyIdentifier             = keyid,issuer:always
keyUsage                           = critical, nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage                   = clientAuth, emailProtection
subjectAltName                     = @alt_names

[alt_names]
email.1 = john.doe@example.com     # Optional
DNS.1   = workstation.internal     # Optional
```

**Combined Certificate (.ext) - Minimum 2 DNS names required:**
```ini
basicConstraints                   = CA:FALSE
nsCertType                         = client, server, email
nsComment                          = "OpenSSL Generated Client+Server Certificate"
subjectKeyIdentifier               = hash
authorityKeyIdentifier             = keyid,issuer:always
keyUsage                           = critical, digitalSignature, keyEncipherment, nonRepudiation
extendedKeyUsage                   = serverAuth, clientAuth, emailProtection
subjectAltName                     = @alt_names

[alt_names]
DNS.1 = kafka-0.internal.local     # Required
DNS.2 = kafka.example.com          # Required
IP.1  = 192.168.1.101              # Optional
```

---

### Manual Certificate Operations

For advanced users who want to understand the underlying OpenSSL commands or perform operations manually. This section explains what each operation does and when you might need it.

#### Generating a Root CA Manually

**What it does:** Creates a self-signed root certificate that will act as your Certificate Authority. This is the trust anchor for all certificates you issue.

**When to use:** Only when setting up a new CA from scratch, or when you want complete manual control over the CA creation process.

```bash
# Generate CA private key and self-signed certificate
# -x509: Output a self-signed certificate instead of a certificate request
# -newkey rsa:4096: Generate a new 4096-bit RSA private key
# -sha256: Use SHA-256 for the signature
# -nodes: Don't encrypt the private key (no DES encryption)
# -days 3650: Certificate valid for 10 years
openssl req -x509 -config ~/CA/openssl-ca.cnf \
  -newkey rsa:4096 -sha256 -nodes \
  -out root-ca.cert \
  -keyout root-ca.key \
  -days 3650

# Set proper permissions
chmod 600 root-ca.key   # Private key: owner read/write only
chmod 644 root-ca.cert  # Public cert: readable by all
```

#### Generate Server Certificate Manually

**What it does:** Creates a server certificate for applications that need to prove their identity to clients (e.g., web servers, database servers).

**When to use:** When you need a certificate for a server application and want to understand or customize each step of the process. Most applications need server certificates.

```bash
# Step 1: Create server private key and Certificate Signing Request (CSR)
# -new: Generate a new certificate request
# -newkey rsa:4096: Generate a 4096-bit RSA key
# -nodes: Don't encrypt the private key
# -keyout: Where to save the private key
# -out: Where to save the CSR
# -subj: Set the Distinguished Name without prompting
openssl req -new -newkey rsa:4096 -nodes \
  -keyout server.key \
  -out server.csr \
  -config ~/CA/openssl-ca.cnf \
  -subj "/C=AU/O=Community/OU=Services/CN=svc-server"

# Step 2: Create extension file (server.ext)
# This defines the certificate's capabilities and alternate names
# SANs (Subject Alternative Names) are required for modern browsers
cat > server.ext << 'EOF'
basicConstraints = CA:FALSE
keyUsage = digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1 = server.example.com
DNS.2 = server.internal.local
IP.1  = 192.168.1.100
EOF

# Step 3: Sign the server CSR with CA
# This creates the actual certificate by signing the CSR
# -config: CA configuration file
# -policy: Policy to use when signing
# -extensions: Certificate extensions to apply
# -extfile: External file containing additional extensions
openssl ca -config ~/CA/openssl-ca.cnf \
  -policy signing_policy \
  -extensions server_cert \
  -extfile server.ext \
  -out server.crt \
  -infiles server.csr
```

#### Generate Client Certificate Manually

**What it does:** Creates a client certificate for authenticating clients to servers (e.g., users, applications).

**When to use:** When implementing mutual TLS where clients need to prove their identity to the server, or for user authentication. Not all applications need client certificates.

```bash
# Step 1: Create client private key and CSR
# Same process as server, but with different DN and purpose
openssl req -new -newkey rsa:4096 -nodes \
  -keyout client.key \
  -out client.csr \
  -config ~/CA/openssl-ca.cnf \
  -subj "/C=AU/O=Community/OU=People/CN=Firtname Lastname"

# Step 2: Sign the client CSR with CA
# No .ext file needed for basic client certs
# The client_cert extension from openssl-ca.cnf is used
openssl ca -config ~/CA/openssl-ca.cnf \
  -policy signing_policy \
  -extensions client_cert \
  -out client.crt \
  -infiles client.csr
```

#### Creating Keystores Manually

**What it does:** Packages the private key, certificate, and CA chain into a single encrypted file (keystore).

**When to use:** ONLY for Java-based applications (Kafka, Spring Boot, Tomcat, etc.). Most non-Java applications (Nginx, PostgreSQL, Prometheus, Grafana) do NOT need keystores and use separate .crt and .key files instead.

**Convert certificate to PKCS12 keystore:**

```bash
# Create PKCS12 keystore
# PKCS12 is the standard format for keystores, works with Java 8+
# Contains: private key + certificate + CA certificate chain
# -export: Export to PKCS12 format
# -inkey: Input private key file
# -in: Input certificate file
# -certfile: CA certificate to include in the chain
# -name: Alias for this entry in the keystore
openssl pkcs12 -export \
  -out server-keystore.p12 \
  -inkey server.key \
  -in server.crt \
  -certfile root-ca.cert \
  -name "server" \
  -password pass:changeit
```

**Convert PKCS12 to JKS (if needed):**

```bash
# Convert to legacy JKS format
# Only needed for very old Java versions (pre-Java 9)
# Modern Java applications should use PKCS12 directly
keytool -importkeystore \
  -srckeystore server-keystore.p12 \
  -srcstoretype PKCS12 \
  -srcstorepass changeit \
  -destkeystore server-keystore.jks \
  -deststoretype JKS \
  -deststorepass changeit \
  -alias server
```

#### Creating Truststores Manually

**What it does:** Creates a file containing trusted CA certificates. Applications use this to verify certificates presented by other parties.

**When to use:** For Java applications that need to verify SSL/TLS connections. Also ONLY needed for Java-based applications. Non-Java apps typically use the system's trusted CA store or a .crt file directly.

**Create JKS truststore:**

```bash
# Import CA certificate into truststore
# Truststore = store of trusted certificates (not keys)
# -trustcacerts: Mark this as a trusted certificate
# -alias: Name for this certificate in the truststore
# -noprompt: Don't ask for confirmation
keytool -import -trustcacerts \
  -alias root-ca \
  -file root-ca.cert \
  -keystore truststore.jks \
  -storepass changeit \
  -noprompt
```

**Create PKCS12 truststore:**

```bash
# Create PKCS12 format truststore
# Recommended for modern Java applications
keytool -import -trustcacerts \
  -alias root-ca \
  -file root-ca.cert \
  -keystore truststore.p12 \
  -storetype PKCS12 \
  -storepass changeit \
  -noprompt
```

**Add additional trusted certificates:**

```bash
# Add more certificates to existing truststore
# Useful when you need to trust multiple CAs or specific certificates
keytool -import -trustcacerts \
  -alias server-cert \
  -file server.crt \
  -keystore truststore.jks \
  -storepass changeit \
  -noprompt
```

#### Verifying Certificates and Keystores

**What it does:** Checks certificate details, validity, and truststore/keystore contents.

**When to use:** To troubleshoot certificate issues, verify certificate details, or confirm what's in a keystore/truststore.

**View certificate details:**

```bash
# View full certificate information
# Shows: subject, issuer, validity dates, extensions, public key, signature
openssl x509 -in server.crt -text -noout

# Check certificate validity dates
openssl x509 -in server.crt -noout -dates

# Verify certificate against CA
# Confirms the certificate was signed by the specified CA
openssl verify -CAfile root-ca.cert server.crt
```

Expected output from verify:
```
server.crt: OK
```

**List keystore contents:**

```bash
# List JKS keystore
# Shows: aliases, creation dates, certificate types
keytool -list -v -keystore server-keystore.jks -storepass changeit

# List PKCS12 keystore
keytool -list -v -keystore server-keystore.p12 -storetype PKCS12 -storepass changeit

# List truststore
# Shows all trusted certificates
keytool -list -v -keystore truststore.jks -storepass changeit
```

**Export certificate from keystore:**

```bash
# Extract just the certificate (not the private key)
# Useful for sharing the public certificate
keytool -export \
  -alias server \
  -file exported-cert.crt \
  -keystore server-keystore.jks \
  -storepass changeit
```

#### Common keytool Operations

**What they do:** Various maintenance operations on keystores and truststores.

**When to use:** When you need to manage, modify, or secure existing keystores.

**Change keystore password:**

```bash
# Update the password protecting the keystore file itself
keytool -storepasswd -keystore server-keystore.jks
```

**Change key password:**

```bash
# Update the password protecting a specific private key entry
# This is separate from the keystore password
keytool -keypasswd \
  -alias server \
  -keystore server-keystore.jks \
  -storepass changeit
```

**Delete entry from keystore:**

```bash
# Remove a certificate/key entry from the keystore
keytool -delete \
  -alias server \
  -keystore server-keystore.jks \
  -storepass changeit
```

**Convert JKS to PKCS12:**

```bash
# Migrate from legacy JKS to modern PKCS12 format
# Recommended for all Java applications
keytool -importkeystore \
  -srckeystore keystore.jks \
  -destkeystore keystore.p12 \
  -deststoretype PKCS12
```

---

### Application Integration

#### Java Applications

**Configure JVM SSL properties:**

```bash
# {keystore-path} - Full path to the PKCS12 or JKS keystore file
# {keystore-password} - Password protecting the keystore
# {truststore-path} - Full path to the JKS or PKCS12 truststore file
# {truststore-password} - Password protecting the truststore

# Server application (needs both keystore and truststore)
# Keystore: Contains the server's identity (private key + certificate)
# Truststore: Contains trusted CAs to verify client certificates
java -Djavax.net.ssl.keyStore={keystore-path} \
     -Djavax.net.ssl.keyStorePassword={keystore-password} \
     -Djavax.net.ssl.keyStoreType=PKCS12 \
     -Djavax.net.ssl.trustStore={truststore-path} \
     -Djavax.net.ssl.trustStorePassword={truststore-password} \
     -jar your-app.jar

# Client application (usually just needs truststore)
# Truststore: Contains trusted CAs to verify server certificates
java -Djavax.net.ssl.trustStore={truststore-path} \
     -Djavax.net.ssl.trustStorePassword={truststore-password} \
     -jar your-client-app.jar
```

**Spring Boot application.properties:**

```properties
# Server SSL (for HTTPS endpoints)
server.ssl.key-store=/path/to/server-keystore.p12
server.ssl.key-store-password=changeit
server.ssl.key-store-type=PKCS12

# Client SSL (for outgoing HTTPS requests)
server.ssl.trust-store=/path/to/truststore.jks
server.ssl.trust-store-password=changeit

# Mutual TLS (require client certificates)
server.ssl.client-auth=need
```

#### Apache Kafka

**Kafka Broker Configuration (server.properties):**

```properties
# Listeners
listeners=SSL://0.0.0.0:9093
advertised.listeners=SSL://kafka-0.example.com:9093
security.inter.broker.protocol=SSL

# SSL Keystore (broker's identity)
ssl.keystore.location=/path/to/kafka-0-keystore.p12
ssl.keystore.password=changeit
ssl.keystore.type=PKCS12
ssl.key.password=changeit

# SSL Truststore (to trust clients and other brokers)
ssl.truststore.location=/path/to/kafka-truststore.jks
ssl.truststore.password=changeit
ssl.truststore.type=JKS

# Require client authentication (mutual TLS)
ssl.client.auth=required

# Optional: Disable hostname verification for internal networks
ssl.endpoint.identification.algorithm=
```

**Kafka Producer/Consumer Configuration:**

```python
from confluent_kafka import Producer

# {bootstrap-servers} - Comma-separated list of Kafka broker addresses (e.g., kafka-0.example.com:9093,kafka-1.example.com:9093)
# {client-cert} - Path to client certificate file (.crt or .pem)
# {client-key} - Path to client private key file (.key)
# {ca-cert} - Path to CA certificate file to verify broker certificates

# Kafka Producer Configuration
config = {
    'bootstrap.servers': '{bootstrap-servers}',
    'security.protocol': 'SSL',
    
    # Client certificate (for mutual TLS)
    'ssl.certificate.location': '{client-cert}',
    'ssl.key.location': '{client-key}',
    
    # CA certificate (to trust Kafka brokers)
    'ssl.ca.location': '{ca-cert}',
    
    # Optional: Disable hostname verification
    'ssl.endpoint.identification.algorithm': 'none'
}

# Create producer
producer = Producer(config)

# Example usage
try:
    producer.produce('my-topic', key='key', value='value')
    producer.flush()
except Exception as e:
    print(f'Error producing message: {e}')
```

**Note:** Unlike Java Kafka clients, `confluent-kafka-python` does NOT use keystores or truststores. Instead, it uses separate certificate and key files directly (.crt and .key files), similar to how Nginx and PostgreSQL work.

**Consumer example:**

```python
from confluent_kafka import Consumer

# Kafka Consumer Configuration
config = {
    'bootstrap.servers': '{bootstrap-servers}',
    'group.id': 'my-consumer-group',
    'security.protocol': 'SSL',
    
    # Client certificate (for mutual TLS)
    'ssl.certificate.location': '{client-cert}',
    'ssl.key.location': '{client-key}',
    
    # CA certificate (to trust Kafka brokers)
    'ssl.ca.location': '{ca-cert}',
    
    # Optional: Disable hostname verification
    'ssl.endpoint.identification.algorithm': 'none',
    
    # Consumer-specific settings
    'auto.offset.reset': 'earliest'
}

# Create consumer
consumer = Consumer(config)
consumer.subscribe(['my-topic'])

try:
    while True:
        msg = consumer.poll(1.0)
        if msg is None:
            continue
        if msg.error():
            print(f'Consumer error: {msg.error()}')
            continue
        print(f'Received message: {msg.value().decode("utf-8")}')
except KeyboardInterrupt:
    pass
finally:
    consumer.close()
```

#### PostgreSQL

**Note:** PostgreSQL does NOT use keystores. It uses separate .crt and .key files.

**Server Configuration (postgresql.conf):**

```conf
ssl = on
ssl_cert_file = '/path/to/server.crt'
ssl_key_file = '/path/to/server.key'
ssl_ca_file = '/path/to/root-ca.cert'

# Require SSL for all connections
ssl_min_protocol_version = 'TLSv1.2'

# Require client certificates
ssl_client_auth = 'verify-full'
```

**Client Connection:**

```bash
# {username} - PostgreSQL username
# {hostname} - Database server hostname
# {port} - PostgreSQL port (default: 5432)
# {database} - Database name
# {client-cert} - Path to client certificate file
# {client-key} - Path to client private key file
# {ca-cert} - Path to CA certificate file
psql "postgresql://{username}@{hostname}:{port}/{database}?sslmode=verify-full&sslcert={client-cert}&sslkey={client-key}&sslrootcert={ca-cert}"
```

#### Nginx

**Note:** Nginx does NOT use keystores. It uses separate .crt and .key files.

**Nginx SSL Configuration:**

```nginx
server {
    listen 443 ssl;
    server_name grafana.example.com;

    # Server certificate and key
    ssl_certificate /path/to/server.crt;
    ssl_certificate_key /path/to/server.key;

    # SSL settings
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;

    # Client certificate verification (mutual TLS)
    ssl_client_certificate /path/to/root-ca.cert;
    ssl_verify_client on;
    ssl_verify_depth 2;

    location / {
        proxy_pass http://localhost:3000;
    }
}
```

#### Docker Registry

**Note:** Docker Registry does NOT use keystores. It uses separate .crt and .key files.

**Registry Configuration (config.yml):**

```yaml
version: 0.1
storage:
  filesystem:
    rootdirectory: /var/lib/registry

http:
  addr: :5000
  tls:
    certificate: /certs/server.crt
    key: /certs/server.key
    clientcas:
      - /certs/root-ca.cert
```

#### MongoDB

**Note:** MongoDB does NOT use keystores. It uses PEM bundle files (certificate + key combined).

**MongoDB Configuration (mongod.conf):**

```yaml
net:
  port: 27017
  ssl:
    mode: requireSSL
    PEMKeyFile: /path/to/server-bundle.pem  # Combined cert + key
    CAFile: /path/to/root-ca.cert
    allowConnectionsWithoutCertificates: false
```

**Client Connection:**

```bash
# {client-bundle} - PEM bundle containing client certificate and private key
# {ca-cert} - Path to CA certificate
# {hostname} - MongoDB server hostname
# {port} - MongoDB port (default: 27017)
mongo --ssl \
  --sslPEMKeyFile {client-bundle} \
  --sslCAFile {ca-cert} \
  --host {hostname}:{port}
```

#### Redis

**Note:** Redis does NOT use keystores. It uses separate .crt and .key files.

**Redis Configuration (redis.conf):**

```conf
port 0
tls-port 6379
tls-cert-file /path/to/server.crt
tls-key-file /path/to/server.key
tls-ca-cert-file /path/to/root-ca.cert
tls-auth-clients yes
```

#### Grafana

**Note:** Grafana does NOT use keystores. It uses separate .crt and .key files.

**Grafana Configuration (grafana.ini):**

```ini
[server]
protocol = https
cert_file = /path/to/server.crt
cert_key = /path/to/server.key

[security]
# Require client certificates
ssl_mode = require_cert
ca_cert_file = /path/to/root-ca.cert
```

#### Prometheus

**Note:** Prometheus does NOT use keystores. It uses separate .crt and .key files.

**Prometheus Configuration (prometheus.yml):**

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'node-exporters'
    scheme: https
    tls_config:
      cert_file: /path/to/client.crt
      key_file: /path/to/client.key
      ca_file: /path/to/root-ca.cert
      insecure_skip_verify: false
    static_configs:
      - targets: ['node-1:9100', 'node-2:9100']
```

---

### Troubleshooting

**Script not found after installation:**

Make sure `~/.local/bin` is in your PATH:

```bash
echo $PATH | grep -q "$HOME/.local/bin" && echo "[OK] In PATH" || echo "[ERROR] Not in PATH"
```

If not in PATH, add the export line to your shell config and reload.

**Permission denied:**

Make sure the script is executable:

```bash
chmod +x ~/.local/bin/generate-certs
```

**keytool command not found:**

Install the JDK (see [Prerequisites](#prerequisites) section).

**OpenSSL errors:**

Ensure OpenSSL is installed and up to date:

```bash
openssl version
# Should show version 1.1.1 or higher
```

**Certificate validation failed - "minimum 2 DNS names required":**

For server and combined certificates, you must have at least 2 DNS entries in your `.ext` file:

```ini
[alt_names]
DNS.1 = service.example.com
DNS.2 = service.internal.local
```

**Extension file validation failed:**

Ensure your `.ext` file has all required fields. Use `--template` to generate a correct template:

```bash
generate-certs --service myservice --type server --template
```

**Certificate already exists error:**

If you see "There is already a certificate for...", you need to revoke the old certificate first:

```bash
# Use --cleanup to automatically revoke and delete old certificates
generate-certs --service myservice --cleanup
```

**hostname verification failed in Java applications:**

If connecting to services by IP address or internal hostnames, you may need to disable hostname verification:

```bash
# Kafka
props.put("ssl.endpoint.identification.algorithm", "");

# Or set in Kafka broker config
ssl.endpoint.identification.algorithm=
```

**Certificate expired:**

Certificates generated by this tool are valid for 1000 days by default. To change this, edit `~/CA/openssl-ca.cnf`:

```ini
default_days = 1000  # Change this value
```

Then regenerate certificates.

**Cannot connect to service with client certificate:**

Verify that:
1. The server's truststore contains your CA certificate
2. Your client keystore contains the client certificate and private key
3. The client certificate hasn't been revoked
4. The server is configured to require/accept client certificates

**Keystore password not working:**

Load the passwords from the generated file:

```bash
source ~/.secrets/{service-name}/passwords.env
echo $KEYSTORE_PASSWORD_{service_name}
```

---

**Additional Resources:**
- [OpenSSL Documentation](https://www.openssl.org/docs/)
- [Java keytool Documentation](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/keytool.html)
- [Certificate Authorities and X.509](https://en.wikipedia.org/wiki/X.509)
- [Mutual TLS (mTLS) Explained](https://en.wikipedia.org/wiki/Mutual_authentication)

**License:**

This project is licensed under the MIT License - see the LICENSE file for details.
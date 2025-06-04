# OpenSSL Key Generation Guide

A comprehensive guide for generating various types of cryptographic keys and certificates using OpenSSL.

## Table of Contents
- [RSA Key Generation](#rsa-key-generation)
- [Elliptic Curve Keys](#elliptic-curve-keys)
- [SSL Certificates](#ssl-certificates)
- [Certificate Signing Requests](#certificate-signing-requests)
- [Random Keys and Secrets](#random-keys-and-secrets)
- [Key Format Conversion](#key-format-conversion)
- [Key Information and Verification](#key-information-and-verification)
- [File Extensions Reference](#file-extensions-reference)
- [Security Best Practices](#security-best-practices)

## RSA Key Generation

### Generate RSA Private Keys

Generate 2048-bit RSA private key:
```bash
openssl genrsa -out private_key.pem 2048
```

Generate 4096-bit RSA private key (more secure):
```bash
openssl genrsa -out private_key.pem 4096
```

Generate password-protected RSA private key:
```bash
openssl genrsa -aes256 -out private_key_encrypted.pem 2048
```

### Extract Public Key from Private Key

Extract public key from RSA private key:
```bash
openssl rsa -in private_key.pem -pubout -out public_key.pem
```

Extract public key from encrypted private key:
```bash
openssl rsa -in private_key_encrypted.pem -pubout -out public_key.pem
```

## Elliptic Curve Keys

### List Available Curves

Show all available elliptic curves:
```bash
openssl ecparam -list_curves
```

### Generate EC Private Keys

Generate EC private key (P-256 curve):
```bash
openssl ecparam -genkey -name prime256v1 -noout -out ec_private_key.pem
```

Generate EC private key (P-384 curve - more secure):
```bash
openssl ecparam -genkey -name secp384r1 -noout -out ec_private_key.pem
```

Generate EC private key (P-521 curve - highest security):
```bash
openssl ecparam -genkey -name secp521r1 -noout -out ec_private_key.pem
```

### Extract EC Public Key

Extract EC public key:
```bash
openssl ec -in ec_private_key.pem -pubout -out ec_public_key.pem
```

## SSL Certificates

### Self-Signed Certificates

Generate private key and self-signed certificate (no password):
```bash
openssl req -x509 -newkey rsa:2048 -keyout server_key.pem -out server_cert.pem -days 365 -nodes -subj "/C=US/ST=State/L=City/O=Organization/OU=Department/CN=example.com"
```

Generate with password protection:
```bash
openssl req -x509 -newkey rsa:2048 -keyout server_key.pem -out server_cert.pem -days 365 -subj "/C=US/ST=State/L=City/O=Organization/OU=Department/CN=example.com"
```

Generate wildcard certificate:
```bash
openssl req -x509 -newkey rsa:4096 -keyout wildcard_key.pem -out wildcard_cert.pem -days 365 -nodes -subj "/C=US/ST=State/L=City/O=Organization/CN=*.example.com"
```

### Multi-Domain (SAN) Certificates

Create a config file for SAN certificate:
```bash
cat > san.conf << EOF
[req]
distinguished_name = req_distinguished_name
req_extensions = v3_req
prompt = no

[req_distinguished_name]
C=US
ST=State
L=City
O=Organization
CN=example.com

[v3_req]
keyUsage = keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1 = example.com
DNS.2 = www.example.com
DNS.3 = api.example.com
DNS.4 = admin.example.com
EOF
```

Generate certificate with multiple domains:
```bash
openssl req -x509 -newkey rsa:2048 -keyout multi_domain_key.pem -out multi_domain_cert.pem -days 365 -nodes -config san.conf -extensions v3_req
```

## Certificate Signing Requests

### Generate CSRs

Generate private key and CSR:
```bash
openssl req -new -newkey rsa:2048 -nodes -keyout server_key.pem -out server_csr.pem -subj "/C=US/ST=State/L=City/O=Organization/OU=Department/CN=example.com"
```

Generate CSR from existing private key:
```bash
openssl req -new -key private_key.pem -out server_csr.pem -subj "/C=US/ST=State/L=City/O=Organization/OU=Department/CN=example.com"
```

Generate CSR with SAN (Subject Alternative Names):
```bash
openssl req -new -key private_key.pem -out server_csr.pem -config san.conf
```

## Random Keys and Secrets

### Generate Random Data

Generate 32-byte random key (base64 encoded):
```bash
openssl rand -base64 32
```

Generate 64-byte random key (base64 encoded):
```bash
openssl rand -base64 64
```

Generate 256-bit random key (hex encoded):
```bash
openssl rand -hex 32
```

Generate 512-bit random key (hex encoded):
```bash
openssl rand -hex 64
```

Generate random password (16 characters):
```bash
openssl rand -base64 16
```

Generate URL-safe random string:
```bash
openssl rand -base64 32 | tr -d "=+/" | cut -c1-25
```

### Application-Specific Keys

Generate JWT private key:
```bash
openssl genrsa -out jwt_private_key.pem 2048
```

Generate JWT public key:
```bash
openssl rsa -in jwt_private_key.pem -pubout -out jwt_public_key.pem
```

Generate session secret key:
```bash
openssl rand -base64 32
```

Generate API key:
```bash
openssl rand -hex 20
```

Generate database encryption key:
```bash
openssl rand -base64 44
```

## Key Format Conversion

### Format Conversions

Convert PEM to DER format:
```bash
openssl rsa -in private_key.pem -outform DER -out private_key.der
```

Convert DER to PEM format:
```bash
openssl rsa -in private_key.der -inform DER -out private_key.pem
```

Convert to PKCS#8 format (unencrypted):
```bash
openssl pkcs8 -topk8 -inform PEM -outform PEM -nocrypt -in private_key.pem -out private_key_pkcs8.pem
```

Convert to PKCS#8 format (encrypted):
```bash
openssl pkcs8 -topk8 -inform PEM -outform PEM -in private_key.pem -out private_key_pkcs8_encrypted.pem
```

Convert certificate PEM to DER:
```bash
openssl x509 -in certificate.pem -outform DER -out certificate.der
```

Create PKCS#12 bundle (certificate + private key):
```bash
openssl pkcs12 -export -out certificate.p12 -inkey private_key.pem -in certificate.pem
```

## Key Information and Verification

### View Key and Certificate Details

View private key details:
```bash
openssl rsa -in private_key.pem -text -noout
```

View public key details:
```bash
openssl rsa -in public_key.pem -pubin -text -noout
```

View certificate details:
```bash
openssl x509 -in certificate.pem -text -noout
```

View CSR details:
```bash
openssl req -in certificate.csr -text -noout
```

View PKCS#12 bundle contents:
```bash
openssl pkcs12 -in certificate.p12 -info -noout
```

### Verify Keys and Certificates

Verify private key:
```bash
openssl rsa -in private_key.pem -check
```

Verify certificate:
```bash
openssl x509 -in certificate.pem -noout -text
```

Check certificate expiration:
```bash
openssl x509 -in certificate.pem -noout -dates
```

Verify certificate chain:
```bash
openssl verify -CAfile ca_certificate.pem certificate.pem
```

Check if private key matches certificate (private key hash):
```bash
openssl rsa -noout -modulus -in private_key.pem | openssl md5
```

Check if private key matches certificate (certificate hash):
```bash
openssl x509 -noout -modulus -in certificate.pem | openssl md5
```

### Test SSL Connections

Test SSL connection to server:
```bash
openssl s_client -connect example.com:443 -servername example.com
```

Test with specific certificate:
```bash
openssl s_client -connect example.com:443 -CAfile ca_certificate.pem
```

Show certificate chain:
```bash
openssl s_client -connect example.com:443 -showcerts
```

## File Extensions Reference

| Extension | Description |
|-----------|-------------|
| `.pem` | Privacy Enhanced Mail (Base64 encoded) |
| `.der` | Distinguished Encoding Rules (Binary) |
| `.crt`, `.cer` | Certificate files |
| `.key` | Private key files |
| `.csr` | Certificate Signing Request |
| `.p12`, `.pfx` | PKCS#12 (contains both certificate and private key) |
| `.p7b`, `.p7c` | PKCS#7 certificate chain |
| `.jks` | Java KeyStore |

## Security Best Practices

### Key Generation
- Always use at least 2048-bit RSA keys (4096-bit preferred for long-term use)
- Consider Elliptic Curve keys (P-256, P-384) for better performance and security
- Use strong passphrases for encrypted keys
- Generate keys in a secure environment

### Key Management
- Set proper file permissions (600 for private keys, 644 for public keys/certificates)
- Keep private keys secure and never share them
- Store keys in secure locations (hardware security modules for production)
- Regularly rotate keys and certificates
- Use certificate pinning in production applications

### File Permissions

Set proper permissions for private keys (owner read/write only):
```bash
chmod 600 private_key.pem
```

Set proper permissions for public keys and certificates (owner read/write, others read):
```bash
chmod 644 public_key.pem certificate.pem
```

Set proper permissions for CSRs:
```bash
chmod 644 certificate.csr
```

### Certificate Validation
- Always validate certificate chains
- Check certificate expiration dates
- Verify certificate subjects and SANs
- Use OCSP or CRL for revocation checking
- Implement certificate transparency monitoring

## Common OpenSSL Commands Quick Reference

Generate new RSA key pair:
```bash
openssl genrsa -out key.pem 2048
```

```bash
openssl rsa -in key.pem -pubout -out pub.pem
```

Generate self-signed certificate:
```bash
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365 -nodes
```

Generate CSR:
```bash
openssl req -new -key key.pem -out csr.pem
```

Convert formats:
```bash
openssl x509 -in cert.pem -outform DER -out cert.der
```

View certificate info:
```bash
openssl x509 -in cert.pem -text -noout
```

Test SSL connection:
```bash
openssl s_client -connect domain.com:443
```

---

**Note**: Always ensure you're using the latest version of OpenSSL and follow current security recommendations for your specific use case.

# OpenSSL Cheat Sheet

The most common OpenSSL commands in one place — generate keys, create certificates, decode, convert, and troubleshoot SSL/TLS.

> **Want to skip the command line?** [getaCert.com](https://getacert.com) does all of this from your browser — no installs, no flags to memorize. Free since 2005.

---

## Table of Contents

- [Generate Private Keys](#generate-private-keys)
- [Generate a CSR](#generate-a-csr)
- [Generate a Self-Signed Certificate](#generate-a-self-signed-certificate)
- [Decode and View Certificates](#decode-and-view-certificates)
- [Decode and View CSRs](#decode-and-view-csrs)
- [Check a Remote Server's Certificate](#check-a-remote-servers-certificate)
- [Convert Certificate Formats](#convert-certificate-formats)
- [PKCS#12 / PFX Bundles](#pkcs12--pfx-bundles)
- [Verify and Validate](#verify-and-validate)
- [CA Operations](#ca-operations)
- [Quick Reference Table](#quick-reference-table)

---

## Generate Private Keys

### RSA

```bash
# RSA 2048-bit (standard)
openssl genrsa -out server.key 2048

# RSA 4096-bit (stronger, slower)
openssl genrsa -out server.key 4096

# RSA with passphrase
openssl genrsa -aes256 -out server.key 2048
```

### ECDSA

```bash
# ECDSA P-256 (fast, widely supported)
openssl ecparam -genkey -name prime256v1 -noout -out server.key

# ECDSA P-384
openssl ecparam -genkey -name secp384r1 -noout -out server.key
```

### Ed25519

```bash
openssl genpkey -algorithm Ed25519 -out server.key
```

### Remove passphrase from a key

```bash
openssl rsa -in server.key -out server-nopass.key
```

---

## Generate a CSR

### Interactive (prompts for details)

```bash
openssl req -new -key server.key -out server.csr
```

### One-liner with subject

```bash
openssl req -new -key server.key -out server.csr \
  -subj "/C=US/ST=Washington/L=Seattle/O=My Company/CN=www.example.com"
```

### CSR with Subject Alternative Names (SANs)

```bash
openssl req -new -key server.key -out server.csr \
  -subj "/CN=www.example.com" \
  -addext "subjectAltName=DNS:www.example.com,DNS:example.com,DNS:api.example.com"
```

### Generate key and CSR in one step

```bash
openssl req -new -newkey rsa:2048 -nodes -keyout server.key -out server.csr \
  -subj "/C=US/ST=Washington/L=Seattle/O=My Company/CN=www.example.com"
```

> **Easier way:** Paste your CSR at [getaCert.com/signcsr](https://getacert.com/signcsr) to get it signed instantly, or generate a complete certificate set at [getaCert.com/selfsign](https://getacert.com/selfsign).

---

## Generate a Self-Signed Certificate

### Basic (30 days)

```bash
openssl req -x509 -newkey rsa:2048 -nodes \
  -keyout server.key -out server.pem -days 30 \
  -subj "/CN=www.example.com"
```

### With SANs (required by modern browsers)

```bash
openssl req -x509 -newkey rsa:2048 -nodes \
  -keyout server.key -out server.pem -days 365 \
  -subj "/CN=www.example.com" \
  -addext "subjectAltName=DNS:www.example.com,DNS:localhost,IP:127.0.0.1"
```

### With full subject details

```bash
openssl req -x509 -newkey rsa:2048 -nodes \
  -keyout server.key -out server.pem -days 365 \
  -subj "/C=US/ST=Washington/L=Seattle/O=My Company/OU=Engineering/CN=www.example.com" \
  -addext "subjectAltName=DNS:www.example.com,DNS:example.com" \
  -addext "keyUsage=digitalSignature,keyEncipherment" \
  -addext "extendedKeyUsage=serverAuth"
```

### From an existing key and CSR

```bash
openssl x509 -req -in server.csr -signkey server.key -out server.pem -days 365
```

> **Easier way:** [getaCert.com/selfsign](https://getacert.com/selfsign) generates the key, CSR, certificate, and PKCS#12 bundle in one click. Supports RSA, ECDSA, and Ed25519.

---

## Decode and View Certificates

### View certificate details

```bash
openssl x509 -in server.pem -text -noout
```

### View just the subject and issuer

```bash
openssl x509 -in server.pem -subject -issuer -noout
```

### View expiry dates

```bash
openssl x509 -in server.pem -dates -noout
```

### View SANs

```bash
openssl x509 -in server.pem -noout -ext subjectAltName
```

### View serial number and fingerprint

```bash
openssl x509 -in server.pem -serial -fingerprint -noout
```

> **Easier way:** Paste any PEM certificate at [getaCert.com/viewcert](https://getacert.com/viewcert) to see all fields in a readable format.

---

## Decode and View CSRs

### View CSR details

```bash
openssl req -in server.csr -text -noout
```

### Verify a CSR's signature

```bash
openssl req -in server.csr -verify -noout
```

> **Easier way:** Paste any CSR at [getaCert.com/decodecsr](https://getacert.com/decodecsr) to view the parsed fields.

---

## Check a Remote Server's Certificate

### View a server's certificate

```bash
openssl s_client -connect example.com:443 -servername example.com </dev/null 2>/dev/null | \
  openssl x509 -text -noout
```

### Check expiry date

```bash
openssl s_client -connect example.com:443 -servername example.com </dev/null 2>/dev/null | \
  openssl x509 -noout -dates
```

### View the full certificate chain

```bash
openssl s_client -connect example.com:443 -servername example.com -showcerts </dev/null
```

### Check a specific port (e.g., SMTP with STARTTLS)

```bash
openssl s_client -connect mail.example.com:587 -starttls smtp </dev/null 2>/dev/null | \
  openssl x509 -text -noout
```

### Test TLS version support

```bash
# Test TLS 1.2
openssl s_client -connect example.com:443 -tls1_2 </dev/null

# Test TLS 1.3
openssl s_client -connect example.com:443 -tls1_3 </dev/null
```

> **Easier way:** [getaCert.com/check](https://getacert.com/check) checks any domain's certificate, chain, TLS version, and cipher suites from your browser.

---

## Convert Certificate Formats

### PEM to DER

```bash
openssl x509 -in cert.pem -outform DER -out cert.der
```

### DER to PEM

```bash
openssl x509 -in cert.der -inform DER -outform PEM -out cert.pem
```

### PEM to PKCS#7

```bash
openssl crl2pkcs7 -nocrl -certfile cert.pem -out cert.p7b
```

### PKCS#7 to PEM

```bash
openssl pkcs7 -in cert.p7b -print_certs -out cert.pem
```

### PEM key to PKCS#8

```bash
openssl pkcs8 -topk8 -in server.key -out server-pkcs8.key -nocrypt
```

> **Easier way:** [getaCert.com/convert](https://getacert.com/convert) converts between PEM, DER, and PKCS#7 with drag-and-drop.

---

## PKCS#12 / PFX Bundles

### Create a PKCS#12 bundle (for importing into browsers, Java, Windows)

```bash
openssl pkcs12 -export -out cert.p12 \
  -inkey server.key -in server.pem -certfile ca.pem \
  -passout pass:mypassword
```

### Extract certificate from PKCS#12

```bash
openssl pkcs12 -in cert.p12 -clcerts -nokeys -out cert.pem
```

### Extract private key from PKCS#12

```bash
openssl pkcs12 -in cert.p12 -nocerts -nodes -out server.key
```

### Extract CA certificates from PKCS#12

```bash
openssl pkcs12 -in cert.p12 -cacerts -nokeys -out ca.pem
```

---

## Verify and Validate

### Verify a certificate against a CA

```bash
openssl verify -CAfile ca.pem server.pem
```

### Verify a certificate chain

```bash
openssl verify -CAfile ca.pem -untrusted intermediate.pem server.pem
```

### Check that a key matches a certificate

```bash
# These two commands should output the same modulus
openssl x509 -in server.pem -noout -modulus | openssl md5
openssl rsa -in server.key -noout -modulus | openssl md5
```

### Check that a CSR matches a key

```bash
openssl req -in server.csr -noout -modulus | openssl md5
openssl rsa -in server.key -noout -modulus | openssl md5
```

> **Easier way:** [getaCert.com/chaincheck](https://getacert.com/chaincheck) validates your certificate chain and identifies ordering issues.

---

## CA Operations

### Create a root CA

```bash
# Generate CA key
openssl genrsa -aes256 -out ca.key 4096

# Generate CA certificate (10 years)
openssl req -x509 -new -nodes -key ca.key -sha256 -days 3650 \
  -out ca.pem -subj "/C=US/ST=Washington/O=My CA/CN=My Root CA"
```

### Sign a CSR with your CA

```bash
openssl x509 -req -in server.csr -CA ca.pem -CAkey ca.key \
  -CAcreateserial -out server.pem -days 365 -sha256
```

### Sign with extensions (SANs)

```bash
openssl x509 -req -in server.csr -CA ca.pem -CAkey ca.key \
  -CAcreateserial -out server.pem -days 365 -sha256 \
  -extfile <(printf "subjectAltName=DNS:www.example.com,DNS:example.com")
```

> **Managing a CA with OpenSSL is painful.** Serial numbers, index files, config files, revocation lists — it's a lot of overhead for signing a few certs. [getaCert.com](https://getacert.com) handles all of this for you. For programmatic access, the [REST API](https://getacert.com/about) lets you generate and sign certs with a single HTTP request.

---

## Quick Reference Table

| Task | Command |
|------|---------|
| Generate RSA key | `openssl genrsa -out key.pem 2048` |
| Generate ECDSA key | `openssl ecparam -genkey -name prime256v1 -noout -out key.pem` |
| Generate CSR | `openssl req -new -key key.pem -out req.csr` |
| Self-signed cert | `openssl req -x509 -newkey rsa:2048 -nodes -keyout key.pem -out cert.pem -days 30` |
| View certificate | `openssl x509 -in cert.pem -text -noout` |
| View CSR | `openssl req -in req.csr -text -noout` |
| Check remote cert | `openssl s_client -connect host:443 </dev/null \| openssl x509 -text -noout` |
| PEM → DER | `openssl x509 -in cert.pem -outform DER -out cert.der` |
| DER → PEM | `openssl x509 -in cert.der -inform DER -out cert.pem` |
| Create PKCS#12 | `openssl pkcs12 -export -out cert.p12 -inkey key.pem -in cert.pem` |
| Verify cert chain | `openssl verify -CAfile ca.pem cert.pem` |
| Key matches cert? | Compare: `openssl x509 -modulus` vs `openssl rsa -modulus` |

---

## About

This cheat sheet is maintained by [getaCert.com](https://getacert.com) — free SSL certificate tools since 2005. No installs. No command line. Just your certificate.

- [Generate a self-signed certificate](https://getacert.com/selfsign)
- [Generate a CA-signed certificate](https://getacert.com/casign)
- [Sign a CSR](https://getacert.com/signcsr)
- [Decode a certificate](https://getacert.com/viewcert)
- [Check a domain's SSL](https://getacert.com/check)
- [Convert certificate formats](https://getacert.com/convert)
- [Validate a certificate chain](https://getacert.com/chaincheck)
- [REST API for automation](https://getacert.com/about)

## License

This cheat sheet is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Share it, fork it, print it out and tape it to your monitor.

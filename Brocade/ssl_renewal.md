# 🔐 SOP: Brocade SSL Certificate Renewal

Here are the exact steps to renew your expiring SSL certificate on your Brocade switch (`10.1.1.12`). To clarify, Brocade FOS requires generating a *new* certificate to replace the expiring one. 

Choose the path that fits your company's security policy! 👇

---

## 🚀 Option 1: Renew via Self-Signed Certificate (Fastest)
If you don't use an internal CA and just need HTTPS back online, do this directly on your Brocade switch.

**1. Generate and replace the certificate (Valid for 5 years):**
```bash
seccertmgmt generate -cert https -type rsa -keysize 2048 -hash sha256 -years 5
```
*(⚠️ Note: Press `y` when prompted. This deletes the old cert, temporarily drops HTTPS sessions, and causes a brief CPU spike.)*

**2. Verify the new expiration date:**
```bash
seccertmgmt show -cert https
```

---

## 🏢 Option 2: Renew via CA-Signed Certificate
If your security team requires an internal Certificate Authority (CA), follow these steps using your Jump Host (`10.1.1.1`).

### 🛠️ Step 1: Generate & Export the CSR (On Brocade)
Log into your Brocade switch (`10.1.1.12`).

1. **Generate the new CSR:**
   ```bash
   seccertmgmt generate -csr https
   ```
   *(💡 When prompted for the Common Name, enter `10.1.1.12`)*

2. **Export the CSR to your Jump Host:**
   ```bash
   seccertmgmt export -csr https -protocol scp -ipaddr 10.1.1.1 -remotedir /tmp -login admin
   ```

### ✍️ Step 2: Get the Certificate Signed (Off-Switch)
1. Take the `/tmp/https.csr` file from your Jump Host (`10.1.1.1`) to your CA.
2. Place the newly signed certificate (e.g., `switch.pem`) and your CA trust chain (e.g., `ca_chain.pem`) back into the `/tmp` folder of your Jump Host.
*(⚠️ Ensure both files are in Base64 PEM format!)*

### 📥 Step 3: Import Renewed Certificates (On Brocade)
Go back to your Brocade switch (`10.1.1.12`) SSH session.

1. **Import the CA chain first:**
   ```bash
   seccertmgmt import -ca -server https -protocol scp -ipaddr 10.1.1.1 -remotedir /tmp -certname ca_chain.pem -login admin
   ```

2. **Import the new switch certificate:**
   ```bash
   seccertmgmt import -cert https -protocol scp -ipaddr 10.1.1.1 -remotedir /tmp -certname switch.pem -login admin
   ```

### ✅ Step 4: Verify the Renewal
Check that the new expiration date is active:
```bash
seccertmgmt show -cert https
```

---
**Would you like to proceed with the Self-Signed method, or do you need to generate a CSR for your CA? 🤔**

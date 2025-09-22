# mTLS Passthrough Listener on Azure Application Gateway

## Overview

This repository provides ARM templates and step-by-step guidance for deploying an **Azure Application Gateway** configured with a **mutual TLS (mTLS) passthrough listener**.

In mTLS passthrough mode, Application Gateway requests a client certificate during TLS negotiation but does **not enforce client-certificate verification at the edge**. The gateway forwards the client certificate (and related attributes, if available) to the backend via server variables or header injection. The backend is responsible for verifying the certificate chain, mapping identity, and enforcing policy. Connectivity is not rejected at the gateway based solely on client-certificate state.

---

## Features

- **mTLS Passthrough Listener:** Gateway requests client certificates but does not enforce validation.
- **Backend Verification:** Backend services receive the client certificate and perform verification.
- **Automated ARM Deployment:** Templates automate secure, scalable, and compliant connectivity.

---

## Prerequisites

- Azure subscription and resource group
- Admin access to Azure CLI
- SSL certificate for listener  
  *No Client CA certificate is required for mTLS passthrough listener*

---

## Key Properties

| Name                   | Type      | Description                                               |
|------------------------|-----------|-----------------------------------------------------------|
| verifyClientIssuerDN   | boolean   | Verify client certificate issuer name on the gateway      |
| verifyClientRevocation | enum      | Verify client certificate revocation                      |
| verifyClientCertMode   | enum      | Client certificate mode (`Strict` or `Passthrough`)       |

**ApplicationGatewayVerifyClientCertMode Options:**
- **Strict:** Gateway enforces mTLS. Requests without a valid client certificate are rejected at the gateway.
- **Passthrough:** Gateway requests a client certificate but does not enforce it. Backend receives the certificate and performs verification. Requests are not rejected by the gateway solely due to absent or invalid client certificate.

---

## Example ARM Template Snippet

```json
"sslProfiles": [
  {
    "name": "mTlsPassthroughCert",
    "properties": {
      "clientAuthConfiguration": {
        "verifyClientIssuerDN": false,
        "verifyClientRevocation": "None",
        "verifyClientCertMode": "Passthrough"
      }
    }
  }
]
```

---

## Deployment Steps

### 1. Review and Customize Parameters

- Open `azuredeployv2.newcert.parameters.json`
- Update values for:
  - `addressPrefix` and `subnetPrefix`: Define your network ranges.
  - `skuName` and `capacity`: Choose gateway size and instance count.
  - `adminUsername` and `adminSSHKey`: Set admin credentials.
  - `certData` and `certPassword`: Paste your SSL certificate and password.

### 2. Deploy Using Azure CLI

Run the following command in your working directory:
```sh
az deployment group create --resource-group <your-resource-group> --template-file mtlsdeploy_novmss.json --parameters azuredeployv2.newcert.parameters.json
```

### 3. Validate Deployment

- Go to the Azure Portal and check the Application Gateway resource.
- Confirm that the **mTLS passthrough listener** is enabled and associated with your SSL certificate.

### 4. Test Connectivity

- Verify connectivity. Connection should be established even if a client certificate is not provided.

---

## Notes

- The backend is responsible for all client certificate validation and policy enforcement.
- This deployment ensures secure, scalable, and compliant connectivity for your applications.

---

**For questions or issues, please open an issue in this repository.**

---

Let me know if you want to add usage examples, troubleshooting tips, or more details for contributors!

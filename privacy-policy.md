# Velum Privacy Policy

**Last updated:** February 2026

Velum is designed as a privacy-first budgeting application.  
This privacy policy explains what data Velum handles, where it is stored, and how it is protected.

## Overview

Velum:

- Does **not operate developer-controlled servers**
- Does **not collect analytics or telemetry**
- Does **not track users across apps or websites**
- Does **not sell, share, or monetize user data**

Your financial data remains under your control at all times.

---

## Data Storage

### Default: iCloud Storage (CloudKit)

By default, Velum stores your data in your personal Apple iCloud account using Apple's CloudKit framework.

This means:

- Data is stored in your own iCloud container, not on developer servers
- Apple manages infrastructure security and encryption
- The Velum developer cannot access your private data

CloudKit data is protected using Apple’s standard encryption and security systems as part of iCloud.

You can learn more about Apple's data protection here:

https://support.apple.com/en-us/HT202303

---

### Optional: Self-Hosted Backend

Velum optionally supports connecting to a self-hosted backend.

If you enable this:

- Your data is stored on infrastructure you control
- Security depends on your server configuration
- The Velum developer has no access to this data

You are responsible for maintaining the security of any self-hosted deployment.

#### Self-Hosted Backend Source Code

Velum’s optional self-hosted backend is open source.  
If you choose to run your own server, you can review or deploy it here:

https://github.com/VelumApp/velum

---

## Financial Data Access (SimpleFIN Integration)

Velum can optionally connect to financial institutions via **SimpleFIN Bridge**.

When enabled:

- Velum receives read-only transaction data
- Bank credentials are handled by SimpleFIN, not Velum
- Velum does not store banking login credentials

SimpleFIN’s privacy policy is available here:

https://beta-bridge.simplefin.org/privacy

---

## Data Collection by Apple (TestFlight & Platform Diagnostics)

If you install Velum through TestFlight or the App Store, Apple may collect:

- Crash logs
- Device diagnostics
- Performance data

This data is handled by Apple under their privacy policies and is not accessible to the Velum developer except in aggregated diagnostic form.

---

## Analytics, Tracking, and Advertising

Velum does **not**:

- Include analytics SDKs
- Use tracking technologies
- Serve advertisements
- Profile user behavior
- Share data with advertisers

---

## Data Security

Velum relies on platform-level security:

- Apple CloudKit encryption for iCloud storage
- Apple device security protections
- Optional user-managed server security for self-hosting

While reasonable safeguards are used, no system can guarantee absolute security.

---

## Data Deletion

You can delete your data at any time:

### iCloud users
Delete app data through:

- iOS Settings → Apple ID → iCloud → Manage Storage  
or  
- By removing app data within Velum (if available)

### Self-hosted users
Delete data directly from your backend database or server.

---

## Children’s Privacy

Velum is not directed at children under 13 and does not knowingly collect information from children.

---

## Changes to This Policy

This policy may be updated occasionally.

If significant changes occur:

- The updated policy will appear here
- The "Last updated" date will change

Continued use of Velum indicates acceptance of the updated policy.

---

## Contact

If you have questions about privacy:

**Email:** petergelgor7@gmail.com

---

## Legal Note

This policy is provided for transparency regarding Velum’s data practices.  
Velum is designed to minimize developer access to user data wherever possible.
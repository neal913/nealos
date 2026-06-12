# B2B Client Account Setup (Manual Process)

**Last Updated:** 2026-04-27

This document describes how to manually set up a SendSwell account for a B2B client when using invoice-based billing instead of self-service checkout.

---

## Overview

For B2B clients, the normal self-signup flow is bypassed. Instead, you:
1. Create a Stripe customer and subscription/invoice
2. Create a Firebase Auth user
3. Create the Firestore documents (Organization + User)
4. Send login credentials to the client

### What's Auto-Generated vs Manual

| Value | Source | You Do |
|-------|--------|--------|
| Stripe Customer ID (`cus_...`) | Stripe | Copy it |
| Stripe Subscription ID (`sub_...`) | Stripe | Copy it |
| Firebase User UID | Firebase Auth | Copy it |
| Organization Doc ID | Firestore Auto-ID | Copy it |
| Password | You | Generate (see requirements below) |
| Email, Name, Company | Client | Enter as provided |

---

## Prerequisites

You need access to:
- **Stripe Dashboard:** https://dashboard.stripe.com
- **Firebase Console:** https://console.firebase.google.com/project/optinboxer

---

## Plan Reference

| Plan | Leads/Month | Price (Monthly) | Price (Annual) |
|------|-------------|-----------------|----------------|
| starter | 50 | $7 | $70 |
| creator | 150 | $19 | $190 |
| pro | 375 | $39 | $390 |
| publisher | 1,500 | $79 | $790 |

---

## Step 1: Create Stripe Customer + Subscription

### 1A. Create Customer

1. Go to **Stripe Dashboard** → **Customers** → **+ Add customer**
2. Fill in:
   - **Name:** Client's company name
   - **Email:** Client's email address
3. Click **Add customer**
4. **Copy the Customer ID** (starts with `cus_...`)

### 1B. Create Subscription (or Invoice)

**Option A: Recurring Subscription**

1. On the customer page, click **Actions** → **Create subscription**
2. Select the appropriate price:
   - Starter Monthly: `price_1SuJhVEcF65UWHvMb00ZZ7OP`
   - Starter Annual: `price_1SuJhVEcF65UWHvMmPVGpLGj`
   - Creator Monthly: `price_1SuJjxEcF65UWHvMTD1oavic`
   - Creator Annual: `price_1SuJkREcF65UWHvM2CyF5cBq`
   - Pro Monthly: `price_1SuJlcEcF65UWHvMTHKLPSoV`
   - Pro Annual: `price_1SuJlrEcF65UWHvMRtEPAyty`
   - Publisher Monthly: `price_1SuJmwEcF65UWHvM96MP7Ob5`
   - Publisher Annual: `price_1SuJnBEcF65UWHvMCzL9slIi`
3. Click **Start subscription**
4. **Copy the Subscription ID** (starts with `sub_...`)

**Option B: One-Time Invoice**

1. On the customer page, click **Actions** → **Create invoice**
2. Add line item with custom amount for the agreed pricing
3. Send the invoice
4. Note: You'll need to manually create subscription data in Firestore

---

## Step 2: Create Firebase Auth User

1. Go to **Firebase Console** → **Authentication** → **Users**
2. Click **Add user**
3. Fill in:
   - **Email:** Client's email address
   - **Password:** Generate a secure password (see requirements below)
4. Click **Add user**
5. **Copy the User UID** from the table (Firebase generates this automatically)

### Password Requirements

Password must meet **4 of these 5 criteria**:
- 8+ characters
- One lowercase letter
- One uppercase letter
- One number
- One special character (`!@#$%^&*` etc.)

**Good examples:**
- `Welcome2026!`
- `Client@Acme99`
- `SendSwell#2026`

**Save this info:**
```
Email: client@company.com
Password: [your generated password]
UID: [copy from Firebase - auto-generated]
```

---

## Step 3: Create Firestore Documents

### 3A. Create Organization Document

1. Go to **Firebase Console** → **Firestore Database**
2. Select the `organizations` collection (or create it if it doesn't exist)
3. Click **Add document**
4. For **Document ID:** Click **"Auto-ID"** button (Firestore generates a unique ID like `oEaV1nCW7rA27Bp36xwq`)
5. Add fields:

```
name: "Client Company Name"                    (string)
ownerId: "[Firebase UID from Step 2]"          (string)
members: ["[Firebase UID from Step 2]"]        (array of strings)
createdAt: [click timestamp icon]              (timestamp - now)
stripeCustomerId: "cus_..."                    (string)
```

6. Add `subscription` as a **map** with these fields:

```
subscription:
  status: "active"                             (string)
  planId: "pro"                                (string - starter/creator/pro/publisher)
  subscriptionId: "sub_..."                    (string - from Stripe, or leave empty for invoices)
  billingInterval: "month"                     (string - "month" or "year")
  currentPeriodEnd: [timestamp - 30 days out]  (timestamp)
  cancelAtPeriodEnd: false                     (boolean)
```

7. Add `quota` as a **map** with these fields:

```
quota:
  leadsLimit: 375                              (number - based on plan, see table)
  leadsUsed: 0                                 (number)
  bonusLeads: 0                                (number)
  shadowLimit: 200000                          (number - based on plan)
  shadowUsed: 0                                (number)
  resetDate: [same as currentPeriodEnd]        (timestamp)
```

**Quota limits by plan:**

| Plan | leadsLimit | shadowLimit |
|------|------------|-------------|
| starter | 50 | 25000 |
| creator | 150 | 75000 |
| pro | 375 | 200000 |
| publisher | 1500 | 750000 |

8. Click **Save**
9. **Copy the Organization Document ID**

### 3B. Create User Document

1. In Firestore, go to **users** collection (or create it)
2. Click **Add document**
3. For **Document ID:** Paste the **exact Firebase UID** from Step 2 (do NOT use Auto-ID here)
   - Example: `a1B2c3D4e5F6g7H8i9J0`
4. Add fields:

```
uid: "[Firebase UID]"                          (string)
email: "client@company.com"                    (string)
fullName: "Client Name"                        (string)
organizationId: "[Organization ID from 3A]"    (string)
onboardingStatus: "account_created"            (string)
createdAt: [click timestamp icon]              (timestamp - now)
```

5. Click **Save**

---

## Step 4: Send Login Credentials

Email the client:

```
Subject: Your SendSwell Account

Hi [Name],

Your SendSwell account is ready. Here are your login details:

Login URL: https://sendswell.com/login
Email: [their email]
Password: [generated password]

Please change your password after first login.

Next Steps:
1. Log in at sendswell.com/login
2. Add your first site (domain where you'll capture leads)
3. Install the snippet on your website
4. Configure your Inbox Magic settings

Let me know if you have any questions!
```

---

## Verification Checklist

After setup, verify:

- [ ] Client can log in at sendswell.com/login
- [ ] Dashboard shows correct plan in Settings
- [ ] Client can create a Site
- [ ] Integrations page loads (not "No site found")
- [ ] Leads page loads (empty is fine, just no errors)

---

## Troubleshooting

### "No site found" Error
- Client hasn't created a Site yet. This is expected for new accounts.
- Guide them to create their first site in the dashboard.

### "Access denied" or Blank Pages
- Check that `users/{uid}.organizationId` matches the Organization document ID
- Check that `organizations/{orgId}.ownerId` matches the user's UID

### Wrong Plan/Quota Showing
- Verify `organizations/{orgId}.subscription.planId` is correct
- Verify `organizations/{orgId}.quota.leadsLimit` matches the plan

### Can't Access Billing Portal
- Verify `organizations/{orgId}.stripeCustomerId` is set and matches Stripe

---

## Future: Admin CLI Script

When manual process becomes painful (10+ clients), build a script:

```bash
npm run create-client -- \
  --email="client@company.com" \
  --name="Client Name" \
  --company="Company Inc" \
  --plan="pro" \
  --billing="monthly"
```

This would automate Steps 1-4 and output credentials.

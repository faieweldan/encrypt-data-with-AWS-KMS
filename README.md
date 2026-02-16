# Encryption with AWS KMS and DynamoDB

This project shows how I used AWS KMS to encrypt a DynamoDB table, control who can decrypt the data, and test access using different IAM users.

The goal was to understand how encryption at rest works and how KMS permissions are enforced separately from normal DynamoDB permissions.

---

## Architecture Overview

- Customer Managed KMS Key
- DynamoDB table encrypted with that key
- IAM Admin user (full access)
- Test IAM user (no KMS access at first)

---

# Part 1 – Create a Customer Managed KMS Key

## What I did

- Created a Customer Managed Key
- Used symmetric encryption
- Key usage: Encrypt and decrypt
- Set IAM Admin as:
  - Key Administrator
  - Key User

## Key configuration

- Alias: `nextwork-kms-key`
- Type: Symmetric
- Usage: Encrypt and decrypt

<img width="764" height="444" alt="Screenshot 2026-02-16 at 3 57 18 PM" src="https://github.com/user-attachments/assets/7cca24f9-970d-4d91-ab84-4308a14ef892" />

Customer Managed Keys give full control over:
- Key policy
- Key lifecycle
- Who can encrypt
- Who can decrypt

---

# Part 2 – Create and Encrypt DynamoDB Table

## Table configuration

- Table name: `nextwork-kms-table`
- Partition key: `id` (String)
- Encryption type: Stored in your account (Customer Managed Key)
- Selected key: `nextwork-kms-key`

<img width="767" height="450" alt="Screenshot 2026-02-16 at 3 57 41 PM" src="https://github.com/user-attachments/assets/1dca99c8-398d-4363-a752-435dc0696442" />

Encryption at rest means the data is encrypted while stored.  
Even if someone gets access to the storage layer, they cannot read the data without the key.

AWS uses transparent data encryption, so authorized users see decrypted data automatically.

---

# Part 3 – Insert and Verify Data

Added item:

```
id = 1
```

Verified:
- Admin user can see the item
- Data is decrypted automatically

<img width="768" height="455" alt="Screenshot 2026-02-16 at 3 58 04 PM" src="https://github.com/user-attachments/assets/117d209f-5236-4cc9-b931-d1834a43c192" />

---

# Part 4 – Create Restricted Test User

Created IAM user:

`nextwork-kms-user`

Attached permission:
- AmazonDynamoDBFullAccess

Did NOT give any KMS permissions.

Goal:
Check if DynamoDB access alone is enough to read encrypted data.

---

# Part 5 – Validate Encryption Enforcement

Logged in as test user using incognito.

Tried to view items in DynamoDB.

Result:

Access Denied  
Error: `kms:Decrypt`

<img width="763" height="450" alt="Screenshot 2026-02-16 at 3 58 58 PM" src="https://github.com/user-attachments/assets/816ce04b-a481-4e6f-b702-39648d41df25" />

This confirmed:
- DynamoDB permission alone is not enough
- KMS key permission is required to decrypt data

---

# Part 6 – Grant Key Access

As Admin:

- Went to KMS
- Modified key policy
- Added `nextwork-kms-user` as Key User

This allowed the user to:
- Encrypt
- Decrypt
- ReEncrypt
- Generate data keys
- Describe key

---

# Final Validation

Logged back in as test user.

Refreshed DynamoDB.

Result:
- Data is visible
- Decryption works

This proves KMS controls decryption separately from DynamoDB permissions.

---

# Key Takeaways

1. Encryption protects the data itself, not just access to the service.
2. IAM controls service access.
3. KMS controls decryption rights.
4. Both must allow access for data to be readable.

---

# Cleanup

- Deleted DynamoDB table
- Scheduled KMS key deletion (7 days)
- Deleted IAM test user
- Removed local credential files

---

# What I Learned

- How KMS works with DynamoDB
- Difference between AWS managed keys and customer managed keys
- How key policies translate into permissions
- How encryption enforcement works in real scenarios



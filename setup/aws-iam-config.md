# AWS IAM Configuration — Bedrock Least-Privilege Access

Setup guide for creating the dedicated least-privilege IAM user and policy used to run Garak against AWS Bedrock.

---

## Overview

Rather than using an admin account for lab testing, a dedicated IAM user (`garak-bedrock`) is created with the minimum permissions required to invoke Bedrock models. This follows least-privilege principles and isolates lab credentials from the admin account.

---

## Step 1 — Enable Bedrock Model Access

Before creating IAM resources, enable access to the target model in the AWS Bedrock console.

1. Navigate to **AWS Bedrock → Model access**
2. Request access to **Meta Llama 3.1 8B Instruct**
3. Wait for access to be granted (typically immediate for on-demand models)

> **Note:** The Bedrock model access console UI has changed. If you don't see the expected model list, check under **Bedrock configurations → Model access** in the left sidebar.

---

## Step 2 — Create the IAM Policy

Create a custom policy named `GarakBedrockInvokePolicy` scoped to the minimum required actions.

### Policy JSON

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "BedrockInvokeAccess",
      "Effect": "Allow",
      "Action": [
        "bedrock:InvokeModel",
        "bedrock:InvokeModelWithResponseStream"
      ],
      "Resource": [
        "arn:aws:bedrock:us-east-1::foundation-model/meta.llama3-1-8b-instruct-v1:0",
        "arn:aws:bedrock:us-west-2::foundation-model/meta.llama3-1-8b-instruct-v1:0",
        "arn:aws:bedrock:us-east-2::foundation-model/meta.llama3-1-8b-instruct-v1:0"
      ]
    }
  ]
}
```

> **Why three regions?** Cross-region inference profiles (e.g., `us.meta.llama3-1-8b-instruct-v1:0`) route requests across `us-east-1`, `us-west-2`, and `us-east-2` dynamically. Scoping the policy to only one region will cause `AccessDeniedException` when the inference profile routes to another region.

### Create via AWS CLI

```bash
aws iam create-policy \
  --policy-name GarakBedrockInvokePolicy \
  --policy-document file://garak-bedrock-policy.json \
  --profile greg-admin
```

---

## Step 3 — Create the IAM User

```bash
aws iam create-user \
  --user-name garak-bedrock \
  --profile greg-admin
```

---

## Step 4 — Attach the Policy

```bash
aws iam attach-user-policy \
  --user-name garak-bedrock \
  --policy-arn arn:aws:iam::<ACCOUNT_ID>:policy/GarakBedrockInvokePolicy \
  --profile greg-admin
```

---

## Step 5 — Create Access Keys

```bash
aws iam create-access-key \
  --user-name garak-bedrock \
  --profile greg-admin
```

Save the `AccessKeyId` and `SecretAccessKey` from the output — this is the only time the secret key is shown.

---

## Step 6 — Configure AWS CLI Profile

Add the `garak-bedrock` profile to your AWS CLI credentials:

```bash
aws configure --profile garak-bedrock
```

Enter:
- AWS Access Key ID: `<from Step 5>`
- AWS Secret Access Key: `<from Step 5>`
- Default region: `us-east-1`
- Default output format: `json`

Verify the profile resolves correctly:

```bash
aws sts get-caller-identity --profile garak-bedrock
```

Expected output:
```json
{
    "UserId": "AIDA...",
    "Account": "<ACCOUNT_ID>",
    "Arn": "arn:aws:iam::<ACCOUNT_ID>:user/garak-bedrock"
}
```

---

## Step 7 — Verify Bedrock Access

Test that the `garak-bedrock` user can invoke the model:

```bash
aws bedrock-runtime invoke-model \
  --model-id us.meta.llama3-1-8b-instruct-v1:0 \
  --body '{"prompt":"Hello","max_gen_len":10}' \
  --cli-binary-format raw-in-base64-out \
  --profile garak-bedrock \
  output.json && cat output.json
```

A successful response confirms the IAM policy and Bedrock access are configured correctly.

---

## Common Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `AccessDeniedException` on `InvokeModel` | Policy missing one or more regional ARNs | Add all three US region ARNs to the policy resource block |
| `AccessDeniedException` on `InvokeModel` | Model access not enabled in Bedrock console | Enable model access under Bedrock → Model access |
| `ValidationException: on-demand throughput not supported` | Using standard model ID instead of inference profile | Use `us.meta.llama3-1-8b-instruct-v1:0` not `meta.llama3-1-8b-instruct-v1:0` |
| `NoCredentialsError` in Garak | Profile not exported to environment | Run `export AWS_PROFILE=garak-bedrock` before invoking Garak |

---

## Cost Controls

This lab used under $1.00 in total Bedrock charges. To avoid unexpected costs:

- Set an AWS Budget alert on the root account (recommended as a substitute for direct root billing access)
- Monitor usage in **Billing → Cost Explorer** grouped by service
- The `garak-bedrock` user has no billing permissions by design — review costs via the admin account only

---

## Security Notes

- The `garak-bedrock` user has no console access — programmatic access only
- Access keys should be rotated or deleted after lab work is complete
- Never use the root account or admin account credentials directly in Garak
- Do not commit access keys to the repository — use environment variables or the AWS credentials file only
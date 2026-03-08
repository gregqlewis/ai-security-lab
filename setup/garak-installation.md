# Garak Installation & Configuration — AWS Bedrock

Setup guide for Garak v0.14.0 on Kali Linux targeting Meta Llama 3.1 8B Instruct via AWS Bedrock.

---

## Prerequisites

- Kali Linux
- AWS CLI configured with a least-privilege IAM profile (see `aws-iam-config.md`)
- Python 3.x
- Internet access for pip and rustup

---

## Step 1 — Create an Isolated Virtual Environment

Garak has dependencies that can conflict with system packages on Kali. Use a virtual environment.

```bash
python3 -m venv ~/garak-env
source ~/garak-env/bin/activate
```

**Issue:** `pip` not found after activating the venv.  
**Fix:** Use `python -m pip` instead of calling `pip` directly:

```bash
python -m pip install --upgrade pip
```

---

## Step 2 — Install Rust (Required for base2048 Dependency)

Garak's `base2048` encoding package requires Rust/Cargo to compile. Kali does not include Rust by default.

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source "$HOME/.cargo/env"
```

Verify:
```bash
rustc --version
cargo --version
```

---

## Step 3 — Install Garak

```bash
python -m pip install garak
```

Verify installation:
```bash
python -m garak --version
```

Expected output: `garak v0.14.0`

---

## Step 4 — Configure AWS Credentials

Garak does not read AWS credentials directly from `~/.aws/credentials` profiles. Pass them via environment variables.

```bash
export AWS_PROFILE=garak-bedrock
export AWS_DEFAULT_REGION=us-east-1
```

Verify the credentials resolve correctly:
```bash
aws sts get-caller-identity --profile garak-bedrock
```

Expected output should show the `garak-bedrock` IAM user ARN, not your admin account.

> **Note:** Setting `AWS_PROFILE` alone is not always sufficient. If you receive a `NoCredentialsError`, explicitly export both variables in the same shell session before running Garak.

---

## Step 5 — Configure the Bedrock Target

Meta Llama 3.1 8B Instruct is accessed via a cross-region inference profile, not the standard on-demand model ID.

**Correct target format:**
```
us.meta.llama3-1-8b-instruct-v1:0
```

**Do not use:**
```
meta.llama3-1-8b-instruct-v1:0   # on-demand format — throws ValidationException
```

### Deprecated CLI Flags

Garak v0.14.0 removed the old `--model_type` and `--model_name` flags.

| Old (deprecated) | New (v0.14.0) |
|-----------------|---------------|
| `--model_type` | `--target_type` |
| `--model_name` | `--target_name` |

---

## Step 6 — Run a Test Probe

Verify end-to-end connectivity before running full suites:

```bash
python -m garak \
  --target_type bedrock \
  --target_name us.meta.llama3-1-8b-instruct-v1:0 \
  --probes dan.Dan_11_0 \
  --generations 1
```

**Expected:** Garak connects, sends one generation, and returns a result report.

**Common errors at this stage:**

| Error | Cause | Fix |
|-------|-------|-----|
| `NoCredentialsError` | AWS credentials not resolved | Export `AWS_PROFILE` and `AWS_DEFAULT_REGION` in current shell |
| `ValidationException: on-demand throughput not supported` | Using standard model ID instead of inference profile | Switch to `us.meta.llama3-1-8b-instruct-v1:0` |
| `AccessDeniedException` on inference profile | IAM policy missing one or more regional ARNs | Update policy to include all three US region ARNs (see `aws-iam-config.md`) |

---

## Step 7 — Run Full Probe Suites

```bash
# DAN Jailbreak
python -m garak \
  --target_type bedrock \
  --target_name us.meta.llama3-1-8b-instruct-v1:0 \
  --probes dan.Dan_11_0 \
  --generations 5

# Prompt Injection
python -m garak \
  --target_type bedrock \
  --target_name us.meta.llama3-1-8b-instruct-v1:0 \
  --probes promptinject \
  --generations 5

# Encoding Evasion
python -m garak \
  --target_type bedrock \
  --target_name us.meta.llama3-1-8b-instruct-v1:0 \
  --probes encoding \
  --generations 5

# Training Data Leakage
python -m garak \
  --target_type bedrock \
  --target_name us.meta.llama3-1-8b-instruct-v1:0 \
  --probes leakreplay \
  --generations 5
```

> **Note:** The leakage probe name in v0.14.0 is `leakreplay`, not `replay`. Using `replay` will throw a probe-not-found error.

**SSH session drop mid-suite:** If your SSH session drops during a long probe run, rerun only the remaining probes by specifying them individually rather than rerunning the full suite:

```bash
python -m garak \
  --target_type bedrock \
  --target_name us.meta.llama3-1-8b-instruct-v1:0 \
  --probes encoding.InjectNato,encoding.InjectZalgo \
  --generations 5
```

---

## Step 8 — Locate Reports

Garak saves HTML reports automatically after each run:

```bash
ls ~/.local/share/garak/
```

Reports are named `garak.<uuid>.report.html`. Copy them to the repo:

```bash
cp ~/.local/share/garak/garak.*.report.html \
  ~/Documents/GitHub/ai-security-lab/probes/<suite>/results/
```

---

## Quick Reference

```bash
# Activate environment
source ~/garak-env/bin/activate

# Set credentials
export AWS_PROFILE=garak-bedrock
export AWS_DEFAULT_REGION=us-east-1

# Run a probe
python -m garak \
  --target_type bedrock \
  --target_name us.meta.llama3-1-8b-instruct-v1:0 \
  --probes <probe_name> \
  --generations 5
```
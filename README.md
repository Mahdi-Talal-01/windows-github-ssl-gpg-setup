# GitHub SSL and GPG Setup Guide for Windows 11

This guide will walk you through setting up GitHub with SSL authentication and GPG commit signing on Windows 11.

## Prerequisites

- Windows 11 computer
- Git installed
- GitHub account

## Table of Contents

1. [Initial Git Configuration](#1-initial-git-configuration)
2. [SSH Key Setup](#2-ssh-key-setup)
3. [Adding SSH Key to GitHub](#3-adding-ssh-key-to-github)
4. [Testing SSH Connection](#4-testing-ssh-connection)
5. [GPG Setup for Commit Signing](#5-gpg-setup-for-commit-signing)
6. [Configuring Git for GPG Signing](#6-configuring-git-for-gpg-signing)
7. [Testing Signed Commits](#7-testing-signed-commits)
8. [Troubleshooting](#8-troubleshooting)

---

## 1. Initial Git Configuration

Configure Git with your credentials:

```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

**Example:**
```bash
git config --global user.name "John Doe"
git config --global user.email "john.doe@example.com"
```

---

## 2. SSH Key Setup

### 2.1 Generate SSH Key

Open Command Prompt or PowerShell and run:

```bash
ssh-keygen -t ed25519 -C "your.email@example.com"
```

**Options during generation:**
- **File location:** Press Enter (uses default location)
- **Passphrase:** Enter a passphrase or press Enter for no passphrase

### 2.2 Display Your Public Key

```bash
type C:\Users\%USERNAME%\.ssh\id_ed25519.pub
```

Copy the entire output (starts with `ssh-ed25519`).

---

## 3. Adding SSH Key to GitHub

1. Go to [GitHub SSH Settings](https://github.com/settings/keys)
2. Click **"New SSH key"**
3. **Title:** Give it a descriptive name (e.g., "Windows 11 Laptop")
4. **Key type:** Select **"Authentication Key"**
5. **Key:** Paste your public key
6. Click **"Add SSH key"**

---

## 4. Testing SSH Connection

### 4.1 Start SSH Agent (PowerShell as Administrator)

```powershell
Start-Service ssh-agent
ssh-add C:\Users\%USERNAME%\.ssh\id_ed25519
```

### 4.2 Test Connection

```bash
ssh -T git@github.com
```

**Expected output:**
```
Hi [your-username]! You've successfully authenticated, but GitHub does not provide shell access.
```

---

## 5. GPG Setup for Commit Signing

### 5.1 Install Kleopatra (GPG Frontend)

1. Download from [GnuPG website](https://www.gnupg.org/download/)
2. Install with default settings

### 5.2 Create GPG Key Pair

1. Open **Kleopatra**
2. Click **"New Key Pair"**
3. Select **"Create a personal OpenPGP key pair"**
4. Fill in details:
   - **Name:** Your full name
   - **Email:** Same email as your Git configuration
5. Click **"Create"**
6. Set a passphrase (recommended)

### 5.3 Export GPG Public Key

1. Right-click on your new key in Kleopatra
2. Select **"Export..."**
3. Save to a file or export to clipboard
4. Copy the entire key content (from `-----BEGIN PGP PUBLIC KEY BLOCK-----` to `-----END PGP PUBLIC KEY BLOCK-----`)

### 5.4 Add GPG Key to GitHub

1. Go to [GitHub SSH and GPG Keys](https://github.com/settings/keys)
2. Click **"New GPG key"**
3. Paste your exported public key
4. Click **"Add GPG key"**

---

## 6. Configuring Git for GPG Signing

### 6.1 Get Your GPG Key ID

**In Git Bash:**
```bash
gpg --list-secret-keys --keyid-format=long
```

**Or with full path:**
```bash
"C:/Program Files (x86)/GnuPG/bin/gpg.exe" --list-secret-keys --keyid-format=long
```

**Example output:**
```
sec   ed25519/1234567890ABCDEF 2025-09-07 [SC]
```

Your key ID is: `1234567890ABCDEF`

### 6.2 Configure Git

```bash
git config --global user.signingkey YOUR_KEY_ID
git config --global commit.gpgsign true
git config --global gpg.program "C:/Program Files (x86)/GnuPG/bin/gpg.exe"
```

**Example:**
```bash
git config --global user.signingkey 1234567890ABCDEF
git config --global commit.gpgsign true
git config --global gpg.program "C:/Program Files (x86)/GnuPG/bin/gpg.exe"
```

### 6.3 Verify Configuration

```bash
git config --global --list | grep -E "(signingkey|gpgsign|gpg.program)"
```

**Expected output:**
```
user.signingkey=1234567890ABCDEF
commit.gpgsign=true
gpg.program=C:/Program Files (x86)/GnuPG/bin/gpg.exe
```

---

## 7. Testing Signed Commits

### 7.1 Make a Test Commit

```bash
cd /path/to/your/repo
git add .
git commit -m "Test signed commit"
git push
```

### 7.2 Verify on GitHub

Check your repository on GitHub - you should see a green **"Verified"** badge next to your commit.

---

## 8. Troubleshooting

### Common Issues and Solutions

#### 8.1 "Permission denied (publickey)" Error

**Solution:**
- Ensure SSH key is properly added to GitHub
- Test SSH connection: `ssh -T git@github.com`
- Check if SSH agent is running and key is loaded: `ssh-add -l`

#### 8.2 "gpg failed to sign the data" Error

**Possible causes and solutions:**

1. **Wrong key ID format:**
   ```bash
   git config --global user.signingkey YOUR_CORRECT_KEY_ID
   ```

2. **GPG program not found:**
   ```bash
   git config --global gpg.program "C:/Program Files (x86)/GnuPG/bin/gpg.exe"
   ```

3. **Email mismatch between Git and GPG:**
   ```bash
   git config --global user.email "email@used.in.gpg.key"
   ```

#### 8.3 "Unverified" Commits on GitHub

**Cause:** Email in GPG key doesn't match Git committer email.

**Solution:**
```bash
# Check current Git email
git config --global user.email

# Update to match GPG key email
git config --global user.email "email@used.in.gpg.key"
```

#### 8.4 SSH Agent Issues on Windows

**Start SSH agent service:**
```powershell
# In PowerShell as Administrator
Start-Service ssh-agent
Set-Service -Name ssh-agent -StartupType Automatic
```

#### 8.5 GPG Passphrase Prompt Issues

If GPG doesn't prompt for passphrase or fails silently:

1. **Install GPG Suite with GUI support**
2. **Set GPG_TTY environment variable:**
   ```bash
   export GPG_TTY=$(tty)
   ```

### Verification Commands

**Check SSH setup:**
```bash
ssh -T git@github.com
```

**Check GPG setup:**
```bash
gpg --list-secret-keys --keyid-format=long
git config --global --list | grep gpg
```

**Test GPG signing:**
```bash
echo "test" | gpg --clearsign
```

---

## Quick Reference

### SSH Commands
```bash
# Generate SSH key
ssh-keygen -t ed25519 -C "email@example.com"

# Add to SSH agent
ssh-add ~/.ssh/id_ed25519

# Test connection
ssh -T git@github.com
```

### GPG Commands
```bash
# List secret keys
gpg --list-secret-keys --keyid-format=long

# Test signing
echo "test" | gpg --clearsign
```

### Git Configuration
```bash
# Basic setup
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# GPG setup
git config --global user.signingkey YOUR_KEY_ID
git config --global commit.gpgsign true
git config --global gpg.program "C:/Program Files (x86)/GnuPG/bin/gpg.exe"
```

---

## Additional Resources

- [GitHub SSH Documentation](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
- [GitHub GPG Documentation](https://docs.github.com/en/authentication/managing-commit-signature-verification)
- [GnuPG Documentation](https://www.gnupg.org/documentation/)

---

**Note:** Replace placeholder values like `YOUR_KEY_ID`, `your.email@example.com`, and file paths with your actual values.
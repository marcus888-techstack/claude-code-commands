# GCP Auth List

## Purpose
Display all authenticated Google Cloud accounts and identify the active account

## Context
Use to check which accounts are authenticated and which one is currently active. Helpful when working with multiple GCP accounts or troubleshooting authentication issues.

## Parameters
- `$ARGUMENTS` - Optional format flags
  - Optional
  - Example: `--format=json`, `--filter=status:ACTIVE`

## Steps

### 1. List all authenticated accounts
```bash
gcloud auth list $ARGUMENTS
```
Shows all accounts with authentication tokens, marking the active one with an asterisk (*).

### 2. Show detailed account information
```bash
gcloud config get-value account
```
Displays only the currently active account email.

### 3. Check token status
```bash
gcloud auth print-access-token &>/dev/null && echo "Token is valid" || echo "Token is invalid/expired"
```
Verifies if the current authentication token is still valid.

### 4. Display credential file locations
```bash
echo "Credentials stored in: $HOME/.config/gcloud/"
ls -la $HOME/.config/gcloud/credentials.db 2>/dev/null || echo "No credentials database found"
```
Shows where authentication data is stored locally.

## Validation
- At least one account is listed
- Active account is clearly marked with *
- Account email addresses are displayed correctly

## Error Handling
- **"No credentialed accounts"** - Need to run `gcloud auth login` first
- **"Token expired"** - Re-authenticate with `gcloud auth login`
- **Empty output** - Check if gcloud is properly installed

## Safety Notes
- This is a read-only operation
- Does not modify any authentication state
- Safe to run frequently

## Examples
- **List all accounts**
  ```
  gcp-auth-list
  ```
  Shows all authenticated accounts with active account marked

- **List in JSON format**
  ```
  gcp-auth-list "--format=json"
  ```
  Returns account list in JSON for scripting

- **Filter active accounts only**
  ```
  gcp-auth-list "--filter=status:ACTIVE"
  ```
  Shows only the currently active account
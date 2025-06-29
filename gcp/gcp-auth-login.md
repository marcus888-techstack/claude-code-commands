# GCP Auth Login

## Purpose
Authenticate with Google Cloud Platform and set up credentials for gcloud CLI

## Context
Use this command to log into GCP when starting work or switching between different Google accounts. Essential before running any GCP operations. Creates or updates application default credentials.

## Parameters
- `$ARGUMENTS` - Optional flags for authentication
  - Optional
  - Example: `--no-launch-browser`, `--account=user@example.com`

## Steps

### 1. Check current authentication status
```bash
gcloud auth list
```
Shows currently authenticated accounts to understand the starting state.

### 2. Perform authentication
```bash
gcloud auth login $ARGUMENTS
```
Opens browser for Google authentication. If --no-launch-browser is used, provides a URL to authenticate manually.

### 3. Set application default credentials
```bash
gcloud auth application-default login
```
Creates credentials for client libraries and tools that use Application Default Credentials.

### 4. Verify authentication
```bash
gcloud config get-value account
```
Confirms the active account after login.

### 5. Test access
```bash
gcloud projects list --limit=1
```
Quick test to ensure authentication works by listing one project.

## Validation
- Active account is displayed correctly
- Can list at least one project
- No authentication errors in subsequent commands
- Application default credentials are created

## Error Handling
- **"ERROR: (gcloud.auth.login) You do not currently have an active account selected"** - Run the login command again
- **"Permission denied"** - Check if you have the correct permissions for the organization
- **"The browser could not be opened"** - Use `--no-launch-browser` flag
- **"Invalid grant"** - Token may be expired, re-authenticate

## Safety Notes
- Stores credentials locally in ~/.config/gcloud/
- Credentials persist until explicitly revoked
- Use `gcloud auth revoke` to log out when done

## Examples
- **Standard login**
  ```
  gcp-auth-login
  ```
  Opens browser for interactive authentication

- **Login without browser**
  ```
  gcp-auth-login "--no-launch-browser"
  ```
  Provides URL for manual authentication

- **Login with specific account**
  ```
  gcp-auth-login "--account=developer@company.com"
  ```
  Authenticates with a specific Google account
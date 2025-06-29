# GCP Auth Switch

## Purpose
Switch between authenticated Google Cloud accounts without re-authenticating

## Context
Use when you need to switch between different GCP accounts that are already authenticated. Useful for developers working across multiple organizations or projects with different credentials.

## Parameters
- `$ACCOUNT` - Email address of the account to switch to
  - Required
  - Example: `developer@company.com`

## Steps

### 1. List available accounts
```bash
gcloud auth list --format="value(account)"
```
Shows all authenticated accounts available for switching.

### 2. Validate target account exists
```bash
if ! gcloud auth list --format="value(account)" | grep -q "^$ACCOUNT$"; then
    echo "Error: Account $ACCOUNT is not authenticated"
    echo "Run: gcloud auth login --account=$ACCOUNT"
    exit 1
fi
```
Ensures the target account is already authenticated.

### 3. Switch to the specified account
```bash
gcloud config set account $ACCOUNT
```
Changes the active account to the specified one.

### 4. Verify the switch
```bash
echo "Switched to account: $(gcloud config get-value account)"
```
Confirms the active account has changed.

### 5. Update application default credentials
```bash
echo "Updating application default credentials..."
gcloud auth application-default login --account=$ACCOUNT
```
Updates ADC to match the new active account for client libraries.

### 6. Test new account access
```bash
gcloud projects list --limit=1 --format="value(projectId)" || echo "Warning: Could not list projects"
```
Verifies the new account has proper permissions.

## Validation
- Active account matches the requested account
- Can perform basic operations with new account
- Application default credentials are updated

## Error Handling
- **"Account not found"** - Account needs to be authenticated first with `gcloud auth login`
- **"Permission denied"** - New account may not have necessary permissions
- **"Invalid account"** - Check the email format is correct

## Safety Notes
- Only switches between already authenticated accounts
- Previous account remains authenticated (not logged out)
- May need to update project selection after switching

## Examples
- **Switch to work account**
  ```
  gcp-auth-switch work@company.com
  ```
  Switches active account to work@company.com

- **Switch to personal account**
  ```
  gcp-auth-switch personal@gmail.com
  ```
  Switches active account to personal@gmail.com
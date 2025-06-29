# GCP Config List

## Purpose
Display all gcloud CLI configuration properties and their current values

## Context
Use to view your current gcloud configuration settings including active account, project, compute region/zone, and other properties. Helpful for debugging issues or understanding your current CLI context.

## Parameters
- `$CONFIGURATION` - Specific configuration to display
  - Optional
  - Default: Current active configuration
  - Example: `default`, `production`, `development`
- `$SECTION` - Specific section to display
  - Optional
  - Example: `core`, `compute`, `container`

## Steps

### 1. Show active configuration name
```bash
echo "=== GCloud Configuration ==="
ACTIVE_CONFIG=$(gcloud config configurations list --filter="is_active:true" --format="value(name)")
echo "Active configuration: $ACTIVE_CONFIG"

# Use specified configuration if provided
CONFIG_NAME=${CONFIGURATION:-$ACTIVE_CONFIG}
if [ "$CONFIG_NAME" != "$ACTIVE_CONFIG" ]; then
    echo "Showing configuration: $CONFIG_NAME"
fi
```
Identifies which configuration to display.

### 2. List all configurations
```bash
echo -e "\n=== Available Configurations ==="
gcloud config configurations list --format="table(
    name,
    is_active,
    account,
    project,
    compute.region,
    compute.zone
)"
```
Shows all available configurations.

### 3. Display core properties
```bash
echo -e "\n=== Core Properties ==="
if [ -z "$SECTION" ] || [ "$SECTION" = "core" ]; then
    echo "Account: $(gcloud config get-value account --configuration=$CONFIG_NAME 2>/dev/null || echo 'Not set')"
    echo "Project: $(gcloud config get-value project --configuration=$CONFIG_NAME 2>/dev/null || echo 'Not set')"
    echo "Disable usage reporting: $(gcloud config get-value disable_usage_reporting --configuration=$CONFIG_NAME 2>/dev/null || echo 'False')"
    echo "Project quota check: $(gcloud config get-value project --configuration=$CONFIG_NAME 2>/dev/null || echo 'True')"
fi
```
Shows core configuration properties.

### 4. Display compute properties
```bash
if [ -z "$SECTION" ] || [ "$SECTION" = "compute" ]; then
    echo -e "\n=== Compute Properties ==="
    echo "Region: $(gcloud config get-value compute/region --configuration=$CONFIG_NAME 2>/dev/null || echo 'Not set')"
    echo "Zone: $(gcloud config get-value compute/zone --configuration=$CONFIG_NAME 2>/dev/null || echo 'Not set')"
fi
```
Shows compute-related settings.

### 5. Display all properties
```bash
if [ -z "$SECTION" ]; then
    echo -e "\n=== All Configuration Properties ==="
    gcloud config list --configuration=$CONFIG_NAME
fi
```
Shows complete configuration.

### 6. Show configuration file location
```bash
echo -e "\n=== Configuration Files ==="
CONFIG_DIR="$HOME/.config/gcloud"
echo "Configuration directory: $CONFIG_DIR"
echo "Active config file: $CONFIG_DIR/configurations/config_$CONFIG_NAME"

if [ -f "$CONFIG_DIR/configurations/config_$CONFIG_NAME" ]; then
    echo -e "\nFile contents:"
    cat "$CONFIG_DIR/configurations/config_$CONFIG_NAME" | sed 's/^/  /'
fi
```
Shows where configurations are stored.

### 7. Display environment variables
```bash
echo -e "\n=== Environment Variables ==="
echo "Google Cloud environment variables currently set:"
env | grep -E "^(GOOGLE|GCLOUD|GCP)" | sort | sed 's/^/  /' || echo "  None found"

echo -e "\nCommon environment variables:"
echo "  GOOGLE_APPLICATION_CREDENTIALS: ${GOOGLE_APPLICATION_CREDENTIALS:-Not set}"
echo "  GOOGLE_CLOUD_PROJECT: ${GOOGLE_CLOUD_PROJECT:-Not set}"
echo "  CLOUDSDK_CORE_PROJECT: ${CLOUDSDK_CORE_PROJECT:-Not set}"
```
Shows relevant environment variables.

### 8. Show application default credentials
```bash
echo -e "\n=== Application Default Credentials ==="
ADC_PATH="$HOME/.config/gcloud/application_default_credentials.json"
if [ -f "$ADC_PATH" ]; then
    echo "Status: Configured"
    echo "Path: $ADC_PATH"
    
    # Extract account from credentials
    ADC_ACCOUNT=$(cat "$ADC_PATH" | grep -o '"client_email": "[^"]*"' | cut -d'"' -f4)
    if [ -n "$ADC_ACCOUNT" ]; then
        echo "Account: $ADC_ACCOUNT"
    fi
else
    echo "Status: Not configured"
    echo "To configure: gcloud auth application-default login"
fi
```
Shows ADC status.

### 9. Provide configuration management commands
```bash
echo -e "\n=== Configuration Management ==="
cat << 'EOF'
Common configuration commands:

# Create new configuration
gcloud config configurations create CONFIG_NAME

# Switch configuration
gcloud config configurations activate CONFIG_NAME

# Set properties
gcloud config set project PROJECT_ID
gcloud config set compute/zone ZONE
gcloud config set compute/region REGION

# Unset properties
gcloud config unset project

# Delete configuration
gcloud config configurations delete CONFIG_NAME

# Set property for specific configuration
gcloud config set project PROJECT_ID --configuration=CONFIG_NAME
EOF
```
Provides useful configuration commands.

## Validation
- Active configuration is identified
- Properties are displayed correctly
- Configuration files exist
- Values match expected formats

## Error Handling
- **"Configuration not found"** - Check configuration name
- **"Property not set"** - Normal for optional properties
- **"Permission denied"** - Check file permissions in ~/.config/gcloud

## Safety Notes
- Read-only operation, no changes made
- Some properties may contain sensitive project names
- ADC files contain credentials - handle carefully
- Environment variables override config file settings

## Examples
- **List current configuration**
  ```
  gcp-config-list
  ```
  Shows all properties in active configuration

- **List specific configuration**
  ```
  gcp-config-list production
  ```
  Shows properties for 'production' configuration

- **List only compute settings**
  ```
  gcp-config-list "" compute
  ```
  Shows only compute section properties

- **Compare configurations**
  ```
  gcp-config-list default && echo "---" && gcp-config-list production
  ```
  Shows both configurations for comparison
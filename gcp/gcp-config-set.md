# GCP Config Set

## Purpose
Set gcloud CLI configuration properties for project, region, zone, and other settings

## Context
Use to configure default values for gcloud commands. These settings persist across terminal sessions and reduce the need to specify common parameters like project ID or compute zone in every command.

## Parameters
- `$PROPERTY` - The configuration property to set
  - Required
  - Example: `project`, `compute/zone`, `compute/region`
- `$VALUE` - The value to set for the property
  - Required
  - Example: `my-project-id`, `us-central1-a`, `us-central1`
- `$CONFIGURATION` - Configuration to modify
  - Optional
  - Default: Current active configuration
  - Example: `default`, `production`

## Steps

### 1. Validate property format
```bash
# Common properties mapping
case $PROPERTY in
    project|core/project)
        FULL_PROPERTY="project"
        SECTION="core"
        ;;
    zone|compute/zone)
        FULL_PROPERTY="compute/zone"
        SECTION="compute"
        ;;
    region|compute/region)
        FULL_PROPERTY="compute/region"
        SECTION="compute"
        ;;
    account|core/account)
        FULL_PROPERTY="account"
        SECTION="core"
        ;;
    *)
        FULL_PROPERTY="$PROPERTY"
        SECTION=$(echo $PROPERTY | cut -d'/' -f1)
        ;;
esac

echo "Setting property: $FULL_PROPERTY = $VALUE"
```
Normalizes property names.

### 2. Show current value
```bash
echo -e "\n=== Current Configuration ==="
CURRENT_CONFIG=$(gcloud config configurations list --filter="is_active:true" --format="value(name)")
CONFIG_NAME=${CONFIGURATION:-$CURRENT_CONFIG}

echo "Configuration: $CONFIG_NAME"
CURRENT_VALUE=$(gcloud config get-value $FULL_PROPERTY --configuration=$CONFIG_NAME 2>/dev/null)

if [ -n "$CURRENT_VALUE" ]; then
    echo "Current value: $CURRENT_VALUE"
else
    echo "Current value: (not set)"
fi
```
Displays current configuration state.

### 3. Validate value based on property type
```bash
echo -e "\n=== Validation ==="
case $FULL_PROPERTY in
    project)
        # Validate project exists
        if gcloud projects describe $VALUE &>/dev/null; then
            echo "✓ Project '$VALUE' exists"
        else
            echo "⚠️  Warning: Project '$VALUE' not found or not accessible"
            read -p "Continue anyway? (y/N): " -n 1 -r
            echo
            if [[ ! $REPLY =~ ^[Yy]$ ]]; then
                exit 1
            fi
        fi
        ;;
    compute/zone)
        # Validate zone format
        if [[ $VALUE =~ ^[a-z]+-[a-z0-9]+-[a-z]$ ]]; then
            echo "✓ Zone format is valid"
        else
            echo "⚠️  Warning: '$VALUE' doesn't match zone format (region-zone)"
        fi
        ;;
    compute/region)
        # Validate region format
        if [[ $VALUE =~ ^[a-z]+-[a-z0-9]+$ ]]; then
            echo "✓ Region format is valid"
        else
            echo "⚠️  Warning: '$VALUE' doesn't match region format"
        fi
        ;;
    account)
        # Validate account is authenticated
        if gcloud auth list --filter="account:$VALUE" --format="value(account)" | grep -q "$VALUE"; then
            echo "✓ Account '$VALUE' is authenticated"
        else
            echo "⚠️  Warning: Account '$VALUE' is not authenticated"
            echo "Authenticate with: gcloud auth login --account=$VALUE"
        fi
        ;;
esac
```
Validates the value is appropriate for the property.

### 4. Set the configuration property
```bash
echo -e "\n=== Setting Property ==="
if [ "$CONFIG_NAME" != "$CURRENT_CONFIG" ]; then
    gcloud config set $FULL_PROPERTY $VALUE --configuration=$CONFIG_NAME
else
    gcloud config set $FULL_PROPERTY $VALUE
fi

if [ $? -eq 0 ]; then
    echo "✓ Property set successfully"
else
    echo "✗ Failed to set property"
    exit 1
fi
```
Sets the configuration property.

### 5. Verify the change
```bash
echo -e "\n=== Verification ==="
NEW_VALUE=$(gcloud config get-value $FULL_PROPERTY --configuration=$CONFIG_NAME 2>/dev/null)

if [ "$NEW_VALUE" = "$VALUE" ]; then
    echo "✓ Property verified: $FULL_PROPERTY = $NEW_VALUE"
else
    echo "⚠️  Verification failed: expected '$VALUE', got '$NEW_VALUE'"
fi
```
Confirms the property was set correctly.

### 6. Show related properties
```bash
echo -e "\n=== Related Properties ==="
case $FULL_PROPERTY in
    project)
        echo "You may also want to set:"
        echo "  gcloud config set compute/zone ZONE"
        echo "  gcloud config set compute/region REGION"
        ;;
    compute/zone)
        REGION=$(echo $VALUE | sed 's/-[a-z]$//')
        echo "Related region: $REGION"
        echo "Set with: gcloud config set compute/region $REGION"
        ;;
    compute/region)
        echo "Available zones in $VALUE:"
        gcloud compute zones list --filter="region:$VALUE" --format="value(name)" 2>/dev/null | sed 's/^/  /'
        ;;
esac
```
Suggests related configuration options.

### 7. Show current configuration summary
```bash
echo -e "\n=== Current Configuration Summary ==="
echo "Configuration: $CONFIG_NAME"
echo "Account: $(gcloud config get-value account --configuration=$CONFIG_NAME 2>/dev/null || echo 'Not set')"
echo "Project: $(gcloud config get-value project --configuration=$CONFIG_NAME 2>/dev/null || echo 'Not set')"
echo "Region: $(gcloud config get-value compute/region --configuration=$CONFIG_NAME 2>/dev/null || echo 'Not set')"
echo "Zone: $(gcloud config get-value compute/zone --configuration=$CONFIG_NAME 2>/dev/null || echo 'Not set')"
```
Displays updated configuration summary.

### 8. Provide usage examples
```bash
echo -e "\n=== Usage Examples ==="
cat << 'EOF'
# Common properties to set
gcloud config set project my-project-id
gcloud config set compute/zone us-central1-a
gcloud config set compute/region us-central1
gcloud config set account user@example.com

# Set for specific configuration
gcloud config set project prod-project --configuration=production

# Unset a property
gcloud config unset compute/zone

# Create and configure new configuration
gcloud config configurations create staging
gcloud config set project staging-project
gcloud config set compute/region europe-west1
EOF
```
Shows common configuration patterns.

## Validation
- Property is set to the specified value
- Value format matches expected pattern
- Configuration file is updated
- New value is returned by get-value

## Error Handling
- **"Project not found"** - Verify project ID and access
- **"Invalid property"** - Check property name format
- **"Permission denied"** - Check ~/.config/gcloud permissions
- **"Configuration not found"** - Verify configuration name

## Safety Notes
- Changes persist across sessions
- Affects all gcloud commands using this configuration
- Project changes may affect billing
- Some properties require specific formats
- Use configurations to separate environments

## Examples
- **Set default project**
  ```
  gcp-config-set project my-project-123
  ```
  Sets the default project for all commands

- **Set compute zone**
  ```
  gcp-config-set compute/zone us-central1-a
  ```
  Sets default zone for compute operations

- **Set region**
  ```
  gcp-config-set compute/region europe-west1
  ```
  Sets default region

- **Set for specific configuration**
  ```
  gcp-config-set project prod-project production
  ```
  Sets project in 'production' configuration
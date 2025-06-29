# GCP IAM Roles List

## Purpose
List available IAM roles in Google Cloud with their permissions and descriptions

## Context
Use to discover available roles for granting permissions to users, service accounts, or groups. Shows both predefined Google Cloud roles and custom roles created in your organization or project.

## Parameters
- `$ARGUMENTS` - Optional filters or format options
  - Optional
  - Example: `--filter="name:roles/compute"`, `--limit=50`
- `$ROLE_TYPE` - Type of roles to list
  - Optional
  - Default: `all`
  - Options: `predefined`, `custom`, `all`

## Steps

### 1. Set role type filter
```bash
ROLE_TYPE=${ROLE_TYPE:-all}
PROJECT=$(gcloud config get-value project)

echo "=== IAM Roles ==="
echo "Project: $PROJECT"

case $ROLE_TYPE in
    predefined)
        FILTER="name:roles/*"
        echo "Type: Predefined roles only"
        ;;
    custom)
        FILTER="name:projects/$PROJECT/roles/* OR name:organizations/*/roles/*"
        echo "Type: Custom roles only"
        ;;
    *)
        FILTER=""
        echo "Type: All roles"
        ;;
esac
```
Configures filters based on role type.

### 2. List roles with basic information
```bash
echo -e "\n=== Available Roles ==="
if [ -n "$FILTER" ]; then
    gcloud iam roles list --filter="$FILTER" $ARGUMENTS
else
    gcloud iam roles list $ARGUMENTS
fi
```
Lists roles matching the criteria.

### 3. Show role categories
```bash
echo -e "\n=== Roles by Service ==="
# Count roles by service prefix
gcloud iam roles list --format="value(name)" --limit=1000 | \
    grep "^roles/" | \
    cut -d'.' -f1 | \
    sed 's/^roles\///' | \
    sort | uniq -c | sort -rn | head -20 | \
    while read count service; do
        printf "%-20s %d roles\n" "$service" "$count"
    done
```
Groups roles by Google Cloud service.

### 4. Search for common role patterns
```bash
echo -e "\n=== Common Role Patterns ==="
echo "Admin roles:"
gcloud iam roles list --filter="name:*/admin" --format="table(name,title)" --limit=10

echo -e "\nViewer roles:"
gcloud iam roles list --filter="name:*/viewer" --format="table(name,title)" --limit=10

echo -e "\nEditor roles:"
gcloud iam roles list --filter="name:*/editor" --format="table(name,title)" --limit=10
```
Shows commonly used role patterns.

### 5. List custom roles in project
```bash
echo -e "\n=== Custom Roles in Project ==="
CUSTOM_COUNT=$(gcloud iam roles list --project=$PROJECT --format="value(name)" | wc -l)

if [ $CUSTOM_COUNT -gt 0 ]; then
    gcloud iam roles list --project=$PROJECT --format="table(
        name,
        title,
        stage,
        etag
    )"
    echo "Total custom roles: $CUSTOM_COUNT"
else
    echo "No custom roles found in project"
fi
```
Displays project-specific custom roles.

### 6. Show basic roles
```bash
echo -e "\n=== Basic Roles (Use with Caution) ==="
cat << 'EOF'
roles/viewer    - Read access to all resources
roles/editor    - Edit access to all resources  
roles/owner     - Full access including IAM management

⚠️  Warning: Basic roles grant broad permissions.
   Use predefined or custom roles for better security.
EOF
```
Lists basic roles with security warning.

### 7. Display role details for examples
```bash
echo -e "\n=== Example Role Details ==="
# Show details for a few common roles
for role in "roles/compute.viewer" "roles/storage.admin" "roles/iam.securityReviewer"; do
    if gcloud iam roles describe $role &>/dev/null; then
        echo -e "\n--- $role ---"
        gcloud iam roles describe $role --format="yaml(title,description,includedPermissions[0:5])"
        PERM_COUNT=$(gcloud iam roles describe $role --format="value(includedPermissions[].flatten())" | wc -l)
        echo "Total permissions: $PERM_COUNT"
    fi
done
```
Shows detailed information for example roles.

### 8. Search roles by permission
```bash
echo -e "\n=== Search Roles by Permission ==="
echo "Example: Roles that include 'compute.instances.list' permission:"

# This is a sample - searching all roles would be slow
SAMPLE_PERMISSION="compute.instances.list"
FOUND_ROLES=0

for role in $(gcloud iam roles list --format="value(name)" --limit=50); do
    if gcloud iam roles describe $role --format="value(includedPermissions[])" 2>/dev/null | grep -q "^$SAMPLE_PERMISSION$"; then
        echo "  - $role"
        FOUND_ROLES=$((FOUND_ROLES + 1))
    fi
done

echo "Found $FOUND_ROLES roles with this permission (sample of 50 checked)"
```
Demonstrates searching roles by specific permissions.

### 9. Provide role assignment examples
```bash
echo -e "\n=== How to Assign Roles ==="
cat << 'EOF'
# Grant role to user
gcloud projects add-iam-policy-binding PROJECT_ID \
    --member="user:email@example.com" \
    --role="roles/compute.viewer"

# Grant role to service account
gcloud projects add-iam-policy-binding PROJECT_ID \
    --member="serviceAccount:sa-name@PROJECT_ID.iam.gserviceaccount.com" \
    --role="roles/storage.admin"

# Grant role to group
gcloud projects add-iam-policy-binding PROJECT_ID \
    --member="group:group@example.com" \
    --role="roles/editor"

# Create custom role
gcloud iam roles create myCustomRole \
    --project=PROJECT_ID \
    --title="My Custom Role" \
    --description="Description of custom role" \
    --permissions=compute.instances.list,compute.instances.get
EOF
```
Provides examples of role assignment.

## Validation
- Roles are listed successfully
- Role names follow correct format
- Permissions are displayed for detailed views
- Custom roles show correct project

## Error Handling
- **"Permission denied"** - Need iam.roles.list permission
- **"Invalid filter"** - Check filter syntax
- **"Role not found"** - Role may not exist or wrong name format

## Safety Notes
- This is a read-only operation
- Some roles may have sensitive permissions
- Basic roles (viewer/editor/owner) grant broad access
- Custom roles may have organization-specific permissions

## Examples
- **List all roles**
  ```
  gcp-iam-roles-list
  ```
  Shows all available roles

- **List compute roles only**
  ```
  gcp-iam-roles-list "--filter=name:roles/compute.*"
  ```
  Shows only Compute Engine related roles

- **List custom roles**
  ```
  gcp-iam-roles-list "" custom
  ```
  Shows only custom roles in project

- **Search for admin roles**
  ```
  gcp-iam-roles-list "--filter=title:Admin"
  ```
  Shows roles with "Admin" in title
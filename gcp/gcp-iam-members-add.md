# GCP IAM Members Add

## Purpose
Grant an IAM role to a user, service account, or group for a project or resource

## Context
Use to provide access permissions by binding a member (user, service account, or group) to a specific role. This is how you control who can do what in your Google Cloud project.

## Parameters
- `$MEMBER` - The member to grant access to
  - Required
  - Format: `user:email@example.com`, `serviceAccount:sa@project.iam.gserviceaccount.com`, `group:group@example.com`
- `$ROLE` - The role to grant
  - Required
  - Example: `roles/viewer`, `roles/compute.admin`
- `$RESOURCE` - The resource to grant access to
  - Optional
  - Default: Current project
  - Example: `projects/my-project`, `gs://my-bucket`

## Steps

### 1. Validate member format
```bash
# Validate member format
if [[ ! "$MEMBER" =~ ^(user|serviceAccount|group|domain|allUsers|allAuthenticatedUsers):.+ ]]; then
    echo "Error: Invalid member format"
    echo "Valid formats:"
    echo "  user:email@example.com"
    echo "  serviceAccount:name@project.iam.gserviceaccount.com"
    echo "  group:group@example.com"
    echo "  domain:example.com"
    exit 1
fi

MEMBER_TYPE=$(echo $MEMBER | cut -d: -f1)
MEMBER_ID=$(echo $MEMBER | cut -d: -f2-)
```
Ensures member follows correct format.

### 2. Validate role
```bash
# Check if role exists
if ! gcloud iam roles describe $ROLE &>/dev/null; then
    # Try with custom role format
    PROJECT=$(gcloud config get-value project)
    if ! gcloud iam roles describe $ROLE --project=$PROJECT &>/dev/null; then
        echo "Error: Role '$ROLE' not found"
        echo "List available roles with: gcp-iam-roles-list"
        exit 1
    fi
fi

echo "Role: $ROLE"
ROLE_TITLE=$(gcloud iam roles describe $ROLE --format="value(title)" 2>/dev/null || echo $ROLE)
echo "Title: $ROLE_TITLE"
```
Verifies the role exists.

### 3. Determine resource scope
```bash
if [ -z "$RESOURCE" ]; then
    # Default to current project
    PROJECT=$(gcloud config get-value project)
    RESOURCE="projects/$PROJECT"
    RESOURCE_TYPE="project"
    echo "Resource: $RESOURCE (current project)"
else
    # Determine resource type
    if [[ "$RESOURCE" =~ ^projects/ ]]; then
        RESOURCE_TYPE="project"
    elif [[ "$RESOURCE" =~ ^gs:// ]]; then
        RESOURCE_TYPE="bucket"
    elif [[ "$RESOURCE" =~ ^organizations/ ]]; then
        RESOURCE_TYPE="organization"
    else
        echo "Error: Unsupported resource type"
        exit 1
    fi
    echo "Resource: $RESOURCE"
fi
```
Identifies the resource to modify.

### 4. Check current policy
```bash
echo -e "\n=== Current Access ==="
case $RESOURCE_TYPE in
    project)
        CURRENT_ROLES=$(gcloud projects get-iam-policy ${RESOURCE#projects/} \
            --flatten="bindings[].members" \
            --filter="bindings.members:$MEMBER" \
            --format="value(bindings.role)" 2>/dev/null)
        ;;
    bucket)
        CURRENT_ROLES=$(gsutil iam get $RESOURCE | \
            jq -r ".bindings[] | select(.members[] | contains(\"$MEMBER\")) | .role" 2>/dev/null)
        ;;
esac

if [ -n "$CURRENT_ROLES" ]; then
    echo "Member already has roles:"
    echo "$CURRENT_ROLES" | sed 's/^/  - /'
else
    echo "Member has no existing roles on this resource"
fi
```
Shows existing permissions for the member.

### 5. Add the IAM binding
```bash
echo -e "\n=== Adding IAM Binding ==="
echo "Granting $ROLE to $MEMBER..."

case $RESOURCE_TYPE in
    project)
        gcloud projects add-iam-policy-binding ${RESOURCE#projects/} \
            --member="$MEMBER" \
            --role="$ROLE"
        ;;
    bucket)
        gsutil iam ch $MEMBER:$ROLE $RESOURCE
        ;;
    organization)
        gcloud organizations add-iam-policy-binding ${RESOURCE#organizations/} \
            --member="$MEMBER" \
            --role="$ROLE"
        ;;
esac

if [ $? -eq 0 ]; then
    echo "✓ Role granted successfully"
else
    echo "✗ Failed to grant role"
    exit 1
fi
```
Adds the IAM policy binding.

### 6. Verify the change
```bash
echo -e "\n=== Verification ==="
case $RESOURCE_TYPE in
    project)
        NEW_ROLES=$(gcloud projects get-iam-policy ${RESOURCE#projects/} \
            --flatten="bindings[].members" \
            --filter="bindings.members:$MEMBER" \
            --format="value(bindings.role)" 2>/dev/null)
        ;;
    bucket)
        NEW_ROLES=$(gsutil iam get $RESOURCE | \
            jq -r ".bindings[] | select(.members[] | contains(\"$MEMBER\")) | .role" 2>/dev/null)
        ;;
esac

if echo "$NEW_ROLES" | grep -q "^$ROLE$"; then
    echo "✓ Role verified: $MEMBER now has $ROLE"
else
    echo "⚠️  Verification failed - role may not be active yet"
fi
```
Confirms the role was added.

### 7. Show effective permissions summary
```bash
echo -e "\n=== Effective Permissions ==="
echo "Member: $MEMBER"
echo "New role: $ROLE"

# Show key permissions for common roles
case $ROLE in
    *viewer*)
        echo "Key permissions: Read-only access"
        ;;
    *admin*)
        echo "Key permissions: Full administrative access"
        ;;
    *editor*)
        echo "Key permissions: Read and write access"
        ;;
    *)
        # Show first few permissions
        PERMS=$(gcloud iam roles describe $ROLE --format="value(includedPermissions[0:5].flatten())" 2>/dev/null)
        if [ -n "$PERMS" ]; then
            echo "Key permissions:"
            echo "$PERMS" | head -5 | sed 's/^/  - /'
            TOTAL_PERMS=$(gcloud iam roles describe $ROLE --format="value(includedPermissions[].flatten())" 2>/dev/null | wc -l)
            echo "  ... and $((TOTAL_PERMS - 5)) more"
        fi
        ;;
esac
```
Shows summary of granted permissions.

### 8. Provide usage instructions
```bash
echo -e "\n=== Next Steps ==="
case $MEMBER_TYPE in
    user)
        echo "User $MEMBER_ID can now access the resource with $ROLE permissions"
        echo "They may need to refresh their browser or re-authenticate"
        ;;
    serviceAccount)
        echo "Service account $MEMBER_ID now has $ROLE permissions"
        echo "Applications using this service account will automatically have access"
        ;;
    group)
        echo "All members of group $MEMBER_ID now have $ROLE permissions"
        echo "Changes may take a few minutes to propagate"
        ;;
esac

echo -e "\nTo remove this binding later:"
echo "gcloud projects remove-iam-policy-binding ${RESOURCE#projects/} \\"
echo "    --member=\"$MEMBER\" --role=\"$ROLE\""
```
Provides follow-up guidance.

## Validation
- Member format is correct
- Role exists and is valid
- Binding is added successfully
- Member appears in policy with correct role

## Error Handling
- **"Invalid member"** - Check member format (type:identifier)
- **"Role not found"** - Verify role name or use gcp-iam-roles-list
- **"Permission denied"** - Need resourcemanager.projects.setIamPolicy
- **"Invalid resource"** - Check resource format and existence

## Safety Notes
- Changes take effect immediately
- Some roles grant significant permissions
- Avoid using basic roles (owner/editor/viewer) when possible
- allUsers and allAuthenticatedUsers make resources public
- Keep audit logs of IAM changes

## Examples
- **Grant viewer to user**
  ```
  gcp-iam-members-add "user:john@example.com" "roles/viewer"
  ```
  Grants project viewer role to user

- **Grant storage admin to service account**
  ```
  gcp-iam-members-add "serviceAccount:app@project.iam.gserviceaccount.com" "roles/storage.admin"
  ```
  Grants storage admin to service account

- **Grant custom role**
  ```
  gcp-iam-members-add "group:devs@example.com" "projects/my-project/roles/customDeveloper"
  ```
  Grants custom role to group

- **Grant bucket access**
  ```
  gcp-iam-members-add "user:alice@example.com" "roles/storage.objectViewer" "gs://my-bucket"
  ```
  Grants bucket-specific access
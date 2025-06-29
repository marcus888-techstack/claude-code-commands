# GCP Billing Summary

## Purpose
Display billing summary, current costs, and budget information for Google Cloud projects

## Context
Use to monitor cloud spending, view cost breakdowns by service, and check budget alerts. Essential for cost management and identifying expensive resources or unexpected charges.

## Parameters
- `$BILLING_ACCOUNT` - Billing account ID to analyze
  - Optional
  - Default: Account linked to current project
  - Example: `01234-56789-ABCDEF`
- `$TIME_PERIOD` - Time period for cost analysis
  - Optional
  - Default: `current_month`
  - Options: `current_month`, `last_month`, `last_7_days`, `last_30_days`

## Steps

### 1. Get billing account information
```bash
PROJECT=$(gcloud config get-value project)
echo "=== Billing Summary ==="
echo "Project: $PROJECT"

# Get billing account for project
if [ -z "$BILLING_ACCOUNT" ]; then
    BILLING_INFO=$(gcloud beta billing projects describe $PROJECT --format="value(billingAccountName)" 2>/dev/null)
    if [ -n "$BILLING_INFO" ]; then
        BILLING_ACCOUNT=$(basename $BILLING_INFO)
        echo "Billing Account: $BILLING_ACCOUNT"
    else
        echo "Error: No billing account linked to project"
        echo "Link billing with: gcloud beta billing projects link $PROJECT --billing-account=ACCOUNT_ID"
        exit 1
    fi
else
    echo "Billing Account: $BILLING_ACCOUNT"
fi
```
Identifies the billing account to analyze.

### 2. Check billing account status
```bash
echo -e "\n=== Billing Account Status ==="
ACCOUNT_INFO=$(gcloud beta billing accounts describe $BILLING_ACCOUNT 2>/dev/null)
if [ $? -eq 0 ]; then
    echo "$ACCOUNT_INFO" | grep -E "(displayName|open):" | sed 's/^/  /'
    
    IS_OPEN=$(echo "$ACCOUNT_INFO" | grep "open:" | grep -q "true" && echo "Active" || echo "Closed")
    echo "  Status: $IS_OPEN"
else
    echo "⚠️  Unable to access billing account details"
    echo "You may need billing.accounts.get permission"
fi
```
Shows billing account status and details.

### 3. Display current month costs
```bash
echo -e "\n=== Current Month Costs ==="
# Set date range based on TIME_PERIOD
case ${TIME_PERIOD:-current_month} in
    current_month)
        START_DATE=$(date +%Y-%m-01)
        END_DATE=$(date +%Y-%m-%d)
        PERIOD_DESC="Month to date"
        ;;
    last_month)
        START_DATE=$(date -d "last month" +%Y-%m-01 2>/dev/null || date -v-1m +%Y-%m-01)
        END_DATE=$(date -d "last month + 1 month - 1 day" +%Y-%m-%d 2>/dev/null || date -v-1d +%Y-%m-%d)
        PERIOD_DESC="Last month"
        ;;
    last_7_days)
        START_DATE=$(date -d "7 days ago" +%Y-%m-%d 2>/dev/null || date -v-7d +%Y-%m-%d)
        END_DATE=$(date +%Y-%m-%d)
        PERIOD_DESC="Last 7 days"
        ;;
    last_30_days)
        START_DATE=$(date -d "30 days ago" +%Y-%m-%d 2>/dev/null || date -v-30d +%Y-%m-%d)
        END_DATE=$(date +%Y-%m-%d)
        PERIOD_DESC="Last 30 days"
        ;;
esac

echo "Period: $PERIOD_DESC ($START_DATE to $END_DATE)"
echo ""
echo "Note: For detailed cost breakdowns, use the Cloud Console:"
echo "https://console.cloud.google.com/billing/$BILLING_ACCOUNT/reports"
```
Sets up time period for analysis.

### 4. Show cost estimate
```bash
echo -e "\n=== Cost Estimates ==="
cat << 'EOF'
To view current costs in Cloud Console:
1. Go to Billing > Reports
2. Filter by project, service, or SKU
3. Set date range for analysis

Common cost drivers:
- Compute Engine: VM instances running 24/7
- Cloud Storage: Large data volumes
- Network: Egress traffic charges
- BigQuery: Query processing
- Cloud SQL: Database instances
EOF
```
Provides guidance on viewing costs.

### 5. List budgets and alerts
```bash
echo -e "\n=== Budgets and Alerts ==="
# Check if billing budgets API is enabled
if gcloud services list --enabled --filter="name:billingbudgets.googleapis.com" --format="value(name)" | grep -q billingbudgets; then
    BUDGET_COUNT=$(gcloud billing budgets list --billing-account=$BILLING_ACCOUNT --format="value(name)" 2>/dev/null | wc -l)
    
    if [ $BUDGET_COUNT -gt 0 ]; then
        echo "Active budgets: $BUDGET_COUNT"
        gcloud billing budgets list --billing-account=$BILLING_ACCOUNT --format="table(
            displayName,
            budgetFilter.projects[0],
            amount.specifiedAmount.currencyCode,
            amount.specifiedAmount.units
        )" 2>/dev/null
    else
        echo "No budgets configured"
        echo "Create budget: https://console.cloud.google.com/billing/$BILLING_ACCOUNT/budgets"
    fi
else
    echo "Billing Budgets API not enabled"
    echo "Enable with: gcloud services enable billingbudgets.googleapis.com"
fi
```
Shows configured budgets and alerts.

### 6. Show top spending projects
```bash
echo -e "\n=== Projects in Billing Account ==="
gcloud beta billing projects list --billing-account=$BILLING_ACCOUNT --format="table(
    projectId,
    billingEnabled
)" 2>/dev/null | head -20

PROJECT_COUNT=$(gcloud beta billing projects list --billing-account=$BILLING_ACCOUNT --format="value(projectId)" 2>/dev/null | wc -l)
echo -e "\nTotal projects: $PROJECT_COUNT"
```
Lists projects using this billing account.

### 7. Provide cost optimization tips
```bash
echo -e "\n=== Cost Optimization Tips ==="
cat << 'EOF'
Quick wins for reducing costs:

1. Compute Engine:
   - Stop unused instances
   - Use preemptible VMs for batch jobs
   - Right-size instances
   - Delete unattached disks

2. Cloud Storage:
   - Set lifecycle policies
   - Use appropriate storage classes
   - Delete old backups

3. Network:
   - Minimize cross-region traffic
   - Use Cloud CDN for static content
   - Optimize data transfer

4. General:
   - Set up budget alerts
   - Use committed use discounts
   - Enable recommendations
   - Regular cost reviews

View recommendations:
https://console.cloud.google.com/home/recommendations
EOF
```
Provides cost optimization guidance.

### 8. Show billing export setup
```bash
echo -e "\n=== Billing Export ==="
echo "To enable detailed billing analysis:"
echo ""
echo "1. Export to BigQuery (recommended):"
echo "   - Go to Billing > Billing export"
echo "   - Configure BigQuery export"
echo "   - Query costs with SQL"
echo ""
echo "2. Export to Cloud Storage:"
echo "   - CSV files for spreadsheet analysis"
echo "   - Daily export of billing data"
echo ""
echo "3. Use Cloud Billing API:"
echo "   - Programmatic access to billing data"
echo "   - Integration with monitoring tools"
```
Shows how to export billing data.

## Validation
- Billing account is linked to project
- Billing account is active
- User has necessary permissions
- APIs are enabled for budget queries

## Error Handling
- **"No billing account"** - Link billing account to project
- **"Permission denied"** - Need billing.accounts.get permission
- **"API not enabled"** - Enable required APIs
- **"Invalid date range"** - Check date format

## Safety Notes
- Read-only operation for billing data
- Requires billing permissions to view
- Actual costs may have delays
- Estimates are not guarantees
- Check Cloud Console for real-time data

## Examples
- **View current month costs**
  ```
  gcp-billing-summary
  ```
  Shows billing for current month

- **View last month costs**
  ```
  gcp-billing-summary "" last_month
  ```
  Shows previous month's billing

- **View specific billing account**
  ```
  gcp-billing-summary "012345-6789AB-CDEF01"
  ```
  Shows billing for specific account

- **View last 7 days**
  ```
  gcp-billing-summary "" last_7_days
  ```
  Shows recent spending trends
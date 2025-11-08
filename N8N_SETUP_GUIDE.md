# n8n Setup Guide - Salesforce to Supabase Sync

This guide explains how to set up the n8n workflow to sync Salesforce Users to Supabase for the Developer Agent typeahead feature.

## Overview

**Data Flow**: Salesforce → n8n (every 30 min) → Supabase → Form Typeahead

Each developer has their own Salesforce instance, so you'll need one workflow per developer.

---

## Prerequisites

1. n8n instance (cloud or self-hosted)
2. Salesforce OAuth credentials for each developer
3. Supabase project credentials (API URL + Service Role Key)
4. Supabase table `developer_agents` already created

---

## Step 1: Import the Workflow Template

1. Open n8n
2. Click **Workflows** → **Add Workflow**
3. Click the **⋯** menu → **Import from File**
4. Select `n8n-salesforce-sync-workflow.json`
5. The workflow will open with 4 nodes:
   - Schedule Trigger (every 30 minutes)
   - Salesforce - Get Active Users
   - Transform Data
   - Supabase - Upsert Agents

---

## Step 2: Configure Salesforce Credentials

### For Abra (Repeat for each developer)

1. Click the **Salesforce - Get Active Users** node
2. Click **Credentials** → **Create New**
3. Choose **Salesforce OAuth2 API**
4. Fill in:
   - **Name**: `Salesforce - Abra`
   - **Environment**: Production (or Sandbox if testing)
   - **Client ID**: Get from Salesforce Connected App
   - **Client Secret**: Get from Salesforce Connected App
5. Click **Connect my account**
6. Authenticate with Abra's Salesforce
7. Save credential

### SOQL Query (Already configured)

```sql
SELECT Id, Name
FROM User
WHERE IsActive = true
ORDER BY Name
```

This query gets:
- **Id**: Salesforce User ID (used as `salesforce_id`)
- **Name**: Full name (used as `agent_name`)

---

## Step 3: Configure Transform Data Node

1. Click the **Transform Data** node
2. Update the `developerKey` variable:

```javascript
const developerKey = 'abra'; // Change for each developer
```

**Developer Keys**:
- Abra → `'abra'`
- Emaar → `'emaar'`
- DAMAC → `'damac'`
- Nakheel → `'nakheel'`

This code transforms Salesforce data into Supabase format:

```javascript
{
  salesforce_id: "005Dn000001234",  // From Salesforce Id
  developer_key: "abra",            // Hardcoded per workflow
  agent_name: "Yassir El Ghazi",    // From Salesforce Name
  is_active: true                   // Always true for active users
}
```

---

## Step 4: Configure Supabase Credentials

1. Click the **Supabase - Upsert Agents** node
2. Click **Credentials** → **Create New**
3. Choose **Supabase API**
4. Fill in:
   - **Name**: `Supabase - Client Submissions`
   - **Host**: `https://kosbeatukcmzcbuhqexh.supabase.co`
   - **Service Role Secret**: Get from Supabase Dashboard → Settings → API
5. Save credential

### Upsert Configuration (Already set)

- **Operation**: Upsert
- **Table**: `developer_agents`
- **Match On**: `salesforce_id,developer_key` (prevents duplicates)
- **Fields**:
  - `salesforce_id` ← `{{ $json.salesforce_id }}`
  - `developer_key` ← `{{ $json.developer_key }}`
  - `agent_name` ← `{{ $json.agent_name }}`
  - `is_active` ← `{{ $json.is_active }}`
  - `updated_at` ← `{{ $now }}`

---

## Step 5: Test the Workflow

1. Click **Execute Workflow** (top right)
2. Check each node output:
   - **Salesforce**: Should show list of Users
   - **Transform**: Should show transformed data with `developer_key`
   - **Supabase**: Should confirm upsert success
3. Verify in Supabase:
   - Go to Table Editor → `developer_agents`
   - Should see agents with `developer_key = 'abra'`

---

## Step 6: Activate the Workflow

1. Toggle **Active** (top right)
2. The workflow will now run every 30 minutes automatically
3. To run manually: Click **Execute Workflow**

---

## Step 7: Clone for Other Developers

For each new developer (Emaar, DAMAC, Nakheel):

1. **Duplicate the workflow**:
   - Click **⋯** → **Duplicate**
   - Rename to "Sync Salesforce Users to Supabase - Emaar"

2. **Update Salesforce credentials**:
   - Connect to Emaar's Salesforce

3. **Update Transform Data node**:
   - Change `developerKey = 'emaar'`

4. **Test and activate**

---

## Troubleshooting

### Issue: "Authentication failed"

**Solution**: Re-authenticate Salesforce credentials
- Salesforce sessions can expire
- Go to Credentials → Edit → Reconnect

### Issue: "Table developer_agents does not exist"

**Solution**: Make sure Supabase table is created first
- Run the migration in Supabase
- Check table exists in Table Editor

### Issue: "Duplicate key value violates unique constraint"

**Solution**: This is expected!
- The upsert will update existing records
- The unique constraint on (salesforce_id, developer_key) prevents true duplicates

### Issue: No data showing in Supabase

**Solution**: Check the workflow execution
- Go to **Executions** tab in n8n
- Look for errors in failed executions
- Verify SOQL query returns data

---

## Sync Frequency

**Current**: Every 30 minutes

**To change**:
1. Click Schedule Trigger node
2. Update **Minutes Interval**
3. Options: 15 min, 30 min, 1 hour, 6 hours, daily

**Recommendation**: 30 minutes is good balance
- Agents don't join/leave every minute
- Low Salesforce API usage
- Form always has relatively fresh data

---

## API Usage

### Salesforce API Limits

- **Query**: Each sync = 1 SOQL query
- **48 syncs/day** (every 30 min) = 48 API calls
- Well within Salesforce limits (even on Professional Edition)

### Supabase Limits

- **Free tier**: 500MB database, unlimited API requests
- **Paid tier** ($10/mo): 8GB database, unlimited API requests
- This table will use < 1MB (even with 1000 agents)

---

## Data Schema Reference

```sql
Table: developer_agents

salesforce_id   | text      | "005Dn000001234"
developer_key   | text      | "abra"
agent_name      | text      | "Yassir El Ghazi"
is_active       | boolean   | true
created_at      | timestamp | "2025-01-08 08:30:00"
updated_at      | timestamp | "2025-01-08 09:00:00"
```

---

## Next Steps

Once all workflows are set up and syncing:

1. ✅ Agents will appear in form typeahead
2. ✅ When form is submitted, `developerAgentId` is sent to n8n
3. ✅ Use `developerAgentId` to assign Lead/Contact in Salesforce:
   - `OwnerId = developerAgentId`

---

## Support

For questions:
- **n8n**: https://docs.n8n.io
- **Supabase**: https://supabase.com/docs
- **Salesforce SOQL**: https://developer.salesforce.com/docs/atlas.en-us.soql_sosl.meta

---

**Created**: January 2025
**Last Updated**: January 2025

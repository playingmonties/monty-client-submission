# Monty Client Submission Form

A scalable, white-labeled booking form system for real estate developers. Built with clean HTML, CSS, and vanilla JavaScript.

## 🎯 Overview

This system allows you to create custom booking forms for multiple real estate developers using a single codebase. All forms are branded under "Monty" and route submissions to n8n based on URL parameters.

## 🚀 Quick Start

### Local Development

1. Start a local server:
```bash
python3 -m http.server 8000
```

2. Open in browser:
```
http://localhost:8000/index.html?dev=abra
```

### Production Deployment

Simply upload all files to your web server. No build process required!

## 📁 File Structure

```
Client_Submission/
├── index.html                           # Main form page
├── style.css                            # Styling (blue-grey theme)
├── developers.json                      # Developer configurations
├── monty_logo_blue.png                 # Monty branding logo
├── n8n-salesforce-sync-workflow.json   # n8n workflow template (NEW)
├── N8N_SETUP_GUIDE.md                  # n8n setup instructions (NEW)
├── README.md                            # This file
└── .gitignore                           # Git ignore file
```

## 🔧 How It Works

### URL Parameter System

Each developer gets a unique URL parameter that determines routing:

```
https://yoursite.com/index.html?dev=abra     → Abra Properties
https://yoursite.com/index.html?dev=emaar    → Emaar Properties
https://yoursite.com/index.html?dev=nakheel  → Nakheel Properties
```

### Form Flow

1. User opens form with `?dev=DEVELOPER_KEY`
2. JavaScript loads developer config from `developers.json`
3. Form displays with Monty branding
4. On submit, all data POSTs to the n8n webhook
5. n8n receives `developer` field to identify which developer

## 📝 Adding a New Developer

**Super simple - just edit `developers.json`:**

```json
{
  "newdeveloper": {
    "name": "New Developer Name",
    "webhook": "https://thomasmccone.app.n8n.cloud/webhook/monty_client_form"
  }
}
```

Then share: `https://yoursite.com/index.html?dev=newdeveloper`

**That's it!** No code changes, no webhooks to create (single webhook for all).

## 🎨 Design & Styling

- **Color Scheme**: Blue-grey palette (#36454f, #708090, #536878)
- **Font**: System fonts (Apple SF Pro, Segoe UI fallbacks)
- **Responsive**: Mobile-optimized with touch-friendly inputs
- **Theme**: Clean, minimal, professional

### Key Design Elements

- Logo: 60px (desktop), 48px (mobile)
- Submit button: Dark blue (#36454f)
- Borders: 1.5px slate grey (#708090)
- Card: White with soft shadow
- Background: Light grey (#f8f9fa)

## 📤 Form Fields

### Agent Details (External Broker/Agency)
- Agency Name (required)
- Agent Full Name (required)

### Developer Agent (Internal Employee)
- Select Developer Agent (required) - **NEW!**
  - Typeahead search field
  - Pulls data from Supabase
  - Auto-synced from Salesforce every 30 minutes
  - Submits Salesforce User ID for automatic assignment

### Client Details
- Full Name (required)
- Mobile Number (required)
- Email Address (required)
- Home Address (required)

### Documents
- Client Passport (optional)
- Client Emirates ID (optional)

**Note**: Both documents are optional to accommodate different client situations (tourists, new expats, etc.)

## 🏗️ Developer Agent Integration (NEW)

### Architecture Overview

The form now includes a **Developer Agent** typeahead field that pulls internal employees from each developer's Salesforce CRM.

**Data Flow:**
```
Salesforce (each developer) → n8n (sync every 30 min) → Supabase → Form Typeahead
```

### Components

1. **Salesforce** - Source of truth for developer agents (internal employees)
   - Each developer has their own Salesforce instance
   - Stores active Users with IsActive = true

2. **n8n Workflow** - Syncs Salesforce → Supabase
   - One workflow per developer (Abra, Emaar, DAMAC, Nakheel)
   - Runs every 30 minutes
   - SOQL Query: `SELECT Id, Name FROM User WHERE IsActive = true`
   - See `N8N_SETUP_GUIDE.md` for setup instructions

3. **Supabase Table** - Cache layer for fast lookups
   - Project: `Client_Submissions_Monty`
   - Table: `developer_agents`
   - Fields: `salesforce_id`, `developer_key`, `agent_name`, `is_active`
   - Region: ap-south-1 (Mumbai - closest to Dubai)

4. **Form Typeahead** - User-facing search field
   - Queries Supabase filtered by developer
   - Fast client-side search
   - Shows agent names
   - Submits Salesforce User ID

### Supabase Table Structure

```sql
CREATE TABLE developer_agents (
  id uuid PRIMARY KEY,
  salesforce_id text NOT NULL,        -- Salesforce User ID
  developer_key text NOT NULL,        -- "abra", "emaar", "damac", "nakheel"
  agent_name text NOT NULL,           -- "Yassir El Ghazi"
  is_active boolean DEFAULT true,
  created_at timestamp,
  updated_at timestamp,
  UNIQUE(salesforce_id, developer_key)
);
```

### Why This Architecture?

- ✅ **Fast**: Form loads instantly (< 100ms) - no waiting for Salesforce API
- ✅ **Scalable**: Add new developers = just add new n8n workflow
- ✅ **Reliable**: Form works even if one Salesforce is down
- ✅ **No rate limits**: Salesforce only queried every 30 min, not on every form load
- ✅ **Multiple CRMs**: Each developer's Salesforce syncs independently

### Setup Guide

See **`N8N_SETUP_GUIDE.md`** for detailed instructions on:
- Importing the n8n workflow template
- Connecting Salesforce credentials
- Configuring Supabase
- Testing and activating the sync

## 🔗 n8n Integration

### Webhook URL
All forms POST to: `https://thomasmccone.app.n8n.cloud/webhook/monty_client_form`

### Data Format
- **Content-Type**: `multipart/form-data`
- **Files**: Binary data (only sent if uploaded)
- **Developer Field**: String (e.g., "Abra Properties")

### What n8n Receives

```javascript
{
  // External Agent fields
  agencyName: "ABC Realty",
  agentName: "John Doe",

  // Developer Agent fields (NEW!)
  developerAgent: "Yassir El Ghazi",           // Display name
  developerAgentId: "005Dn000001234",          // Salesforce User ID ⭐

  // Client fields
  clientName: "Jane Smith",
  clientMobile: "+971501234567",
  clientEmail: "jane@example.com",
  clientAddress: "123 Dubai Marina",

  // Developer routing
  developer: "Abra Properties",

  // Files (binary data, only if uploaded)
  clientPassport: [File],
  clientEmiratesId: [File]
}
```

**Using the Salesforce User ID:**
When creating a Lead/Contact in Salesforce, use `developerAgentId` to automatically assign ownership:

```javascript
{
  FirstName: "Jane",
  LastName: "Smith",
  Email: "jane@example.com",
  Phone: "+971501234567",
  OwnerId: developerAgentId,  // ⭐ Auto-assigns to correct agent!
  LeadSource: "Monty Form"
}
```

### n8n Routing

Use a **Switch** node or **IF** node to route based on the `developer` field:

```
IF developer = "Abra Properties" → Send to Abra email
IF developer = "Emaar Properties" → Send to Emaar email
IF developer = "Nakheel Properties" → Send to Nakheel email
```

## 🛠️ Technical Details

### File Upload Handling

The form uses smart file handling:
- Only uploads files that are actually selected
- Skips empty file inputs
- Prevents n8n binary data issues

**Code snippet:**
```javascript
fileInputs.forEach(input => {
  if (input.files && input.files.length > 0) {
    formData.append(input.name, input.files[0]);
  }
});
```

### Mobile Optimization

- Input font-size: 16px on mobile (prevents iOS auto-zoom)
- Touch-friendly targets (14px+ padding)
- Proper viewport settings
- Responsive card padding

## 🎯 Key Benefits

### Scalability
- ✅ Add unlimited developers with JSON only
- ✅ Single codebase for all
- ✅ One webhook to manage
- ✅ No build process

### Maintainability
- ✅ Clean, readable code
- ✅ No frameworks or dependencies
- ✅ Simple vanilla JavaScript
- ✅ Well-commented

### White-Label
- ✅ All forms branded as "Monty"
- ✅ Professional appearance
- ✅ Consistent UX
- ✅ Your logo, your brand

## 📱 Browser Support

- Chrome/Edge (latest)
- Safari (latest)
- Firefox (latest)
- Mobile Safari (iOS)
- Chrome Mobile (Android)

## 🔒 Security Notes

- Forms submit via HTTPS (ensure production uses SSL)
- n8n webhook is the only endpoint
- No sensitive data stored client-side
- File types restricted to images and PDFs

## 🐛 Troubleshooting

### Files Not Coming Through to n8n

**Solution**: Make sure files are actually selected. The code only sends files that exist.

### Form Not Loading

**Check**:
1. Is the `?dev=` parameter correct?
2. Does the developer exist in `developers.json`?
3. Is the developers.json file valid JSON?

### Webhook Errors

**Check**:
1. Is the n8n workflow active?
2. Is the webhook URL correct in developers.json?
3. Check n8n execution logs for errors

## 📊 Current Developers

As of now, configured developers:
- Abra Properties (`?dev=abra`)
- Emaar Properties (`?dev=emaar`)
- DAMAC Properties (`?dev=damac`)
- Nakheel Properties (`?dev=nakheel`)

## 🚀 Future Enhancements

Possible improvements:
- Add reCAPTCHA for spam prevention
- Client-side validation messages
- Progress indicators for file uploads
- Email confirmation to client
- Multi-language support

## 📞 Support

For questions or issues:
- Email: thomasmccone@gmail.com
- Review n8n logs for webhook issues
- Check browser console for JavaScript errors

## 📄 License

Proprietary - Monty Real Estate Solutions

---

**Built with ❤️ by Monty**

*Last Updated: January 2025 - Added Developer Agent Integration*
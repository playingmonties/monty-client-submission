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
├── index.html              # Main form page
├── style.css               # Styling (blue-grey theme)
├── developers.json         # Developer configurations
├── monty_logo_blue.png    # Monty branding logo
└── README.md              # This file
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

### Agent Details
- Agency Name (required)
- Agent Full Name (required)

### Client Details
- Full Name (required)
- Mobile Number (required)
- Email Address (required)
- Home Address (required)

### Documents
- Client Passport (optional)
- Client Emirates ID (optional)

**Note**: Both documents are optional to accommodate different client situations (tourists, new expats, etc.)

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
  // Text fields
  agencyName: "ABC Realty",
  agentName: "John Doe",
  clientName: "Jane Smith",
  clientMobile: "+971501234567",
  clientEmail: "jane@example.com",
  clientAddress: "123 Dubai Marina",
  developer: "Abra Properties",

  // Files (binary data, only if uploaded)
  clientPassport: [File],
  clientEmiratesId: [File]
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

*Last Updated: January 2025*
# CloudVault - File Upload Manager
**Secure file uploads with Dropbox storage and Firebase metadata management**

---

## 📋 Quick Overview

CloudVault is a web application that allows users to:
- 🔐 Authenticate securely with Firebase
- 📤 Upload files to Dropbox via OAuth 2.0
- 📊 Store file metadata in Firebase Realtime Database
- 🔗 Generate and manage shareable links
- 🗑️ Delete files from both Dropbox and Firebase

---

## 🚀 Quick Start

### 1. Prerequisites
- Dropbox Developer Account (https://www.dropbox.com/developers/apps)
- Firebase Project (https://console.firebase.google.com)
- VS Code with Live Server extension

### 2. Setup Dropbox OAuth
1. Create a Dropbox app (Scoped API → App folder)
2. Enable scopes: `files.metadata.read`, `files.content.read`, `files.content.write`, `sharing.write`
3. Get your **Client ID** and **Client Secret**
4. Set redirect URI to: `http://localhost:5500/AdsUpload/oauth-callback.html`

### 3. Configure CloudVault
Edit `dropbox-config.js`:
```javascript
clientId: 'YOUR_CLIENT_ID',
clientSecret: 'YOUR_CLIENT_SECRET' // or leave empty to prompt
```

### 4. Run the App
1. Open `index.html` with VS Code Live Server (Go Live button)
2. Sign in with Firebase credentials
3. Click "Upload All" to start uploading files

---

## 📁 File Structure

```
AdsUpload/
├── index.html                    # Main app UI and logic
├── auth.html                     # Firebase authentication page
├── oauth-callback.html           # Dropbox OAuth redirect handler
│
├── dropbox-config.js             # Dropbox URLs and endpoints config
├── dropbox-oauth-handler.js      # OAuth token management
├── navigation-config.js          # Navigation URLs config
│
├── styles.css                    # App styling
│
└── [Documentation Files]
    ├── README.md                 # This file
    ├── DROPBOX_OAUTH_SETUP.md   # Dropbox OAuth detailed setup
    ├── FIREBASE_SETUP.md        # Firebase configuration guide
    └── ... (other guides)
```

---

## 🔐 Authentication Flow

### User Authentication
1. User visits app → redirected to `auth.html`
2. Firebase Google Sign-In (handled via `auth.html`)
3. After auth, redirected back to `index.html`
4. Email verified against `AUTHORIZED_EMAIL`

### Dropbox OAuth
1. User clicks "Upload All"
2. App checks for stored access token
3. If expired → automatically refreshes using refresh token
4. If no token → redirects to Dropbox OAuth login
5. User authorizes → receives tokens
6. Tokens stored in localStorage

---

## 📤 Upload Flow

```
User selects files
    ↓
Drag & drop OR click browse
    ↓
Files added to queue
    ↓
Click "Upload All"
    ↓
Get valid Dropbox token (refresh if needed)
    ↓
FOR EACH FILE:
  1. Upload to Dropbox (to /ads/{type}/{timestamp}_{filename})
  2. Create shareable link (public access)
  3. Save metadata to Firebase (fileName, size, link, timestamp)
    ↓
Show results with download links
```

---

## 🔧 Configuration Files

### `dropbox-config.js`
Contains all Dropbox-related URLs and configurations:
- OAuth endpoints
- API endpoints
- Storage keys
- Helper methods

**Usage:**
```javascript
import DROPBOX_CONFIG from './dropbox-config.js';
DROPBOX_CONFIG.api.fileUpload       // Upload endpoint
DROPBOX_CONFIG.redirectUri          // OAuth redirect URL
DROPBOX_CONFIG.getUploadPath(...)   // Generate Dropbox path
```

### `dropbox-oauth-handler.js`
OAuth token management:
- `getValidDropboxToken()` - Get token, refresh if needed
- `initiateOAuthLogin()` - Start OAuth flow
- `exchangeCodeForToken()` - Exchange auth code for tokens
- `clearDropboxTokens()` - Clear stored tokens (logout)

### `navigation-config.js`
Centralized navigation URLs:
- All page redirects
- OAuth redirects
- Helper methods

**Usage:**
```javascript
import NAVIGATION_CONFIG from './navigation-config.js';
NAVIGATION_CONFIG.redirectToAuth()     // Go to login
NAVIGATION_CONFIG.redirectAfterOAuth() // After OAuth success
```

---

## 🗄️ Database Structure

### Firebase Realtime Database
```
quadraticeqn-b0021-default-rtdb/
└── ads/
    ├── all/
    │   ├── {fileId}: {fileData}
    ├── banner/
    │   ├── {fileId}: {fileData}
    ├── fullimage/
    │   ├── {fileId}: {fileData}
    └── video/
        ├── {fileId}: {fileData}

// File Data Structure:
{
  fileName: "example.jpg",
  fileSize: 1048576,
  fileType: "banner",
  dropboxPath: "/ads/banner/1709...")
  shareLink: "https://www.dropbox.com/s/...",
  uploadedAt: "2026-02-25T10:30:00.000Z"
}
```

---

## 🎯 Key Features

### ✅ Automatic Token Refresh
- Tokens automatically refresh when expired
- No need to re-authenticate
- Refresh tokens stored securely in localStorage

### ✅ Drag & Drop File Selection
- Drag files directly onto upload area
- Or click to browse
- Multiple file selection supported

### ✅ File Type Organization
- Organize files by type (All, Banner, Image, Video)
- Files stored in Dropbox under `/ads/{type}/`

### ✅ Database File Management
- View all uploaded files in database viewer
- See file metadata (name, size, upload date)
- Download files via share link
- Delete files from Dropbox and Firebase simultaneously

### ✅ Progress & Error Handling
- Real-time upload progress display
- Detailed error messages
- Failed uploads don't affect subsequent files

---

## 🛠️ Troubleshooting

### "Cannot GET /AdsUpload/oauth-callback.html"
**Solution:** Ensure redirect URI in Dropbox Console matches:
- For Local: `http://localhost:5500/AdsUpload/oauth-callback.html`
- For Production: `https://yourdomain.com/AdsUpload/oauth-callback.html`

### "client secret is missing"
**Solution:** Either:
1. Add client secret to `dropbox-config.js`
2. Or enter it when prompted in browser

### "Token refresh failed"
**Solution:** 
1. Check Dropbox app scopes are correct
2. Verify Client ID and Secret are valid
3. Clear localStorage: `localStorage.clear()`
4. Re-authenticate

### Files not appearing in database
**Solution:**
1. Check Firebase credentials in `index.html`
2. Verify Dropbox upload succeeded (check results)
3. Check Firebase rules allow read/write for your email

---

## 📊 Storage Keys

### localStorage Keys (Browser)
- `dropbox_access_token` - Short-lived access token (~4 hours)
- `dropbox_refresh_token` - Long-lived refresh token (months)
- `dropbox_token_expiry` - Token expiration timestamp

### Dropbox Paths
- `/ads/all/` - All file types
- `/ads/banner/` - Banner images
- `/ads/fullimage/` - Full-size images
- `/ads/video/` - Video files

---

## 🔒 Security Notes

1. **Client Secret**: For production, never commit to GitHub
2. **HTTPS Only**: Use HTTPS in production (not just localhost)
3. **Email Verification**: Currently restricted to `ugomartinsdegreat@gmail.com`
4. **Token Storage**: Refresh tokens stored in localStorage (not encrypted)
5. **CORS**: Dropbox API calls use Bearer token headers

---

## 📝 File Descriptions

| File | Purpose |
|------|---------|
| `index.html` | Main app UI + upload logic |
| `auth.html` | Firebase authentication page |
| `oauth-callback.html` | Dropbox OAuth redirect handler |
| `dropbox-config.js` | Dropbox URLs & endpoints config |
| `dropbox-oauth-handler.js` | OAuth token lifecycle management |
| `navigation-config.js` | Navigation URL configuration |
| `styles.css` | App styling (UI theme) |

---

## 🚀 Deployment Checklist

- [ ] Update `AUTHORIZED_EMAIL` in `index.html` to your email
- [ ] Add `clientSecret` to `dropbox-config.js` (or use prompt)
- [ ] Update Firebase `projectId` in `index.html`
- [ ] Update Dropbox redirect URI to production domain
- [ ] Change `http://` to `https://` in production
- [ ] Test file upload end-to-end
- [ ] Test token refresh after 4 hours
- [ ] Verify database viewer shows all uploaded files

---

## 📞 Support Resources

- **Dropbox OAuth**: https://developers.dropbox.com/documentation/oauth/guide
- **Firebase Setup**: https://firebase.google.com/docs
- **Live Server**: https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer

---

## 📜 Version History

**v2.0** (Current - Feb 25, 2026)
- ✅ Dropbox OAuth 2.0 with automatic token refresh
- ✅ Centralized configuration objects
- ✅ Database file viewer with delete functionality
- ✅ Improved error handling and progress display

**v1.0** (Earlier)
- Firebase authentication
- Basic file upload
- Database storage

---

## 📝 Notes

- App currently authenticated to single user: `ugomartinsdegreat@gmail.com`
- To add more users, update `AUTHORIZED_EMAIL` validation logic
- Dropbox free tier has storage limits
- Firebase free tier has database read/write limits

---

**Last Updated:** February 25, 2026
   ```bash
   python -m SimpleHTTPServer 8000
   ```
   
   **Using Node.js (with http-server):**
   ```bash
   npx http-server
   ```
   
   **Using PHP:**
   ```bash

3. Open your browser and navigate to:
   ```

## Usage

1. Click the **"Pcloud Configuration"** section
2. Enter your **Access Token** from Pcloud Developer Console
3. Optionally, enter a **Folder ID** (default is `0` for root directory)
### Step 2: Select Files to Upload

Choose files to upload in one of these ways:

- **Click "Choose Files"** button and select files from your computer
- **Drag and drop** files directly onto the upload box
- **Multi-select** is supported - add multiple files to the queue

### Step 3: Upload Files

1. Review files in the **"Files to Upload"** section
2. Remove any files if needed by clicking **"Remove"**
3. Click **"Upload All Files"** button
4. Monitor the progress bar and upload results
5. View results in the **"Upload Results"** section

## API Endpoints Used

### Pcloud Upload Endpoint

**Endpoint:** `https://api.pcloud.com/uploadfile`

**Parameters:**
- `access_token` (required): Your Pcloud API access token
- `folderid` (required): Target folder ID (0 for root)
- `file` (required): File to upload
- `filename` (optional): Custom filename for the uploaded file

**Response:**
- Success: File metadata including file ID and download link
- Error: Error message with error code

## Browser Compatibility

- Chrome/Edge 90+
- Firefox 88+
- Safari 14+
- Opera 76+
- Mobile browsers (iOS Safari, Chrome Mobile)

## Security Considerations

⚠️ **Important Security Notes:**

1. **Token Storage**: Your Pcloud access token is stored in browser's localStorage. This is relatively safe for personal use but consider the security implications for shared computers.

2. **HTTPS Recommended**: Always use HTTPS in production to encrypt data in transit.

3. **Token Permissions**: Create Pcloud API tokens with minimal required permissions (file upload only).

4. **Token Rotation**: Regularly update your API token for enhanced security.

5. **File Validation**: The application accepts all file types. Implement server-side validation if needed.

## Frontend CDN Dependencies

This application uses the following CDN-hosted library:

- **Axios** (v1.6.0): HTTP client for API requests
  ```html
  <script src="https://cdn.jsdelivr.net/npm/axios@1.6.0/dist/axios.min.js"></script>
  ```

## File Structure

```
AdsUpload/
├── index.html          # Main HTML file with structure
├── styles.css          # CSS styling and animations
├── app.js              # JavaScript application logic
└── README.md           # This file
```

## Technical Stack

- **HTML5**: Semantic markup and file input handling
- **CSS3**: Grid, flexbox, gradients, animations
- **Vanilla JavaScript ES6+**: No frameworks or build tools required
- **Axios**: HTTP requests to Pcloud API
- **Pcloud API**: Cloud storage backend

## Features in Detail

### Drag and Drop
- Visual feedback when dragging files over the upload area
- Support for multiple files in a single drag-drop operation

### Progress Tracking
- Real-time progress bar during upload
- Percentage display
- Individual file status tracking

### Result Management
- Success/error status for each uploaded file
- Maintains history of last 50 uploads
- Clear error messages for troubleshooting

### Responsive Design
- Works seamlessly on all screen sizes
- Touch-friendly buttons and controls
- Mobile-optimized layout

## Troubleshooting

### Issue: "Invalid Pcloud access token"
- Verify your access token is correct
- Check that your token hasn't expired
- Generate a new token from Pcloud Developer Console

### Issue: Files fail to upload
- Check your internet connection
- Verify the target folder ID exists in your Pcloud account
- Ensure you have enough storage space on Pcloud
- Check browser console for detailed error messages

### Issue: Configuration not saving
- Ensure localStorage is enabled in your browser
- Check if you're in private/incognito mode (localStorage may be disabled)
- Clear browser cache and try again

### Issue: Drag and drop not working
- Make sure you're using the application over HTTP/HTTPS
- File:// protocol has limitations with drag-drop

## Performance Tips

1. **Batch Uploads**: Upload multiple files at once rather than individually
2. **File Size**: While there's no strict limit in the UI, Pcloud may have limits
3. **Browser Cache**: Clear browser cache periodically for better performance

## API Rate Limiting

Pcloud may have rate limiting on their API. If you encounter rate limit errors:
- Wait a few seconds before retrying
- Reduce the number of simultaneous uploads
- Contact Pcloud support for higher limits

## Support & Documentation

- [Pcloud API Documentation](https://docs.pcloud.com/)
- [Pcloud Developer Console](https://developer.pcloud.com/)
- [Axios Documentation](https://axios-http.com/)

## License

This project is free to use and modify for personal or commercial use.

## Version

**CloudVault v1.0.0**
- Initial release
- Pcloud integration
- Drag and drop support
- Responsive design

---

Made with ❤️ for cloud storage enthusiasts

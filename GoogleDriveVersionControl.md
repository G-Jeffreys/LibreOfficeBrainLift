If you have multiple people who have access to the same Google Drive document and they both edit the document (no in-browser, using something on their computer), how does version control work there?

For context, I am making a plugin for OpenOffice which allows you to open Google Drive documents and edit them in OpenOffice and save them back to the cloud, so I am concerned by how I should/how much I should handle version control.
ChatGPT said:
When multiple users access and edit a Google Drive document without using the browser (i.e., through a third-party app like your OpenOffice plugin), version control becomes complex because Google Docs is inherently built for real-time, in-browser, collaborative editing with automatic conflict resolution.
Since you’re building a plugin for OpenOffice that will download/edit/upload Google Drive documents outside that real-time ecosystem, here’s how Google Drive handles this — and what you need to do:
🧠 How Google Drive Handles Version Control (Non-browser clients):
Versioning is file-based.
Google Drive keeps every uploaded version of a file. When a new file is uploaded with the same name/path, it becomes a new version of that file.
You can view/download previous versions, but Drive doesn't automatically merge content from concurrent edits made offline.
No conflict resolution is automatic outside Docs UI.
If two users edit a file separately (e.g., in OpenOffice) and both upload their edits:
The last one to upload overwrites the previous.
Google Drive does not merge or warn — only version history will show both versions, in sequence.
🔧 What Your Plugin Should Do:
To safely integrate OpenOffice with Google Drive, here are recommendations for handling version control:
✅ 1. Fetch and store a file revision ID upon download
Google Drive API provides a revisionId for every file version.
When downloading a file, store the current revisionId.
✅ 2. Compare revisionId before upload
Before uploading edits:
Fetch the latest revisionId from Drive.
If it matches the one stored → safe to upload.
If it does not match → another user has modified the file in the meantime.
⚠️ 3. If revision mismatch, prompt the user
When the IDs don’t match:
Alert the user: “This document has been updated since you last downloaded it.”
Offer choices:
Overwrite the remote version.
Save as a new version/copy.
Download the new version and re-apply changes manually.
💡 4. Auto-upload as a new version (safe default)
To avoid overwrites without confirmation, default behavior could be:
Save user edits as a new file (e.g., “Filename (YourName’s edits).odt”).
Let the user merge manually or share via Drive.
🔒 Optional Enhancements
Implement file locking: When a user starts editing, create a temporary marker file in Drive or use Drive comments/metadata to signal a "lock".
Use Google Workspace File Stream API or Workspaces for better syncing (but these are enterprise features).
🧭 Summary: What You Must Handle
Concern	Should Plugin Handle?	Notes
File version conflicts	✅ Yes	Use revisionId
Real-time merging	❌ No	Too complex outside browser
Auto-versioning	✅ Yes	Upload with keepRevisionForever if needed
User alerts on conflict	✅ Yes	Warn users before overwriting
Offline edits	✅ Yes	But treat with care if stale
Let me know if you'd like example code using the Google Drive API to handle revisionId comparison or upload workflows.

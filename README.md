# Birthday Card Website 🎂
Website link: https://selena1224.github.io/birthday_card/

A fun, interactive birthday card where our team can leave messages and celebrate together, no matter the distance!

## Website Setup Guide

### 1. GitHub Pages

1. Upload `index.html` to the GitHub repository.
2. Go to **Settings → Pages**.
3. Select **Deploy from a branch**.
4. Choose `main` and `/ (root)`.
5. Click **Save**.

### 2. Supabase Database

1. Go to [Supabase](https://supabase.com/dashboard).
2. Open the `birthday_card` project.
3. Select **SQL Editor → New query**.
4. Copy the contents of `supabase-setup.sql` into the editor.
5. Click **Run**.

### 3. Connect Supabase to GitHub

1. Open the GitHub repository.
2. Click `index.html`.
3. Click the pencil icon to edit the file.
4. Press **Ctrl + F** and search for `SUPABASE_URL`.
5. Find these lines:

```javascript
const SUPABASE_URL = 'YOUR_SUPABASE_PROJECT_URL';
const SUPABASE_ANON_KEY = 'YOUR_SUPABASE_PUBLISHABLE_KEY';
```

Replace them with:

```javascript
const SUPABASE_URL = 'https://gnyuyyyiazpfqnxhaobw.supabase.co';
const SUPABASE_ANON_KEY = 'sb_publishable_YOUR_ACTUAL_KEY_HERE';
```

Replace `sb_publishable_YOUR_ACTUAL_KEY_HERE` with the full publishable key from **Supabase → Settings → API Keys**.

**Important:** Keep the quotation marks and never use a secret or `service_role` key in the website.

6. Click **Commit changes**.

### 4. Test the Birthday Card

1. Open the published GitHub Pages website.
2. Enter a name and birthday message.
3. Click **Add My Birthday Wish**.
4. Refresh the page and confirm the message is still there.
5. Open the website on another device to confirm the message is visible to everyone.

### 5. Organizer Access

The website includes an organizer-only option for deleting birthday messages.

Follow the authentication and security instructions in `SETUP.md` to configure organizer access in Supabase. Delete permissions must be enforced by database policies, not just by hiding the button.

### Notes

- Coworkers do not need accounts to sign.
- Birthday messages are visible to everyone.
- GitHub Pages hosts the website.
- Supabase stores the messages.
- The database must be connected for shared signing to work.

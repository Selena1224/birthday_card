# 🎂 Interactive Group Birthday Card Website
Website link: https://selena1224.github.io/birthday_card/


A custom birthday card website developed for a remote team member in the Philippines. The website allows coworkers to leave birthday messages through a shared link without creating accounts.

Messages are stored in a PostgreSQL database through Supabase, displayed on a shared message wall, and managed through organizer-only access.

## Technologies Used

- **HTML5:** Website structure and forms
- **CSS3:** Responsive design, dark theme, and animations
- **JavaScript:** Form handling, validation, and interactive features
- **Supabase:** Backend services, authentication, and database integration
- **PostgreSQL:** Data storage, SQL queries, and access control
- **GitHub Pages:** Website hosting and deployment

## Key Features

- Group birthday card with a custom design and embedded images
- Public message submission without requiring accounts
- Shared message wall displaying everyone's birthday wishes
- Automatic message refresh approximately every 12 seconds
- Email-based organizer authentication
- Organizer-only message deletion
- Database security using Row Level Security (RLS)
- Mobile-friendly design and confetti animations

## System Architecture

The website uses GitHub Pages for frontend hosting and Supabase for backend operations.

**Workflow:**

1. A coworker opens the website through the shared GitHub Pages link.
2. They enter their name, birthday message, and selected emoji.
3. JavaScript validates the submission and sends it to Supabase.
4. PostgreSQL stores the message in the `birthday_wishes` table.
5. The website retrieves and displays messages for all visitors.
6. The verified organizer can delete messages using an authenticated session.

## Database Design

Created a PostgreSQL table named `birthday_wishes` to store submissions.

| Column | Data Type | Description |
|---|---|---|
| id | BIGINT | Unique message identifier |
| name | TEXT | Name of the person signing |
| message | TEXT | Birthday message |
| emoji | TEXT | Selected emoji |
| created_at | TIMESTAMPTZ | Date and time submitted |

### SQL: Create Database Table

```sql
CREATE TABLE IF NOT EXISTS public.birthday_wishes (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name TEXT NOT NULL,
    message TEXT NOT NULL,
    emoji TEXT NOT NULL DEFAULT '🎂',
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

Enabled Row Level Security:

```sql
ALTER TABLE public.birthday_wishes
ENABLE ROW LEVEL SECURITY;
```

## SQL Queries and Security Policies

### 1. Allow Visitors to Read Messages

Created a policy that allows anonymous users to view birthday messages.

```sql
CREATE POLICY "Anyone can read birthday wishes"
ON public.birthday_wishes
FOR SELECT
TO anon
USING (true);
```

### 2. Allow Visitors to Submit Messages

Created an INSERT policy so coworkers can submit birthday wishes without creating accounts.

```sql
CREATE POLICY "Anyone can leave a birthday wish"
ON public.birthday_wishes
FOR INSERT
TO anon
WITH CHECK (true);
```

### 3. Allow Authenticated Users to Read and Post

During testing, I identified that signing in changed the Supabase user role from `anon` to `authenticated`.

The existing policies did not provide the required permissions to the authenticated role.

Added the following SQL permissions:

```sql
GRANT SELECT, INSERT
ON public.birthday_wishes
TO authenticated;
```

Created an authenticated-user read policy:

```sql
CREATE POLICY "Signed-in users can read wishes"
ON public.birthday_wishes
FOR SELECT
TO authenticated
USING (true);
```

Created an authenticated-user insert policy:

```sql
CREATE POLICY "Signed-in users can post wishes"
ON public.birthday_wishes
FOR INSERT
TO authenticated
WITH CHECK (true);
```

### 4. Organizer Authentication

Implemented email-based authentication using Supabase Auth.

Created a PostgreSQL function to verify whether the signed-in user is the authorized organizer.

```sql
CREATE OR REPLACE FUNCTION public.is_birthday_card_admin()
RETURNS BOOLEAN
LANGUAGE SQL STABLE SECURITY DEFINER
SET search_path = ''
AS $$
    SELECT EXISTS (
        SELECT 1
        FROM auth.users u
        WHERE u.id = auth.uid()
        AND LOWER(u.email) =
            LOWER('ORGANIZER_EMAIL@example.com')
        AND u.email_confirmed_at IS NOT NULL
    );
$$;
```

The organizer's actual email address is excluded from this public documentation.

### 5. Organizer-Only Message Deletion

Implemented a DELETE policy to restrict message removal to the verified organizer.

```sql
GRANT DELETE
ON public.birthday_wishes
TO authenticated;

CREATE POLICY "Organizer can delete birthday wishes"
ON public.birthday_wishes
FOR DELETE
TO authenticated
USING (public.is_birthday_card_admin());
```

The Delete button is shown only to the organizer, while PostgreSQL Row Level Security enforces the actual permission.

## JavaScript and Supabase Integration

### Retrieve Birthday Messages

```javascript
const { data, error } = await supabase
    .from('birthday_wishes')
    .select('id, name, message, emoji, created_at')
    .order('created_at', { ascending: false })
    .limit(500);
```

This query retrieves the most recent birthday messages and displays them in reverse chronological order.

### Insert a New Birthday Message

```javascript
const { error } = await supabase
    .from('birthday_wishes')
    .insert({
        name,
        message,
        emoji
    });
```

Messages are saved to PostgreSQL and displayed on the shared message wall.

### Delete a Birthday Message

```javascript
const { error } = await supabase
    .from('birthday_wishes')
    .delete()
    .eq('id', id);
```

This operation identifies a message by its unique ID. Supabase permissions determine whether the signed-in user can delete it.

### Email Authentication

```javascript
const { error } = await supabase.auth.signInWithOtp({
    email,
    options: {
        emailRedirectTo:
            window.location.origin +
            window.location.pathname
    }
});
```

This sends a one-time sign-in link to the organizer's email address.

## Testing and Troubleshooting

Several issues were identified and resolved during development.

### Issue 1: Database Connection Error

**Problem:** Birthday messages failed to load.

**Cause:** The Supabase URL incorrectly included `/rest/v1/`.

**Solution:** Updated the configuration to use the base project URL.

```javascript
const SUPABASE_URL =
    'https://YOUR_PROJECT_ID.supabase.co';
```

### Issue 2: Authentication Redirect Error

**Problem:** After clicking the verification email, the browser redirected to `localhost`.

**Cause:** Supabase authentication was configured with a local development URL.

**Solution:** Updated Supabase Authentication URL Configuration to use the deployed GitHub Pages URL.

### Issue 3: Organizer Not Recognized

**Problem:** The account successfully signed in but was not recognized as the organizer.

**Cause:** The organizer verification function did not match the authorized email.

**Solution:** Updated the PostgreSQL function to verify the signed-in user's email and confirmation status.

### Issue 4: Message Submission Failed

**Problem:** Messages could not be submitted after organizer authentication.

**Cause:** Database INSERT permissions were configured for anonymous users but not authenticated users.

**Solution:** Added the required `GRANT` statement and INSERT policy for authenticated users.

### Issue 5: Delete Button Not Working

**Problem:** The organizer could see the Delete button, but messages were not deleted.

**Cause:** PostgreSQL DELETE permissions and the organizer-specific RLS policy needed to be configured.

**Solution:** Added the DELETE grant and policy using the organizer verification function.

### Issue 6: Authentication Email Delivery

**Problem:** Repeated login attempts resulted in unsuccessful email requests.

**Possible cause:** Supabase's built-in authentication email rate limits.

**Resolution:** Reduced repeated email requests and corrected the authentication redirect settings.

## Deployment

The frontend is deployed through GitHub Pages.

**Deployment process:**

1. Created a GitHub repository.
2. Uploaded the `index.html` website file.
3. Enabled GitHub Pages deployment from the `main` branch.
4. Created and configured the Supabase project.
5. Connected the website using the Supabase project URL and publishable API key.
6. Executed SQL queries to create database permissions and security policies.
7. Configured email authentication and organizer authorization.
8. Tested anonymous submissions, message visibility, and organizer-only deletion.

## Security Considerations

- Supabase Row Level Security controls database access.
- Anonymous users can read and submit messages.
- Only the authorized organizer can delete messages.
- The website does not expose a database secret or `service_role` key.
- Public message submissions are not moderated automatically.
- Messages are visible to anyone with access to the website.

## Project Outcome

Developed and deployed an interactive birthday card website with persistent shared messages, anonymous submissions, secure organizer authentication, and controlled message deletion.

The project demonstrates practical experience with **frontend development, JavaScript, PostgreSQL, SQL queries, CRUD operations, REST API integration, authentication, database security, debugging, and GitHub Pages deployment**.

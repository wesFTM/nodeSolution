# Movement Live Event Microsite

Tech Stack:

- NextJS
- Typescript
- Sass
- Firebase
- Sendgrid
- Github Actions

Pages:

- index (landing page, which dynamically shows RSVP or Welcome page depending on the user's state)
- lookup (allows getting back to a previous registration)
- success (after rsvp)

API Endpoints:

- /api/register (create a new registration)
- /api/lookup (looks up a previous registration)

User Data Model:

```ts
interface FirestoreUserDocument {
  email: string
  username: string
  zip: string
  dob: string
  referralKey: string
  artist: string
  sipping: string
  maestroAccountId?: string
  referredBy?: string
  utm_source?: string
  utm_medium?: string
  utm_campaign?: string
  utm_term?: string
  utm_content?: string
}
```

## Initial Setup

### Firebase

This microsite uses Firebase for storing user data.

To setup a new Firebase project:

1.  Enable Firestore and set the access rules:
    ```
    rules_version = '2';
    service cloud.firestore {
      match /databases/{database}/documents {
        match /{document=**} {
          allow read, write: if false;
        }
      }
    }
    ```
2.  Put all of the public connection into in `.env.local.example`
3.  Create a Firebase Service Account and add the email/secret to `.env.local`. **Never check your service account details into version control!!**

### Sendgrid

Sendgrid is used to send emails to users.

1.  Create a Sendgrid account
2.  Add a custom domain and verify it
3.  Create dynamic email templates (see below for more info)
4.  Create a sendgrid API key
5.  Update the constants in `util/sendgrid.ts` (from email, template ids)
6.  Add the API key to `.env.local`. **Never check Sendgrid API keys into version control!!**

#### Dynamic Email Templates

You'll need to define a dynamic template in Sendgrid for a post-registration welcome email. It should accept the following variables:

- username
- referralLink

Once created, make sure to update the template id in `util/sendgrid.ts`.

### Vercel

Vercel is used as a scalable NodeJS hosting provider behind a global CDN.

1.  Create a Vercel account
2.  Import this project with default NextJS settings
3.  Add a custom domain
4.  Define environment variables (all of the ones listed in `.env.local`)

If you don't want to use Vercel for any reason, any NodeJS host can work with NextJS. View https://nextjs.org/docs/deployment for more info.

## Local Development

1.  Copy the `.env.local.example` file to `.env.local`.
2.  Add any missing values to `.env.local` (Firebase service account credentials, Sendgrid API key, etc.)
3.  Make sure you have the latest `nodejs`, `npm` and `yarn` installed.
4.  In a terminal, run `yarn` to install dependencies.
5.  Then, run `yarn dev` to start the development server.
6.  Visit the site at http://localhost:3000. As you modify files, the page will automatically update.

This repo includes `prettier` and `eslint`. At any time, you can format code automatically with `yarn format` or run eslint and auto-fix issues when possible with `yarn lint --fix`.

## Customization

This section documents how to customize this microsite to work for other events.

### Change the Design/Copy

These are all the files you should modify if reusing this microsite for a different event:

- The name property in `package.json`
- The Saas stylesheets in `styles/`
- The page HTML in `pages/`
- The component HTML in `components/`
- The favicon and images in `public/`
- The from email address and sendgrid templates in `util/sendgrid.ts`

### Add/Remove User Properties

If you want to remove a user property (e.g. `sipping`) or add a new one (e.g. `favoriteFood`), you need to change the following files:

- Update the FirestoreUserDocument type definition in `types/user.ts`
- Update the RegisterRequestBody type definition in `types/api.ts`
- In `api/register.ts`, add/remove validation checks for the properties and update the data object
- In `components/rsvp.tsx`, add/remove form fields from the UI, add/remove validation checks, and update the api call in the form submit handler

### Replace Maestro with another 3rd party service

This app creates an account in Maestro behind the scenes. If you want to swap out Maestro for another 3rd party service, make these changes:

- Remove `maestroAccountId` and add the new account id property in `types/user.ts`
- In `api/register.ts`, replace the Maestro call with the new service

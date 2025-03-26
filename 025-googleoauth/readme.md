## Prerequisites

Before we begin, ensure you have the following:

- **Node.js** installed on your machine.
- A **Google account** to create OAuth credentials.
- Basic knowledge of JavaScript and Node.js.

## Step 1: Set Up Google OAuth Credentials

To authenticate users via Google, you'll need to set up OAuth 2.0 credentials in the Google Cloud Console.

1. **Access the Google Cloud Console**:
   - Navigate to the [Google Cloud Console](https://console.cloud.google.com/).
   - Sign in with your Google account.

2. **Create a New Project**:
   - Click on the project dropdown at the top of the page.
   - Select "New Project."
   - Provide a name for your project and click "Create."

3. **Enable the OAuth Consent Screen**:
   - In the navigation menu, go to **APIs & Services** > **OAuth consent screen**.
   - Choose "External" for user type and click "Create."
   - Fill in the required details:
     - **App name**: Your application's name.
     - **User support email**: Your email address.
     - **Developer contact information**: Your email address.
   - Click "Save and Continue."

4. **Create OAuth 2.0 Credentials**:
   - Navigate to **APIs & Services** > **Credentials**.
   - Click on **Create Credentials** and select **OAuth client ID**.
   - Choose "Web application" as the application type.
   - Set the **Authorized redirect URIs** to:
     ```
     http://localhost:3000/auth/google/callback
     ```
   - Click "Create."
   - Note down the **Client ID** and **Client Secret** displayed.

## Step 2: Initialize the Node.js Project

1. **Create a New Directory for Your Project**:
   - Open your terminal or command prompt.
   - Run the following commands:
     ```bash
     mkdir google-auth-jwt
     cd google-auth-jwt
     ```

2. **Initialize the Project**:
   - Initialize a new Node.js project by running:
     ```bash
     npm init -y
     ```
   - This will create a `package.json` file with default settings.

3. **Install Required Dependencies**:
   - Install the necessary packages:
     ```bash
     npm install express passport passport-google-oauth20 jsonwebtoken dotenv
     ```
     - **express**: Web framework for Node.js.
     - **passport**: Authentication middleware.
     - **passport-google-oauth20**: Google OAuth 2.0 strategy for Passport.
     - **jsonwebtoken**: Library to work with JSON Web Tokens.
     - **dotenv**: Loads environment variables from a `.env` file.

## Step 3: Configure Environment Variables

1. **Create a `.env` File**:
   - In the root of your project directory, create a file named `.env`.
   - Add the following lines, replacing placeholders with your actual Google OAuth credentials and a secret key for JWT:
     ```env
     GOOGLE_CLIENT_ID=your-google-client-id
     GOOGLE_CLIENT_SECRET=your-google-client-secret
     JWT_SECRET=your-jwt-secret
     ```
     - **GOOGLE_CLIENT_ID**: Your Google OAuth Client ID.
     - **GOOGLE_CLIENT_SECRET**: Your Google OAuth Client Secret.
     - **JWT_SECRET**: A secret key for signing JWTs.

## Step 4: Set Up the Express Application

1. **Create the `app.js` File**:
   - In your project directory, create a file named `app.js`.

2. **Import Required Modules and Configure Middleware**:
   - Open `app.js` and add the following code:
     ```javascript
     require('dotenv').config();
     const express = require('express');
     const passport = require('passport');
     const jwt = require('jsonwebtoken');
     const { Strategy: GoogleStrategy } = require('passport-google-oauth20');

     const app = express();

     app.use(passport.initialize());

     // Configure Passport to use Google OAuth 2.0 strategy
     passport.use(new GoogleStrategy({
       clientID: process.env.GOOGLE_CLIENT_ID,
       clientSecret: process.env.GOOGLE_CLIENT_SECRET,
       callbackURL: '/auth/google/callback',
     }, (accessToken, refreshToken, profile, done) => {
       // Here, you would typically find or create a user in your database
       // For this example, we'll just return the profile
       return done(null, profile);
     }));

     // Route to initiate Google OAuth flow
     app.get('/auth/google',
       passport.authenticate('google', { scope: ['profile', 'email'] })
     );

     // Callback route that Google will redirect to after authentication
     app.get('/auth/google/callback',
       passport.authenticate('google', { session: false }),
       (req, res) => {
         // Generate a JWT for the authenticated user
         const token = jwt.sign({ id: req.user.id, displayName: req.user.displayName }, process.env.JWT_SECRET, { expiresIn: '1h' });
         // Send the token to the client
         res.json({ token });
       }
     );

```

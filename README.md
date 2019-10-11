# Sign In with Apple using Python Django
_An implementation with Python Django + Python Social Auth and iOS._

Apple announced a new feature, "Sign In with Apple" enabling users to sign in to apps using their Apple ID. This new feature is meant to be a secure and privacy-friendly way for users to create an account in apps. Most iOS and Mac users already have an Apple ID, and this new feature lets them use that Apple ID to sign in to other apps and websites.

Apple is taking a firm stance to protect user's privacy, rather than letting applications see the user's real email address, they will provide the app with a fake or random email address unique to each app. Don't worry! Developers will still be able to send emails to these proxy addresses, it just means developers won't be able to use the email addresses in any other way. This feature will also allow users to disable email forwarding per application.

## What this document will cover

This document is about how to implement Sign In with Apple on iOS and server-side verification for account creation in a Python Django backend with Python Social Auth.

1. [Configure Identifiers and Keys](identifiers-and-keys.md)
2. [Implement Sign In with Apple in your backend](backend.md)
3. [Implement Sign In with Apple in your iOS app](iOS.md)

## Flow

<img src="resources/flow-diagram.png">

# Configure Identifiers and Keys on the Apple Developer Portal

## Add Sign In with Apple to your app's capabilities

1. Go to Xcode's project navigator
2. Select your Xcode project
3. Select your target
4. Go to the _Signing & Capabilities_ tab
5. Click the _+ Capability_ button
6. Select _Sign In with Apple_

### A) Automatic Signing

Done. Xcode will add it to the entitlements, sync the capabilities to the Developer Portal and generate new provisioning profiles for you.

### B) Manual Signing

#### Add Sign In With Apple to the app ID

7. Go to the Developer Portal and select _Identifiers_.
8. Select your app your want to add Sign In with Apple to.
9. Scroll down to Sign In with Apple and tick the checkbox.
10. Click Save

By default this app ID is enabled to be the primary app ID.  
Optionally: If you're using multiple apps or web authentication you might want to assign them to this app ID as a group.

#### Re-generate provision profiles

Changing the app ID invalidates the provisioning profiles associated with it, so we need to generate them again.

11. Go to _Profiles_.
12. Click on each profile associated with this app ID and just save it again.
13. Either download it from the portal manually or through Xcode.

Now you can continue with the [iOS implementation](iOS.md#1-add-the-authenticationservices-framework).

## Create a _Sign In with Apple_ Key for your backend

1. Go to "Keys"
2. Click Register a new Key
3. Give it a name and tick the Sign In with Apple checkbox
4. Click "Configure" and select you app ID
5. Click "Save"
6. Click "Continue"
7. Click "Register"
8. Click "Download" (This is important!)

Once you have the key file you can continue with the [backend implementation](backend.md).

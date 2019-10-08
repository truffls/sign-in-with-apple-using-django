# Implement Sign In with Apple on iOS

_Requirements: Xcode 11, iOS 13_

## 1. Add Sign In with Apple to your app's capabilities

1. Go to project navigator
2. Select the Xcode project
3. Select the target
4. Go to the _Signing & Capabilities_ tab
5. Click the _+ Capability_ button
6. Select _Sign In with Apple_

### A) Automatic Signing

Done. Xcode will sync the Capabilities to the Developer Portal and generate new provisioning profiles for you.

### B) Manual Signing

7. Go to the Developer Portal and select _Identifiers_.
8. Select your app your want to add Sign In with Apple to.
9. Scroll down to Sign In with Apple and tick the checkbox.
10. Click Save

By default this app ID is enabled to be the primary app ID. Optionally: If you're using multiple apps or web authentication you might want to assign them to this app ID as a group.

By changing the app ID, the provisioning profiles associated with this it become invalid and we need to generate them again.

11. Go to _Profiles_.
12. Click on each profile associated with this app ID and just save it again.
13. Either download it from the portal manually or through Xcode.


## 2. Add the AuthenticationServices framework

In this example the logic is implemented directly in a view controller.

First, import the `AuthenticationServices` to the view controller:

```swift
import AuthenticationServices
```

## 3. Add the Apple ID button to your view

Add the Sign In with Apple button `ASAuthorizationAppleIDButton` to the view. It behaves pretty much like a `UIButton`, except that you can only change the frame, type of title, corner radius, and style. It scales the text according to its frame automatically and even localizes itself. 

```swift
// Instantiate the button with a type and style
let signInButton = ASAuthorizationAppleIDButton(type: .signIn, style: .black)

// Add an action to be called when tapping the button
signInButton.addTarget(self, action: #selector(signInButtonPressed), for: .touchUpInside)
```

You can read more in the documentation of [ASAuthorizationAppleIDButton](https://developer.apple.com/documentation/authenticationservices/asauthorizationappleidbutton).

## 4. iOS authorization process

The process consists of three parts: 
- Create the request
- Provide a presentation context
- Handle the authorization response

### 4.1. Create the request

The next step is to create the request:

```swift
@objc func signInButtonPressed() {
    // First you create an apple id provider request with the scope of full name and email
    let request = ASAuthorizationAppleIDProvider().createRequest()
    request.requestedScopes = [.fullName, .email]

    // Instanstiate and configure the authorization controller
    let authorizationController = ASAuthorizationController(authorizationRequests: [request])
    authorizationController.presentationContextProvider = self
    authorizationController.delegate = self

    // Perform the request
    authorizationController.performRequests()
}
```

### 4.2. Provide a presentation context

The presentation context provider simply requires the window in which the modal should be presented. For that your view controller needs to conform to `ASAuthorizationControllerPresentationContextProviding`. It requires only one method:

```swift
func presentationAnchor(for controller: ASAuthorizationController) -> ASPresentationAnchor {
    return view.window!
}
```

### 4.3. Handle the authorization response

Now that we have setup the request and the presentation context, we need to handle the response by conforming to `ASAuthorizationControllerDelegate`:

#### 4.3.1 Handle authorization errors

```swift
func authorizationController(controller: ASAuthorizationController, didCompleteWithError error: Error) {
    let authError = ASAuthorizationError(_nsError: error)
    switch authError.code {
        // Add cases and handle errors
    }
}
```

For handling the error you can instantiate an `ASAuthorizationError` object by using the initializer that takes an `NSError` (yes, the parameter label has an underscore. I don’t know why Apple did this) or alternatively you can use `ASAuthorizationError.Code(rawValue: error.code)` to instantiate the code directly which, however, will be optional and might return `nil`.

You can see the error codes in the documentation for [ASAuthorizationError.Code](https://developer.apple.com/documentation/authenticationservices/asauthorizationerror/code). I recommend to not show an error alert for the canceled case, since the user actively canceled the process.


#### 4.3.2 Handle a successful response

```swift 
func authorizationController(controller: ASAuthorizationController, didCompleteWithAuthorization authorization: ASAuthorization) {
    if let appleIDCredential = authorization.credential as? ASAuthorizationAppleIDCredential {
        if let data = credential.authorizationCode, let code = String(data: data, encoding: .utf8) {
            // Now send the 'code' to your backend to get an API token.
            exchangeCode(code)
        } else {
            // Handle missing authorization code ...
        }
    }
}
```

We don't store any of the data Apple gives us here. We want our backend to verify the code, get the name and email on its own, store it, and return an API token.

```swift
func exchangeCode(_ code: String) {
    // `myAPI` is a fictional backend API:
    myAPI.exchangeCode(code) { apiToken, error in
        if let apiToken = apiToken {
            // Store API Token
        } else {
            // Handle error
        }
    }
}
```

## 5. Backend authorization process

The backend will receive the code, send it to the AppleID API and get an `id_token` in return with the name and email in it. Read here about the Backend implementation of Sign In with Apple with Python Social Auth.

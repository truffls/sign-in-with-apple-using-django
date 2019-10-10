# Implement Sign In with Apple on iOS

_Requirements: Xcode 11, iOS 13_

To get started you'll need to [add Sign In with Apple to your app's capabilities](identifiers-and-keys.md#add-sign-in-with-apple-to-your-apps-capabilities).

## Add the AuthenticationServices framework

In this example the logic is implemented directly in a view controller.

First, import the `AuthenticationServices` to the view controller:

```swift
import AuthenticationServices
```

## Add the Apple ID button to your view

Add the Sign In with Apple button `ASAuthorizationAppleIDButton` to the view. It behaves pretty much like a `UIButton`, except that you can only change the frame, type of title, corner radius, and style. It scales the text according to its frame automatically and even localizes itself. 

```swift
// Instantiate the button with a type and style
let signInButton = ASAuthorizationAppleIDButton(type: .signIn, style: .black)

// Add an action to be called when tapping the button
signInButton.addTarget(self, action: #selector(signInButtonPressed), for: .touchUpInside)
```

You can read more in the documentation of [ASAuthorizationAppleIDButton](https://developer.apple.com/documentation/authenticationservices/asauthorizationappleidbutton).

## iOS authorization process

The process consists of three parts: 
- Create the request
- Provide a presentation context
- Handle the authorization response

### Create the request

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

### Provide a presentation context

The presentation context provider simply requires the window in which the modal should be presented. For that your view controller needs to conform to `ASAuthorizationControllerPresentationContextProviding`. It requires only one method:

```swift
func presentationAnchor(for controller: ASAuthorizationController) -> ASPresentationAnchor {
    return view.window!
}
```

### Handle the authorization response

Now that we have setup the request and the presentation context, we need to handle the response by conforming to `ASAuthorizationControllerDelegate`:

#### Handle authorization errors

```swift
func authorizationController(controller: ASAuthorizationController, didCompleteWithError error: Error) {
    let authError = ASAuthorizationError(_nsError: error as NSError)
    switch authError.code {
        // Add cases and handle errors
    }
}
```

For handling the error you can instantiate an `ASAuthorizationError` object by using the initializer that takes an `NSError` (yes, the parameter label has an underscore. I don’t know why Apple did this) or alternatively you can use `ASAuthorizationError.Code(rawValue: error.code)` to instantiate the code directly which, however, will be optional and might return `nil`.

You can see the error codes in the documentation for [ASAuthorizationError.Code](https://developer.apple.com/documentation/authenticationservices/asauthorizationerror/code). I recommend to not show an error alert for the canceled case, since the user actively canceled the process.


#### Handle a successful response

```swift 
func authorizationController(controller: ASAuthorizationController, didCompleteWithAuthorization authorization: ASAuthorization) {
    if let credential = authorization.credential as? ASAuthorizationAppleIDCredential {
        if let data = credential.authorizationCode, let code = String(data: data, encoding: .utf8) {
            // Now send the 'code' to your backend to get an API token.
            exchangeCode(code) { apiToken, error in
                // Handle response
            }
        } else {
            // Handle missing authorization code ...
        }
    }
}
```

The only interesting information at this point is the `authorizationCode`. We don't store any of the data Apple gives us. We want our backend to verify the code, get the name and email on its own, verify or create a user, and return an API token.

```swift
func exchangeCode(_ code: String, handler: (String?, Error?) -> Void) {
    // Call your backend to exchange an API token with the code.
}
```

## Backend authorization process

The backend will receive the code, send it to the AppleID API and get an `id_token` in return with the name and email in it. Read here about the [backend implementation](backend.md) or return to the [overview](README.md).

# Implement Sign In with Apple on iOS

_Requirements: Xcode 11, iOS 13_

In the example we’re going to implement the logic directly inside a view controller.

## 1. Add the AuthenticationServices framework

First, import the `AuthenticationServices` to your view controller:

```swift
import AuthenticationServices
```

## 2. Add the Apple ID button to your view

Add the Sign In with Apple button to your view by using the `ASAuthorizationAppleIDButton`. It behaves pretty much like a `UIButton`, except that you can only change the type of title and style. It scales the text according to its frame automatically and even localizes itself. 

```swift
// Instantiate the button with a type and style
let signInButton = ASAuthorizationAppleIDButton(type: .signIn, style: .black)

// Add an action to be called when tapping the button
signInButton.addTarget(self, action: #selector(signInButtonPressed), for: .touchUpInside)
```

## 3. iOS Authorization Process

The process consists of three parts: 
- Create the request
- Provide a presentation context
- Handle the authorization response

### 3.1. Create the Request

The next step is to create the request:

```swift
@objc func signInButtonPressed() {
    // First you create an apple id provider request with the scope of name and email
    let request = ASAuthorizationAppleIDProvider().createRequest()
    request.requestedScopes = [.fullName, .email]

    // Instanstiate the authorization controller and set its delegates
    let authorizationController = ASAuthorizationController(authorizationRequests: [request])
    authorizationController.presentationContextProvider = self
    authorizationController.delegate = self

    // Perform the request
    authorizationController.performRequests()
}
```

### 3.2. Provide a presentation context

The presentation context provider simply requires the window in which the modal should be presented. For that your view controller needs to conform to `ASAuthorizationControllerPresentationContextProviding`. It requires only one method:

```swift
func presentationAnchor(for controller: ASAuthorizationController) -> ASPresentationAnchor {
    return view.window!
}
```

### 3.3. Handle the authorization response

Now that we have setup the request and the presentation context, we need to handle the response by conforming to `ASAuthorizationControllerDelegate`:

#### 3.3.1 Handle authorization errors

```swift
func authorizationController(controller: ASAuthorizationController, didCompleteWithError error: Error) {
    let authError = ASAuthorizationError(_nsError: error)
    switch authError.code {
        // Add cases and handle errors
    }
}
```

For handling the error by error code you can instantiate an `ASAuthorizationError` object by using the initializer that takes an `NSError` (yes, with underscore. I don’t know why Apple did this) or alternatively you can use `ASAuthorizationError.Code(rawValue: error.code)` to instantiate the code directly which will be optional and return `nil`, however.

You can see here (https://developer.apple.com/documentation/authenticationservices/asauthorizationerror/code) the error codes. I recommend to not show an error for the canceled case, since the user actively canceled the process.


#### 3.3.2 Handle a successful response

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

## 4. Backend authorization process

The backend will receive the code, send it to the AppleID API and get an `id_token` in return with the name and email in it. Read here about the Backend implementation of Sign In with Apple with Python Social Auth.
